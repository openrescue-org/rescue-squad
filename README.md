# Rescue-Squad

A two-sided mobile app that connects people in medical emergencies with nearby on-call EMS providers.

## Status

**Research + planning phase.** No code yet. See `research/` for primary-source background and `.claude/plans/` for the current implementation plan and open scope decisions.

## Concept

Two user roles:
1. **EMS provider** — a trained emergency medical service provider who can go "on call" (on shift) and accept help requests.
2. **Help-seeker** — anyone experiencing or witnessing a medical emergency. They request help with one tap (or the equivalent of dialling 911 / 102).

When a help request is filed:
- It fans out to all on-shift providers within a configurable radius.
- The first provider to accept becomes the assigned responder.
- Live GPS is shared between help-seeker and assigned provider until the help-seeker (or the provider) marks the call resolved.

## What this project is **not** (yet)

- **Not a 911 replacement.** Building a system that supplants the Public Safety Answering Point (PSAP) is a multi-year, multi-jurisdictional regulatory project. The default scope is a **community-responder augment** (in the spirit of PulsePoint / GoodSAM), not a replacement.
- **Not a deployed product.** This is a research + planning artifact. Implementation begins only after the user has resolved the scope decisions surfaced in `.claude/plans/01_initial-plan.md`.

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
└── .claude/
    └── plans/             ← Gitignored. Implementation plans.
        └── 01_initial-plan.md
```

## Git remote

`git-repos/rescue-squad.git` (local bare remote, per workspace CLAUDE.md §5).
