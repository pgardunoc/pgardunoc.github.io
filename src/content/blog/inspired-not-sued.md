---
title: "The risky part of cloning an arcade classic was never the code"
description: "How I built a game \"inspired by\" a 1980s arcade classic without copying the one thing that would've gotten me sued — an engineer's reasoning about what you're actually allowed to borrow."
pubDate: 2026-09-01
draft: false
---

A while back I built a small game that's clearly inspired by a 1980s arcade classic — the kind everyone recognizes in about half a second. I could walk you through the game loop, how the maze is represented, the way the enemies hunt you down. None of that is the part I sweated. The part I sweated was making sure the thing couldn't get me sued.

Quick disclaimer before anything else: I'm an engineer, not a lawyer, and this isn't legal advice. It's how I reasoned about a real decision before I shipped. But if you're building something "inspired by" a product people already know, the same reasoning might save you a genuinely bad afternoon somewhere down the road.

## The temptation is a trap

When you build in a genre everyone recognizes, the reflex is to clone the whole thing. The name, the look, the sounds — all of it — because that's what makes it feel *right*. It reads like a shortcut. Why reinvent a character everyone already loves?

Here's the problem. A too-faithful clone isn't a shortcut. It's a liability that sits quietly dormant until the exact moment it can hurt you most: when the thing gets traction and is finally worth taking from you. Nobody sends a cease-and-desist over a project with ten users. They send it when you've built something worth having. So the "shortcut" is really a bet that you'll never succeed — a strange bet to place on your own work.

## Where the line actually is

So before I wrote much code, I spent a little time working out where the real line sits between "inspired by" and "ripping off." For someone just trying to ship a product, it comes down to a distinction that's simpler than it sounds.

Some things aren't protectable. The *mechanics* of a game — the rules, the idea of moving through a maze while something chases you — aren't something anyone owns. Ideas and mechanics are fair game, and the original patents from that era expired long ago. You're allowed to build the same *kind* of thing.

What *is* protected — and actively defended — is the identity: the specific expression that makes people point and say "that's that game." In practice, that's:

- **The name.** The single most strongly protected element.
- **The characters and their names.** The hero, the enemies, what each one is called.
- **The sounds.** The specific, recognizable audio.
- **The "trade dress"** — the overall distinctive look and feel: the exact maze shape, the iconic character design, the color language that makes it unmistakable.

A landmark case back in the early 1980s drew almost exactly this line — the underlying idea and mechanic on one side, the protected identity and expression on the other. I'm staying at the level of the principle here rather than the citation, partly because the principle is the useful part and partly because I'm not the person to lecture anyone on case law. The principle is enough to make good decisions.

The reframe that made it click for me as an engineer: "is it different *enough*?" is not the test. That's a gut feeling, and gut feelings don't hold up. The real test is "does it copy the protected identity?" Those are very different questions — and you can build something mechanically identical and be completely fine, as long as the identity is your own.

## What I actually changed

So I gave myself one rule: keep the mechanic — the one thing that isn't protectable — and rebuild every layer of identity as my own. Concretely, that meant:

- **The name** is original. Not a pun on the famous one, not a near-homophone — its own word. If the name is the most protected element, it's the last place to get clever.
- **The maze** is my own layout, not the recognizable one. The shape of the board is trade dress; copying the exact maze copies the look, even when I draw it myself.
- **The player** is a character I call `C` — deliberately not a yellow disc. Character design is identity, so the protagonist had to read as mine at a glance.
- **The enemies** have my own names and my own personalities, not the famous four. Character names get enforced; four differently-named characters with behaviors I designed are mine.
- **The sounds** are synthesized bleeps I generated, not the iconic ones. Recognizable audio is protected, so nothing is sampled or imitated.

Notice what I *didn't* touch: the mechanic. You still move through a maze, you still dodge enemies, the tension is the same. That's the part I was always free to keep. Everything a court would actually look at — name, characters, sound, look — is original.

*(For the engineers and founders reading this: "it's different enough" is not a legal standard. "Does it copy the protected identity" is. Keep the idea, change the trade dress. That one sentence would have saved me the reading if someone had handed it to me up front.)*

## Why this is a senior skill, not a footnote

Here's what I keep coming back to. The engineering was the easy part. Anyone with a weekend and a text editor can clone a maze game. The judgment — knowing exactly what you're allowed to borrow and what you have to make your own — is the part that decides whether the product survives contact with success.

And it generalizes far past games. It's the same call a founder makes shipping anything into an established category: a dev tool that looks a lot like the market leader, a UI that borrows a famous interaction pattern, a brand that lives a little too close to a bigger one. Building *on* the best in a category is completely fair — that's how every category improves. Copying its *identity* is a bill you defer, and you pay it with interest at the worst possible time.

What you leave out is a decision too. The borrowed identity I refused to take isn't a limitation — it's the choice that lets the thing stand on its own, and it's the one that ages best.

If you're shipping something close to the line, the answer is the same as always in engineering: know the rule, then make the call deliberately. And if there's real money or real risk on the table, talk to an actual lawyer — this is how I reasoned, not a substitute for someone who does this for a living.

The whole game — name, maze, characters, and sounds all original — is here: https://github.com/pgardunoc/maze-game
