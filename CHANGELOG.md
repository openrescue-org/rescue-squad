# Changelog

All notable changes to Rescue-Squad. Format: most recent at top.

## 2026-06-16 — Bare remote renamed `origin` → `local`

- **`git remote rename origin local`:** the local bare remote is now named
  `local` (was `origin`), matching the naming convention now used across all
  subprojects (UTI-scripts, Delirium, ED-Management-Platform). URL unchanged:
  `/Users/rupeshsilwal/Work_Claude_Safe/git-repos/rescue-squad.git`. Branch
  tracking was updated automatically by the rename. `github` remote unchanged.
- **Docs synced (`README.md`, `CLAUDE.md`):** all `origin` references updated to
  `local`.

## 2026-06-16 — Origin repointed to shared git-repos; remote docs synced

- **`origin` set to the canonical shared bare-repo location:** `/Users/rupeshsilwal/Work_Claude_Safe/git-repos/rescue-squad.git` — the workspace-group `git-repos/` that holds every subproject's bare remote (aws, games, lean-proofs, taxes-2025, …). This supersedes the `Work_Claude_Safe/Exploratory/git-repos/...` path noted in the prior entry; the bares are shared at the `Work_Claude_Safe/` root, not per-workspace. Local, `origin`, and `github` are all in sync.
- **Docs synced (`README.md`, `CLAUDE.md`):** "Git remote" → "Git remotes"; both now document `origin` (local bare) + `github` (public) and a push-to-both workflow.

## 2026-06-16 — Community-health files + origin remote repair

- **Community-health files (`1b27478`):** added `CONTRIBUTING.md` and `.github/` (PR template + bug/feature/country-pack issue forms + config) for OSS contributor onboarding. Pushed to both `origin` (local bare) and `github` (`openrescue-org/rescue-squad`).
- **Fixed stale `origin` remote URL:** `origin` pointed at the old `/Users/rupeshsilwal/Claude-safe/git-repos/...` path (workspace has since moved to `Work_Claude_Safe/Exploratory/`), which broke the push. Repointed to `/Users/rupeshsilwal/Work_Claude_Safe/Exploratory/git-repos/rescue-squad.git`.

## 2026-05-29 — Project scaffold + all four plan reviews clean

Scaffolded the project and ran the full gstack plan-review pipeline (CEO, Eng, Design, DX) to lock direction before any application code is written.

- **Initial commit (`a8c5b75`):** project skeleton + 3 research docs (91 primary sources across regulatory/legal, real-time location architecture, existing apps + dispatch algorithms). CLAUDE.md + README.md + .gitignore + bare git remote at `git-repos/rescue-squad.git`.
- **CEO review (`d531def`):** SCOPE EXPANSION mode. Locked 12 of 15 scope decisions. Accepted 4 cathedral-tier expansions (pan-South-Asia protocol framing, offline+SMS first-class architecture, FCHV/volunteer responder tier, crowd-sourced AED registry). Direction: Approach 3 (open-source community-responder protocol, Apache 2.0), Nepal-first, Android-first via React Native + native Kotlin bridge for foreground service + background location, Go backend, Twilio SMS default + Ncell adapter example.
- **KUDH pilot partner (`79fec19`):** TODO-1 resolved. Kathmandu University Dhulikhel Hospital (KUDH, https://dhulikhelhospital.org/) locked as pilot partner. Unblocks TODO-3 (alert tier matrix needs KUDH medical director). New sub-tasks: MOU, IRC, medical director, outreach-center footprint, FCHV recruitment, ER point-of-contact.
- **ACTION_DIAL lock (`0117d34`):** TODO-2 resolved. `Intent.ACTION_DIAL` (not ACTION_CALL) + 2s long-press REQUEST confirm. Matches PulsePoint/GoodSAM. No CALL_PHONE permission → no Play Store medical-app review friction → no per-hospital deployment paperwork. Unconscious-user case covered by FCHV alert fan-out.
- **Eng review (`985b1e0`):** 14 stack decisions locked (Chi router + net/http stdlib, Mosquitto default + EMQX upgrade option, Atlas declarative migrations, stdlib `testing` only, Maestro RN E2E, lefthook hooks, golangci-lint, k6 load, toxiproxy chaos, pgregory.net/rapid property tests, testcontainers-go, distribution via GitHub Releases + ghcr.io + artifacthub.io + Google Play + APK direct-download). Outside-voice subagent surfaced 8 oversights + 5 assumption risks + 3 strategic miscalibrations + 1 cut recommendation; user chose B (cathedral) on all 4 cross-model tensions. 10 new TODOs (29-38). Test plan written.
- **Design review (`d8cb31e`):** Starter DESIGN.md committed at project root. 4 user-locked decisions (brand name deferred to post-pilot, MapLibre+OSM default with pluggable MapProvider interface, dark mode provider-only seeker-light-only, starter DESIGN.md minimums vs full /design-consultation). 10 auto-locks (Material Red 700 primary, Noto Sans Devanagari + Inter typography, 4dp spacing scale, ≥48dp touch targets, Material Symbols Outlined icons, etc.). 5 new TODOs (39-43). Overall design score 4/10 → 8/10.
- **DX review (`aa2c7f2`):** DX EXPANSION mode. Pan-South-Asia adopter persona, Champion TTHW tier (<2 min hosted sandbox + <5 min `make demo`). 8 passes complete (3/10 → 8/10 overall). GitHub org locked as `github.com/openrescue-org/rescue-squad` (registered 2026-05-29 after `openrescue` base name was unavailable). Community channel: GitHub Discussions. Magical moment: `make demo` + hosted sandbox both ship v1. Diátaxis docs framework adopted. Tier 3 (Stripe-style) HTTP errors + Tier 1 (Elm-style) CLI errors. 11 new TODOs (44-54). v1 timeline reset to 13-17 wks CC+gstack (was 10-14).

**State of play at session end:**
- All 4 plan reviews CLEAR.
- 54 TODOs catalogued in `TODOS.md`.
- Plan + review artifacts in `.claude/plans/01-05_*.md` (gitignored per workspace CLAUDE.md §10).
- `DESIGN.md` (v0.1 starter) committed at project root.
- 7 P0 blockers remain — all are partnership/legal/ops work, not engineering: KUDH MOU + Nepal data-protection legal review + solo-founder on-call rotation + 4 others tied to KUDH sub-tasks.
- `github.com/openrescue-org/` org registered (after `openrescue` base name was unavailable; `-org` suffix per Linux-Foundation/nodejs-org precedent).
- **End-of-session sync + GitHub publish (`3ab5348`, `4872a97`):** updated all org references to `openrescue-org`; added Apache 2.0 LICENSE + NOTICE.txt (covers ICD-11 CC BY-ND attribution + planned third-party deps + acknowledgements to PulsePoint/GoodSAM/HeartRunner/SAMBA/HartslagNu/KATRETTER + KUDH partnership). Repo published live to https://github.com/openrescue-org/rescue-squad with 10 commits. Topics set: emergency-services, ems, nepal, south-asia, open-source, dispatch, community-responder, healthcare, global-health, ohca. Issues + Discussions enabled.
- Next concrete actions: send KUDH outreach to start TODO-1a; engage Nepali counsel for TODO-30; recruit a co-maintainer for the on-call rotation gap (TODO-33).
