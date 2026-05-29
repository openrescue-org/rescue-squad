# CLAUDE.md — Rescue-Squad

Project-specific guidance for Claude Code sessions in this directory. The workspace-level `Claude-safe/CLAUDE.md` still applies; this file adds project-specific rules.

## What this project is

A two-sided medical-emergency dispatch app:
- **EMS providers** go on call and receive fanned-out help requests.
- **Help-seekers** request help with one tap; first accepting provider is assigned; live GPS is shared until resolved.

See `README.md` for the full concept and `.claude/plans/01_initial-plan.md` for the current plan and open scope decisions.

## Status

**Pre-implementation.** Research + planning only. Direction locked by `/plan-ceo-review` (see `.claude/plans/02_ceo-review-decisions.md`):

- **Approach:** Open-source community-responder protocol (Apache 2.0 licensed reference implementation).
- **Geography:** Nepal first, with pan-South-Asia protocol framing (NP, IN, BD, LK, BT, PK seeded in region_config).
- **Language:** Nepali day 1; English fast-follow; Hindi shortly after.
- **Platform:** Android-first (React Native + native Kotlin bridge for foreground service + background location). iOS is post-MVP.
- **Stack:** Go backend, EMQX MQTT broker (self-host), PostgreSQL + PostGIS, Twilio SMS default + Ncell adapter example.
- **Responder tiers:** NMC-licensed pros + Female Community Health Volunteers (FCHV) + Red Cross first-aiders + WHO BEC-trained.
- **In v1:** WHO ICD-11 chief-complaint coding, offline + SMS-fallback architecture, crowd-sourced AED registry.

**Pilot partner (locked):** Kathmandu University Dhulikhel Hospital (KUDH) — https://dhulikhelhospital.org/. Kavrepalanchok district. Multi-site outreach-center network + FCHV training pipeline + KU IRC research pathway.

**Native dial-out (locked):** `Intent.ACTION_DIAL` (not ACTION_CALL) + 2s long-press REQUEST gesture. No CALL_PHONE permission. Banner instructs the user to tap the green call button in Nepali + English. Matches PulsePoint/GoodSAM. Unconscious-user case handled by FCHV alert fan-out, not by auto-dialing.

Open scope blockers before MVP exit: KUDH outreach-center footprint (TODO-1d, needs KUDH input), KUDH IRC approval (TODO-1c, KUDH process), ICD-11 alert-tier matrix (TODO-3, unblocked once TODO-1b assigns KUDH medical director). See `TODOS.md`.

**Do not scaffold app code yet.** Plan-eng-review must complete first.

## Hard constraints

### Privacy
- This app would handle **PHI** under HIPAA the moment a US user invokes it. Treat GPS-linked health calls as PHI by default.
- Never bake real patient data, real responder names, or real PSAP numbers into source, fixtures, or tests. Use synthetic data only.
- Do not commit `.env`, `private/`, `output/`, signing certificates, API keys, or push-notification secrets. `.gitignore` already covers these.

### Safety
- This is a **life-critical** application. The §11 coding-behavior guidelines (workspace CLAUDE.md) are not optional here — especially §11.4 (goal-driven execution with verifiable success criteria). Every dispatch-path change needs a test.
- Do not add features beyond what is explicitly scoped in the active plan. A bug fix is a bug fix; a "while I'm here" refactor of the dispatch core is not.

### Regulatory posture
- The default assumption is: **not a PSAP, not a licensed EMS agency dispatcher.** Any code path that would suggest otherwise (e.g. "we have transferred your 911 call") must be explicitly approved by the user.
- Disclaimers ("This is not 911. Call 911 if you need official emergency services.") are a product requirement, not a nice-to-have.

## Research

All research lives in `research/` and follows workspace CLAUDE.md §8:
- Numbered prefix (`01_`, `02_`, ...).
- Inline citations + `## Sources` section with anchored entries.
- Primary sources only (peer-reviewed, RFCs, official agency publications).
- Rebuild the viewer after any change: `cd /Users/rupeshsilwal/Claude-safe && python3 build-research-viewer.py`.

## Plans

Plans live at `Rescue-Squad/.claude/plans/` (gitignored, per workspace CLAUDE.md §10). Never write plan artifacts to the workspace-root `.claude/plans/`.

## Git remote

`git-repos/rescue-squad.git`. Standard workflow per workspace CLAUDE.md §5 (test before commit; commit per unit of work; push after each commit; never force-push).

## Commands

_None yet — pre-implementation._ This section will be filled in once the stack is chosen and `bash run.sh` (or equivalent) exists.
