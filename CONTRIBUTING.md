# Contributing to Rescue-Squad

Thanks for thinking about contributing. Rescue-Squad is an open-source emergency-dispatch protocol for South Asia, currently in plan-complete / pre-implementation state. Pilot partner: [Kathmandu University Dhulikhel Hospital (KUDH)](https://dhulikhelhospital.org/).

This document tells you the most useful things you can do depending on what you bring.

## State of the project

- **Plan:** four full reviews passed (CEO, Eng, Design, DX). See `.claude/plans/` and `TODOS.md` for full context.
- **Code:** none yet. Implementation is gated on three real-world dependencies — KUDH MOU, Nepal data-protection legal review, solo-founder on-call rotation. See `TODOS.md` (54 items).
- **Design system:** locked in [DESIGN.md](DESIGN.md).
- **Research:** 91 primary sources across regulatory, real-time architecture, and existing-systems literature. See `research/`.

## The single most useful contribution right now

**Read [`research/`](research/) and tell us where we got the regional context wrong.** The plan locks Nepal-first with pan-South-Asia framing (NP, IN, BD, LK, BT, PK). If you have ground-truth knowledge of any of these health systems, emergency-services landscapes, regulatory regimes, or community-responder programs, open a [GitHub Discussion](https://github.com/openrescue-org/rescue-squad/discussions) and tell us what we missed.

## Ways to contribute, in order of present usefulness

### 1. Country pack input (highest leverage, pre-code)

We're designing for 6 countries. Each needs a region pack: emergency number, language(s), regulator, civil-liability framework, AED density assumption, ICD-11 / ICD-10 status. If you have direct knowledge of your country's emergency-services landscape, file a **Country Pack Request** issue (template provided) and tell us what's specific to your jurisdiction.

### 2. Pre-implementation review

Read `.claude/plans/` (gitignored locally but visible in the commit messages and CHANGELOG.md), or read [research/](research/), or read [DESIGN.md](DESIGN.md), and file issues on anything that looks wrong, missing, or weakly justified. Pre-implementation feedback is 10x cheaper to act on than post-code feedback.

### 3. Nepali microcopy review

When the seeker / provider apps land, every Nepali string needs review by ≥2 native speakers. If you read and write Nepali at native level and are willing to be one of those reviewers, comment on [TODO-39 in TODOS.md](TODOS.md) or open a Discussion.

### 4. Code contributions (once implementation begins)

When the first code lands (after KUDH MOU and Nepal legal review):

- **Fix a bug:** find an issue tagged `good first issue` or `help wanted`.
- **Add a feature:** discuss in an issue first; we'd rather say no early than say no after you've written it.
- **Add a country pack:** the highest-leverage code contribution. We'll publish the country-pack template + walkthrough when it exists.
- **Improve docs:** Diátaxis-framework docs (tutorials / how-tos / reference / explanation). Doc PRs are reviewed quickly.

## Local development (when code exists)

```
git clone https://github.com/openrescue-org/rescue-squad.git
cd rescue-squad
make demo     # Docker Compose: dispatcher + Postgres+PostGIS + Mosquitto + seed data + synthetic incident
make test     # Full test suite
make lint     # golangci-lint + ESLint + Prettier + tsc strict
```

Stack: Go (Chi + net/http stdlib) backend, PostgreSQL + PostGIS, Mosquitto MQTT, React Native + native Kotlin module for foreground service + background location, Atlas declarative schema migrations.

## Commit message format

We use [Conventional Commits](https://www.conventionalcommits.org/):

```
feat(dispatch): add tier-aware filter for FCHV-eligible incidents

Filters incoming alerts by chief-complaint code against the ICD-11 tier
matrix before fan-out, so FCHV responders are not alerted to ALS-level
incidents per KUDH medical-director protocol.

Closes #42
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `perf`, `ci`, `build`.

CHANGELOG.md is auto-generated from these via `git-cliff`.

## PR workflow

1. Fork the repo (or branch directly if you're a maintainer).
2. Create a branch: `feat/short-description`, `fix/short-description`, `docs/...`.
3. Make changes, including tests for new behavior and a CHANGELOG entry if user-visible.
4. Run `make lint && make test` locally before pushing.
5. Push and open a PR. Fill out the PR template.
6. A maintainer will review within 7 days (target). If we don't, ping us in [Discussions](https://github.com/openrescue-org/rescue-squad/discussions).
7. Squash-merge when approved.

## Code of Conduct

This project follows the [Contributor Covenant v2.1](CODE_OF_CONDUCT.md). Be kind, be precise, assume good faith.

## Security

Found a security issue? **Please do not file a public issue.** See [SECURITY.md](SECURITY.md) for private-disclosure process.

## Governance (interim)

This is a single-maintainer project at pre-implementation stage. Decisions are documented in CHANGELOG.md and `.claude/plans/` review artifacts. As the project grows, governance will move toward a steering-committee model documented in `rescue-squad-protocol/GOVERNANCE.md` (TODO-38). If you want to be part of that conversation, open a Discussion.

## License

By contributing, you agree your contributions will be licensed under the [Apache License 2.0](LICENSE) (matching the project license).
