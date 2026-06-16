# Rescue-Squad

A two-sided mobile app that connects people in medical emergencies with nearby on-call EMS providers.

## Status

**Plan-complete, implementation-pending.** All four plan reviews clean (CEO + Eng + Design + DX). Direction locked: **open-source community-responder protocol (Apache 2.0), Nepal-first with pan-South-Asia framing, KUDH (Kathmandu University Dhulikhel Hospital) as pilot partner, Android-first via React Native + native Kotlin module, Go backend with Mosquitto + PostgreSQL/PostGIS + MapLibre+OSM.** Real-world implementation blockers (not engineering): KUDH MOU, Nepal data-protection legal review, solo-founder on-call rotation story.

See `research/` (91 primary sources), `DESIGN.md` (design system tokens), `TODOS.md` (54-item catalog), and `.claude/plans/` (gitignored review artifacts) for full context.

## Concept

Two user roles:
1. **EMS provider** — a trained emergency medical service provider who can go "on call" (on shift) and accept help requests.
2. **Help-seeker** — anyone experiencing or witnessing a medical emergency. They request help with one tap (or the equivalent of dialling 911 / 102).

When a help request is filed:
- It fans out to all on-shift providers within a configurable radius.
- The first provider to accept becomes the assigned responder.
- Live GPS is shared between help-seeker and assigned provider until the help-seeker (or the provider) marks the call resolved.

## What this project is **not** (yet)

- **Not a 102/911 replacement.** The app dials the country's emergency number (102 in Nepal) via the native dialer; it's a parallel community-alerting layer, not a Public Safety Answering Point (PSAP) substitute. Matches PulsePoint/GoodSAM precedent.
- **Not yet deployed.** Plan-complete; implementation gated on the three real-world dependencies above. Application code will live in [github.com/openrescue-org/rescue-squad](https://github.com/openrescue-org/rescue-squad).

## Layout

```
Rescue-Squad/
├── CLAUDE.md              ← Project-specific Claude Code guidance
├── README.md              ← This file
├── .gitignore
├── research/              ← Primary-source research (CLAUDE.md §8)
│   ├── 01_regulatory-and-legal-framework.md
│   ├── 02_realtime-location-and-dispatch-architecture.md
│   ├── 03_existing-apps-and-dispatch-algorithms.md
│   └── index.html         ← Auto-generated viewer
├── DESIGN.md              ← Design system tokens (color, type, spacing, motion, a11y)
├── TODOS.md               ← 54-item deferred-work catalog
├── CHANGELOG.md           ← Session-by-session change log
└── .claude/
    └── plans/             ← Gitignored. Plan + review artifacts (01-05).
```

## Git remotes

- **`origin`** — `/Users/rupeshsilwal/Work_Claude_Safe/git-repos/rescue-squad.git` (local bare remote, per workspace CLAUDE.md §5).
- **`github`** — [github.com/openrescue-org/rescue-squad](https://github.com/openrescue-org/rescue-squad) (public, live).

Push to both after each commit. The `origin` bare remote was repaired on 2026-06-16 — repointed from the dead legacy path under `Claude-safe/git-repos/` to the current workspace `git-repos/`.
