# Rescue-Squad — TODOs

Deferred-work catalog. Generated 2026-05-29 by `/plan-ceo-review` in EXPANSION mode. Items are grouped by phase. Each item has: what, why, pros, cons, effort (human team → CC+gstack), priority, dependencies.

---

## Open scope decisions (must resolve before MVP)

### TODO-1 — Pilot partner organization in Nepal — **PARTNER LOCKED: Kathmandu University Dhulikhel Hospital (KUDH)**
- **What:** Operationalize KUDH (https://dhulikhelhospital.org/) as pilot partner.
- **Why:** No real users = no real validation; MVP exit criteria require pilot medical-director sign-off.
- **Why KUDH fits exceptionally well:** academic medical center (research/publication path via KU School of Medical Sciences); outreach-center network in surrounding rural Kavrepalanchok (multi-site pilot from day 1); Department of Community Programs already trains FCHVs across multiple districts (FCHV-tier ready supply); KU Institutional Review Committee available for ethics approval; Araniko Highway road-trauma chief complaints; heavily affected by 2015 earthquake (real-world disaster-resilience relevance for v2 mesh-fallback).
- **Sub-tasks unblocked (now P1):**
  - **TODO-1a:** Sign MOU between project and KUDH.
  - **TODO-1b:** Identify KUDH medical director(s) for ICD-11 alert-tier protocol sign-off (unblocks TODO-3).
  - **TODO-1c:** Submit IRC (Institutional Review Committee) protocol to KU. Required for research-grade outcomes publication.
  - **TODO-1d:** Select pilot footprint: which 1–2 KUDH outreach centers + their FCHV catchment villages? Affects region_config region scoping and SMS/4G coverage assumptions. **OPEN — needs KUDH input.**
  - **TODO-1e:** Recruit pilot FCHV cohort (N=5–10) in chosen footprint.
  - **TODO-1f:** Identify KUDH ER point-of-contact for human-dispatcher fallback role.

### TODO-2 — Native dial-out behavior — **LOCKED: ACTION_DIAL only + long-press REQUEST confirm**
- **What:** Use `Intent.ACTION_DIAL` to open the dialer pre-filled with the region's emergency number (102 in Nepal). User taps the green call button to start the call. REQUEST itself is a 2s long-press to prevent accidental presses.
- **Rationale:** Matches PulsePoint and GoodSAM. No `CALL_PHONE` permission needed → no Play Store medical-app review friction → no per-hospital deployment paperwork when downstream operators ship the OSS code. Multigenerational Nepali household structure means bystanders are usually present; the unconscious-user collapse-after-REQUEST scenario is covered by the FCHV alert fan-out (someone responds even if the patient can't tap call), not by app auto-dialing.
- **Implementation notes:**
  - After dialer opens, show full-screen banner: "**TAP THE GREEN CALL BUTTON** / **हरियो कल बटन थिच्नुहोस्**".
  - On `ActivityNotFoundException` (no dialer installed — very rare): in-app fallback message asking the user to call the region emergency number manually.
  - Long-press threshold: 2 seconds with haptic feedback at 1s; cancellable by lifting finger before 2s.
  - The REQUEST flow still publishes MQTT + SMS-fallback in parallel (so responder fan-out doesn't depend on the user actually completing the dial).

### TODO-3 — Define alert tier matrix per ICD-11 chief complaint — P1 (UNBLOCKED via TODO-1b)
- **What:** Map every ICD-11 emergency chapter code (JA00–JA86 + others) to one of: `FCHV_ELIGIBLE`, `RED_CROSS_ELIGIBLE`, `WHO_BEC_ELIGIBLE`, `NMC_ONLY`, `ALS_ONLY`.
- **Why:** FCHV tier needs explicit alert filtering or you re-create the KATRETTER false-positive war story (research/03).
- **Pros:** Defensible dispatch logic; KUDH medical director can review and own.
- **Cons:** Requires clinical input (medical director needed before this can finalize).
- **Effort:** Human team: 1 wk clinical review + 1 day engineering; CC+gstack: 1 day eng once matrix is written.
- **Depends on:** TODO-1b (KUDH medical director identified).

---

## v1 in-scope (locked by CEO review, captured for plan-eng-review)

### TODO-4 — Region-config schema with NP+IN+BD+LK+BT+PK seed — P1
- **What:** Postgres table + seed data for emergency_number, language, civil_liability_framework, ICD-11 vs ICD-10, alert_radius_default, ack_timeout, escalation_timeout per country.
- **Effort:** CC+gstack: 1 day. **Depends on:** Phase 1 foundation.

### TODO-5 — i18n harness with Nepali, English, Hindi locales — P1
- **What:** ICU MessageFormat (`go-i18n` or `react-intl`), string extraction, locale switching, fallback chain.
- **Why:** Nepali day 1 per CEO decision. Hindi for India migrants working in Nepal + pan-South-Asia framing.
- **Effort:** CC+gstack: 2 days harness + ongoing string-add. Find a Nepali-speaking reviewer early.

### TODO-6 — WHO ICD-11 chief-complaint code embed + attribution — P1
- **What:** Embed ICD-11 emergency chapter codes; add NOTICE.txt with CC BY-ND 3.0 IGO attribution.
- **Effort:** CC+gstack: 1 day.

### TODO-7 — Crowd-sourced AED registry tables + flows — P2
- **What:** `aed_location`, `aed_verification` tables; seeker add-flow with photo; admin verify flow; public read endpoint.
- **Effort:** CC+gstack: 3–4 days. **Depends on:** Phase 2 seeker app + admin web UI.

### TODO-8 — Offline + SMS-fallback architecture — P1
- **What:** Outbound WAL on both clients; MQTT session-resume; Twilio gateway adapter; Ncell adapter as second reference impl.
- **Effort:** CC+gstack: 1 wk. **Depends on:** Phases 1–3.

### TODO-9 — Family-notification SMS — P2 (delight item, MVP-included)
- **What:** Seeker registers 2–3 emergency contacts; on REQUEST, contacts receive SMS with live GPS link.
- **Effort:** CC+gstack: 2 days. **Depends on:** TODO-8.

### TODO-10 — Voice-memo emergency description — P2 (delight item, MVP-included)
- **What:** Seeker records 10s voice memo on REQUEST; uploaded with incident; played to responder UI.
- **Why:** Low-literacy + emotional-distress robustness.
- **Effort:** CC+gstack: 2 days.

### TODO-11 — what3words Nepali address fallback — P2 (delight item, MVP-included)
- **What:** Integrate what3words SDK; show 3-word address when GPS accuracy > 30m; share-to-responder.
- **Effort:** CC+gstack: 1 day (SDK is free for emergency-services use).

### TODO-12 — Responder wellbeing PCL-5 short-form SMS — P2 (delight item, MVP-included)
- **What:** 24h post-incident SMS to responder with 3 PCL-5 short-form questions; opt-out per responder.
- **Why:** Sarkisian 2022 (research/03 §Known failure modes).
- **Effort:** CC+gstack: 2 days. **Depends on:** TODO-8 (SMS gateway).

---

## v2 (deferred, captured for the roadmap)

### TODO-13 — Disaster-mode mesh fallback (Bluetooth/LoRa) — P3
- **What:** Peer-to-peer alert delivery when cell network is down (earthquake-aftermath scenario).
- **Why:** Nepal earthquake risk is real (2015 reference); cell towers go down for weeks.
- **Effort:** CC+gstack: 3–4 wks. Significant additional dependency surface.

### TODO-14 — iOS app (Swift + same native location bridge pattern) — P3
- **What:** iOS port of seeker + provider apps. Same native-bridge architecture as Android (Swift module exposed via RN bridge).
- **Why:** Urban Kathmandu has meaningful iOS population.
- **Effort:** CC+gstack: 4–6 wks after Android proven.

### TODO-15 — HMIS-2 (Nepal Health Management Information System) integration — P3
- **What:** Push incident outcomes to Nepal Ministry of Health HMIS-2 system.
- **Why:** Real public-health-system integration; unlocks government partnership.
- **Effort:** CC+gstack: 2 wks engineering + months of MoH relationship work.

### TODO-16 — Pulsara-style hospital-handoff layer — P3
- **What:** Structured handoff to receiving hospital (chief complaint, vitals, ETA).
- **Why:** Pulsara BMJ Open 2022 evidence (research/03).
- **Effort:** CC+gstack: 2 wks. **Depends on:** Hospital partner.

### TODO-17 — ICD-10 mapping layer — P3
- **What:** Code mapping for hospitals still on ICD-10.
- **Why:** Real adoption blocker if a partner hospital is ICD-10-only.
- **Effort:** CC+gstack: 1 wk + ongoing mapping maintenance.

### TODO-18 — Multi-language beyond NP/EN/HI — P3
- **What:** Bengali, Sinhala, Urdu, Dzongkha for pan-South-Asia framing.
- **Effort:** CC+gstack: 2 days per language + translation cost.

### TODO-19 — National AED-registry partnership with NRCS — P3
- **What:** Promote the crowd-sourced AED registry to a Nepal Red Cross Society-endorsed national registry.
- **Effort:** Relationship work.

---

## Infrastructure + ops (must exist before pilot)

### TODO-20 — ARCHITECTURE.md, ADRs, OpenAPI spec, OpenRPC for MQTT — P1
- **What:** Living architecture doc; per-decision ADRs (Architecture Decision Records); OpenAPI spec for dispatch HTTP API; OpenRPC for MQTT topic conventions.
- **Why:** 1-year question: new contributor can onboard in a day.
- **Effort:** CC+gstack: 1 wk initial + ongoing as decisions land.

### TODO-21 — DESIGN.md before any UI work — P1
- **What:** Visual design system (colors, type, spacing, components, Devanagari-aware typography); accessibility commitments (≥48dp targets, TalkBack support, contrast); voice-of-the-product.
- **Effort:** CC+gstack: 3 days. **Depends on:** Recommend `/plan-design-review` first.

### TODO-22 — Day-1 observability dashboards + alerts — P1
- **What:** Per-region incidents/hr, p50/p99 ack, dispatch channel mix, error rates; alert classes per Section 8.
- **Effort:** CC+gstack: 3 days.

### TODO-23 — Runbook per alert class — P1
- **What:** `RUNBOOK.md` with one section per alert; published in repo so any operator can use.
- **Effort:** CC+gstack: 2 days.

### TODO-24 — CI dependency scanning (govulncheck + npm audit + Snyk) — P1
- **What:** Block PRs with high-severity CVEs.
- **Effort:** CC+gstack: 1 day.

### TODO-25 — Helm chart + Docker Compose + systemd unit for deploy modes — P1
- **What:** Three packaging modes for resource-constrained-to-cloud spectrum.
- **Effort:** CC+gstack: 1 wk.

### TODO-26 — Synthetic-incident smoke test in CI — P1
- **What:** Post-deploy synthetic exercise; auto-rollback on failure.
- **Effort:** CC+gstack: 2 days.

---

## Research addenda needed

### TODO-27 — Nepal regulatory stack addendum to research/01 — P1
- **What:** NTA numbering policy, Privacy Act 2018 (Nepal), Civil Code 2017 §§ on Good-Samaritan-equivalent, NMC licensing, FCHV credentialing model, Nepal Ambulance Service framework. Replace US-centric §1.1–§1.5 with Nepal-stack reality.
- **Effort:** CC+gstack: 2 days research (parallel subagent).

### TODO-28 — Nepal mobile-network reality addendum to research/02 — P2
- **What:** Ncell + NTC coverage maps; SMS reliability outside Kathmandu Valley; 4G vs 3G density; load-shedding impact on background services.
- **Effort:** CC+gstack: 1 day research.

---

## Added by eng-review outside voice (cross-model tensions deferred to TODOs)

### TODO-29 — Human-dispatcher staffing model — P0
- **What:** Define who staffs the human-dispatcher fallback role in the ack ladder. 24/7 or business hours? KUDH ER nurse cannot also be a real-time dispatch fallback for multi-village incidents.
- **Why:** Escalation ladder terminates in "a human dispatcher" with no defined operator. Single point of failure both reviews glossed over.
- **Depends on:** TODO-1f (KUDH ER point-of-contact).
- **Effort:** Human relationship work (no CC accel). 1-2 wks coordination with KUDH.

### TODO-30 — Nepal data-protection legal review — P0
- **What:** Privacy Act 2018 (Nepal) + Individual Privacy Act review before user-data architecture is finalized. Covers: server location (data residency), Twilio cross-border SMS transfer consent, AED photo storage, FCHV PII handling, audit log retention.
- **Why:** CEO + eng reviews locked architectural decisions without checking whether they're legal in Nepal. In pilot, the project IS the data controller.
- **Effort:** Legal review (no CC accel). Estimate: 2-3 wks with Nepali counsel.

### TODO-31 — Maestro + RN + native Kotlin fg-service E2E pilot scenario — P0
- **What:** Build one Maestro scenario that drives long-press REQUEST → fg-service start → ACTION_DIAL hand-off → return to app → mark resolved. Confirms Maestro can handle the hardest planned flow before Phase 1 commits the framework.
- **Why:** Eng review picked Maestro without piloting the scenario; could be a 2-wk sinkhole if Maestro can't drive fg-service + dialer hand-off.
- **Effort:** CC+gstack: 2-3 days spike.

### TODO-32 — Synthetic-to-live go-live trigger criteria + safety committee — P0
- **What:** Define what makes the system "ready for live activation." Acceptance criteria, liability transfer mechanism, safety-committee composition (KUDH medical director, KU IRC, project lead), exit ramp if first real activation causes harm.
- **Why:** MVP exit criteria say "synthetic exercises only until safety review passes" with no defined criteria. Shipping to the edge of the cliff and stopping.
- **Depends on:** TODO-1a (KUDH MOU), TODO-1b (medical director), TODO-1c (IRC).
- **Effort:** 1 wk doc work + KUDH alignment.

### TODO-33 — Solo-founder on-call rotation story — P0
- **What:** Define who gets paged when EMQX/Postgres/Twilio/FCM/dispatcher dies at 3am Kathmandu. Cannot be only the founder — that itself is a safety failure for a life-critical system. Options: bring in a co-maintainer; on-call rotation with KUDH IT team; SLA-degradation tolerance with public banner.
- **Why:** Both prior reviews assumed an operations team exists. There isn't one.
- **Effort:** Relationship work + runbook design. 1-2 wks.

### TODO-34 — Android-share survey in Nepal emergency-call originators — P1
- **What:** Validate that 95% Android share applies to people who actually originate emergency calls (often elderly, often bystanders using nearest phone). Statcounter measures browser-share, not emergency-call-originator-device-share.
- **Why:** If 30% of emergency calls come from feature phones, the Android-first decision shrinks the addressable seeker population — and SMS-originated REQUEST becomes a primary path, not a fallback.
- **Effort:** Survey with KUDH ER intake data + Nepal Telecom data. 1-2 wks.

### TODO-35 — FCHV smartphone + data-plan + charging penetration survey — P1
- **What:** Source current data on FCHV smartphone ownership, data-plan affordability, charging-during-load-shedding reality. If FCHVs are mainly SMS-reachable, the Kotlin foreground-service architecture is wasted on the FCHV tier.
- **Why:** Decision #6 built a tier on FCHVs without confirming the phone-distribution assumption.
- **Effort:** Survey with KUDH Community Programs Department. 1-2 wks.

### TODO-36 — MQTT-over-WSS through Ncell + NTC carrier-grade NAT test — P1
- **What:** Test that MQTT-over-WSS sustains a long-lived connection through both Nepali carriers' CGNAT. Some LMIC carriers idle-kill TLS at 60-180s; if true, push becomes the data path, not just wake-up, breaking §2.2 architecture.
- **Why:** Architecture assumption never validated against actual Nepal carrier middleboxes.
- **Effort:** 2-3 day spike with two Nepal SIMs.

### TODO-37 — ICD-11 CC BY-ND clause analysis vs dispatch-time filtering — P1
- **What:** Legal analysis of whether code that filters incidents by ICD-11 codes at dispatch time creates a derivative work or runs afoul of "no derivatives" clause. Worst case: cannot customize the ICD-11 tier matrix at dispatch time, which kills TODO-3.
- **Why:** CEO review noted CC BY-ND but did not analyze the dispatch-time filtering case.
- **Effort:** Legal opinion + WHO communications. 1-2 wks.

### TODO-38 — Protocol formalization v1 scope — P0 (consequence of cross-model tension #3)
- **What:** Separate `rescue-squad-protocol` repo (Apache 2.0) containing: OpenAPI spec + OpenRPC MQTT topic spec + semver policy for protocol version (separate from impl version) + governance doc (decision-making model + extension policy) + conformance test harness (any implementation can be validated against this).
- **Why:** User chose B on cross-model tension #3 — formalize protocol in v1, not v2. Apache 2.0 + GitHub repo is not a protocol; this TODO makes the protocol framing real.
- **Effort:** CC+gstack: 1-2 wks. Real artifact: conformance test harness in Go (k6 + curl-based) that runs against any spec-conformant impl.
- **Depends on:** Phase 1 (foundation) for the OpenAPI/OpenRPC definitions.
