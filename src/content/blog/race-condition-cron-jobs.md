---
title: "The night my World Cup model started fighting itself"
description: "A race condition between two cron jobs, and why the fix was to stop merging and start rebuilding."
pubDate: 2026-08-07
draft: false
---

Here's a small bug with a satisfying fix.

Back in July I [wrote up my World Cup model](https://pedrogarduno.com/blog/jetlagxi-world-cup-model/) — Elo, jet-lag penalties, and 10,000 Monte Carlo tournaments, refreshing itself on a schedule all through the tournament. That "refreshing itself on a schedule" part is where this story lives.

A GitHub Action wakes up, pulls the latest scores from the football API, re-runs the simulation, writes a snapshot of the day's predictions, and pushes the result back to the repo. Nothing exotic. It ran fine for weeks.

Then one night in early July it just... failed. And kept failing.

## What I saw

The Action would go red with a merge conflict, of all things — on a file that no human had touched. The conflict was always on the daily snapshot:

```
CONFLICT (add/add): Merge conflict in data/snapshots/2026-07-03.json
```

An *add/add* conflict means two commits each created the same file, from scratch, with different contents. Git has no idea which one you meant, so it gives up and the job exits 1.

But there's only one cron job. How do two commits create the same file at the same time?

## The actual bug

The answer was in the schedule. The cron runs every 30 minutes:

```yaml
- cron: "10,40 * * * *"
```

Except GitHub Actions cron isn't punctual. Under load it lags — 15 to 60 minutes is normal, and I'd deliberately set it to fire twice an hour to catch matches quickly. So on a busy evening, the :10 run would still be grinding away when the :40 run started. **Two copies of the same job, running at once.**

That overlap on its own would've been harmless if the job produced the same bytes every time. It didn't. Two things in the pipeline are non-deterministic:

1. **A timestamp.** Every build stamps itself: `generatedAt: new Date().toISOString()`. Two runs, two different clocks.
2. **Monte Carlo noise.** The whole model is 10,000 randomized tournament simulations — `Math.random()` at its core. Run it twice and Spain's title odds come back 24.1% one time, 24.3% the next. Close, but not byte-identical.

So both overlapping runs generated `data/snapshots/2026-07-03.json` — a brand-new file that day — with slightly different numbers inside. Whoever pushed first, won. The second run came back, tried to rebase its commit on top of the first, and hit two commits both *adding* the same path with different contents. Add/add conflict. Exit 1. Red X.

The old reconciliation step was trying to be careful, and that was exactly the problem:

```bash
git add data/
git commit -m "data: refresh scores (...)"
git pull --rebase --autostash origin main   # <- tries to MERGE two generated files
git push || (git pull --rebase --autostash origin main && git push)
```

It was attempting to *merge* two machine-generated files that were never meant to be merged. There's no sensible resolution — you don't want half of one simulation and half of another.

## The fix

The insight: this data isn't something you *merge*. It's something you *rebuild*. The scores live in the API, not in the repo — the repo is just a cache. So instead of reconciling two divergent versions, throw the loser away and regenerate on top of the winner:

```bash
for attempt in 1 2 3 4 5; do
  git fetch origin main
  git reset --hard origin/main      # start from whatever main is NOW

  npm run scores                    # re-fetch from the API (source of truth)
  npm run lock:knockouts            # cascade the bracket (idempotent)
  npm run build:data                # regenerate the derived data
  npm run snapshot                  # write today's snapshot if absent

  [[ -z "$(git status --porcelain)" ]] && { echo "No changes."; exit 0; }

  git add data/
  git commit -m "data: refresh scores ($(date -u +%Y-%m-%dT%H:%MZ))"
  git push && exit 0                # won the race? done.

  echo "Push lost a race; rebuilding on new main…"
  sleep $((RANDOM % 5 + 2))         # jittered backoff, then try again
done
exit 1
```

The whole shape changes. If a push loses the race, the job doesn't try to force its stale work through — it resets to the new `main` and rebuilds from there. Because every step is idempotent and re-reads scores from the API, rebuilding on fresh main can *never* conflict and can *never* drop a result. The random `sleep` keeps two retriers from lockstepping into each other again.

It's been green ever since — all 104 matches captured, right through to Spain lifting the trophy.

## The lesson I keep relearning

The bug wasn't really in the git commands. It was in treating generated, non-deterministic output like hand-written source you can merge. Two overlapping jobs, a timestamp, and a random seed were all it took.

Once I stopped asking git to reconcile two simulations and instead made the job *regenerate from the source of truth*, the race condition didn't need clever handling — it just stopped being able to happen. The best fix for a race is usually to remove the thing being raced over, not to guard it more carefully.
