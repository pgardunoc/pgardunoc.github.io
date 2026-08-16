---
title: "I moved StoryKept from a $0 host to a $19/month one on purpose"
description: "Why I moved StoryKept off a free frontend host to a $19/month Hetzner VPS and S3-compatible bucket and the two things that didn't just work."
pubDate: 2026-08-15
draft: false
---

Vercel is the easiest way to ship a Next.js app. I moved StoryKept off it anyway from a bill of exactly $0 to $19 a month.

That sounds backwards. Free is cheaper than $19. But "free" was only free for the
part of StoryKept that Vercel does beautifully: the frontend. The moment the app
needed a database, persistent storage for user media, and Docker, "free" was about
to stop being the number and the number it was about to become was not $19.

This is the story of that move: why I made it, what I set up by hand, the two things
that didn't just work, and why $19 flat turned out to be the cheaper decision.

## Why leave a free host at all

StoryKept isn't a frontend. People send in voice notes and videos over WhatsApp,
and those files have to be stored safely and served back. That means, at minimum:

- **Next.js** — the part Vercel is genuinely the best at
- **A database** — Postgres
- **Object storage** — for the media, potentially a lot of it
- **Docker** — so the whole thing runs the same everywhere

On Vercel, the frontend is free and the rest is a menu. Each managed piece,
the database, the storage, the bandwidth once media traffic grows, is its own line
item, and each one climbs with usage. The free tier is a doorway, not a house. For
a media-heavy app, the honest projection wasn't "$0 forever." It was "$0 until it
isn't, then a bill that grows with every uploaded video."

So I went shopping for a box that could run the whole stack for one predictable
number. DigitalOcean, Linode, Hetzner. Hetzner won on price-for-performance, and
$19/month covers **both** the VPS and an S3-compatible storage bucket. One number,
flat, for the entire stack.

## What you actually get: a bare Linux box

Hetzner hands you something deliberately minimal: a Linux VPS with a public IP. You
pick the type, the RAM, the location, the OS — and that's the end of the hand-holding.
Everything else is yours to stand up:

- Docker and the container runtime
- Postgres
- The storage bucket
- The SSL certificate
- An SSH-based deploy path

This is the "old way," and it's exactly the part that scares people off Hetzner.
It shouldn't. None of it is exotic it's a sequence of well-documented steps, and
the whole thing was easier than the reputation suggests. The setup, roughly:

1. Spin up the VPS, lock down SSH (key-only, no password login).
2. Install Docker and bring up the app + Postgres as containers.
3. Point a domain at the public IP.
4. Put the app behind a reverse proxy and get a TLS cert.
5. Wire up the storage bucket via the S3 API.
6. Set up a simple SSH deploy so shipping a new version is one command.

Two of those six steps are where I actually spent my time. The rest was typing.

## The storage bucket: same SDK, different endpoint

Here's the piece that makes lock-in a non-issue, and it's the most reusable idea in
this whole post. Amazon's S3 became so ubiquitous that other providers built their
storage to speak the exact same API. Hetzner's bucket is S3-compatible, which means
my application code doesn't know or care that it isn't AWS.

You don't rewrite anything. You keep the same S3 SDK and change three values, the
endpoint, the credentials, and (for some providers) the addressing style:

```js
import { S3Client } from "@aws-sdk/client-s3";

const s3 = new S3Client({
  region: "auto",
  endpoint: process.env.S3_ENDPOINT,        // Hetzner bucket URL, not AWS
  credentials: {
    accessKeyId: process.env.S3_ACCESS_KEY,
    secretAccessKey: process.env.S3_SECRET_KEY,
  },
  forcePathStyle: true,                       // matters more than you'd think
});
```

That's the whole switch. The same code that talked to AWS now talks to Hetzner by
changing environment variables. And the reverse is just as cheap: if Hetzner ever
slips, moving back to AWS or to any other S3-compatible provider is a config
change, not a migration project. The point of avoiding lock-in was never to refuse
to pick a vendor. It's to build so the choice never becomes a cage.

## The two things that didn't just work

A migration where everything works on the first try isn't a real migration. Two
things fought me.

**1. Presigned URLs needed configuration.** StoryKept serves user media through
presigned URLs time-limited links that let a browser fetch a file directly from
the bucket without routing bytes through my server. Against AWS these are close to
zero-config. Against an S3-compatible provider, they didn't work out of the box; the
signing had to be configured to match how the provider expects the request to be
addressed before the generated URLs would validate. Once that was aligned, they
worked exactly like the AWS ones but it was the opposite of plug-and-play, and
it's the kind of thing that passes in local testing and fails the first time a real
file is requested.

**2. HTTPS: the config was manual, the cert was not.** On a managed platform, HTTPS
is a checkbox. On a bare VPS, it's yours to handle and here the right tool choice
did most of the work for me. I put the app behind Caddy as the reverse proxy, and
Caddy does automatic HTTPS: it provisions a free Let's Encrypt certificate on its
own and renews it before it expires, with no cron job and no certbot ritual to
remember. That's a deliberate senior dev call picking the tool whose default is the
thing you don't want to babysit.

It still wasn't zero-effort. You write the Caddyfile, point DNS at the box, and make
sure the ACME challenge can actually reach the server — get any of those wrong and
the cert silently never issues. But that's setup you do once, not a renewal you have
to nurse forever. Compared to hand-rolling certbot on a bare box, choosing Caddy
turned "SSL is a chore I'll forget to renew" into "SSL is a line of config."

## The number, and the actual senior dev call

So: $0 to $19/month. Framed as a raw bill, that's a $19 increase. Framed honestly,
it's the cheaper decision because the $0 was temporary and load-bearing on a
frontend-only definition of the app. $19 flat buys the database, the storage, the
compute, and the bandwidth headroom in one predictable line that doesn't spike when
a user uploads a 200MB video. The bill I avoided isn't $0; it's the per-service
climb the $0 was quietly deferring.

I want to be careful not to oversell it. Hetzner isn't AWS. There are no fifty
managed services on tap, no autoscaling magic, no one to page when the box misbehaves
at 3am that's you. If your time is worth more than the money a managed platform
saves you, pay for the managed platform. That's a real and correct call for a lot of
teams.

But that *is* the call and making it well is the actual senior skill here. Not
"can you run a Linux box," but knowing when a managed platform's convenience is worth
paying for and when it's just a tax on things you're perfectly capable of running
yourself. For a solo founder watching a budget, who knows his way around a server,
$19/month for the whole stack with the freedom to walk away from any piece of it
by changing an environment variable was the right trade.

The hard part was never knowing the tools. It was knowing which brand-name defaults
were worth paying for, and building so that every one of those choices could be
undone.
