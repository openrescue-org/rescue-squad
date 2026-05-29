# Rescue-Squad — Design System

**Version:** 0.1 (starter minimums)
**Generated:** 2026-05-29 via `/plan-design-review`
**Status:** Locked for v1 implementation. Full design system pass deferred to `/design-consultation`.

This document is the design source-of-truth for the Rescue-Squad seeker app, provider app, and admin web. Every UI decision aligns against the tokens here. Update via PR with reasoning.

---

## 1. Brand

- **Working product name:** Rescue-Squad (repo name as placeholder; real brand name deferred to post-pilot per CEO + design review).
- **Logo:** None. Until a real brand decision lands, the app icon is the Material Symbols `emergency` glyph rendered on the primary red surface. Avoid AI-generated logos.
- **Voice:** Calm, factual, never alarmist. Match Nepal Red Cross visual lineage.
- **Tone (NE):** Direct, respectful (use ने. respectful pronouns: तपाईं, not तिमी).
- **Tone (EN):** Plain English, no jargon, no "we" marketing language.

---

## 2. Color

All colors are Material 3 token-style. CSS variables in web/admin; theme tokens in RN.

| Token | Light | Dark | Usage |
|---|---|---|---|
| `--color-primary` | `#D32F2F` | `#FF6659` | Emergency CTA, primary action |
| `--color-on-primary` | `#FFFFFF` | `#000000` | Text on primary surfaces |
| `--color-background` | `#FFFFFF` | `#121212` | App background |
| `--color-surface` | `#F5F5F5` | `#1E1E1E` | Cards, banners |
| `--color-text-primary` | `#212121` | `#E0E0E0` | Body, headings |
| `--color-text-secondary` | `#757575` | `#9E9E9E` | Captions, hints |
| `--color-success` | `#2E7D32` | `#66BB6A` | On-scene / resolved positive |
| `--color-warning` | `#F57C00` | `#FFB74D` | Degraded mode / SMS fallback |
| `--color-error` | `#C62828` | `#EF5350` | Failed states |
| `--color-divider` | `#E0E0E0` | `#424242` | Lines |

**Rules:**
- Single accent color (red). No secondary/tertiary palette.
- Banned: purple/violet gradients, multi-color decorative blobs.
- WCAG AA contrast required everywhere: ≥4.5:1 body text, ≥3:1 large text. Verify in CI.
- Visited vs unvisited link distinction preserved everywhere.

**Dark mode scope:** Provider app honors Android system theme (dark/light/auto). Seeker app is **light-only** (used briefly, in panic, where high-contrast white-on-red works best). Admin web honors system theme.

---

## 3. Typography

| Script | Font | License | Why |
|---|---|---|---|
| Devanagari (Nepali, Hindi) | **Noto Sans Devanagari** | SIL OFL 1.1 | Reliable Android 7+ rendering, Apache-compatible, Google-stewarded |
| Latin (English) | **Inter** | SIL OFL 1.1 | Excellent legibility, free, ships on most devices |

**Type scale (sp / px):**

| Token | Size | Weight | Line-height | Use |
|---|---|---|---|---|
| `display` | 32 | 700 | 1.2 | Brand mark, hero numerals |
| `headline` | 24 | 700 | 1.3 | Screen titles |
| `title` | 18 | 600 | 1.4 | Card titles, prompts |
| `body` | 16 | 400 | 1.5 | Body text, button labels |
| `label` | 14 | 500 | 1.4 | Form labels, hints |
| `caption` | 12 | 400 | 1.3 | Diagnostics, fine print |

**Rules:**
- Never use default font stacks (system, sans-serif). Always specify Noto Sans Devanagari or Inter.
- Devanagari and Latin in the same UI: same line-height, slightly larger Devanagari size (e.g. body Devanagari = 17sp, body English = 16sp) for visual parity. Test on real Nepali speakers.
- Conjunct rendering must be correct on Android 7+. Test ट्र, क्ष, ज्ञ.
- Respect system text scaling up to 200%.

---

## 4. Spacing

Base unit: **4dp**. Scale: `4, 8, 12, 16, 24, 32, 48, 64`.

| Token | dp | Use |
|---|---|---|
| `space-xs` | 4 | Icon-to-text, tight padding |
| `space-sm` | 8 | Inner padding |
| `space-md` | 16 | Default padding, gap |
| `space-lg` | 24 | Section gap |
| `space-xl` | 32 | Major sections |
| `space-2xl` | 48 | Hero / above CTA |
| `space-3xl` | 64 | Top-of-screen breathing room |

---

## 5. Touch targets

| Element | Min size | Recommended |
|---|---|---|
| Standard interactive | 48dp | — |
| Emergency CTA (REQUEST, Accept) | 48dp | 64dp |
| Icon-only button | 48dp tappable, 24dp icon | — |
| Form input | 48dp height | — |

**Rule:** never sacrifice touch-target size for visual density. Emergency context demands generous targets.

---

## 6. Shape

| Element | Radius |
|---|---|
| Button | 8dp |
| Input field | 4dp |
| Card | 12dp |
| Circular (FAB, profile photo) | 50% (fully circular) |
| Bottom-sheet | 16dp top corners only |

**Banned:** uniform bubbly radius on every element. Vary by purpose.

---

## 7. Elevation

Material 3 surface tones (color shift), not drop shadows.

| Level | Use |
|---|---|
| 0 | Background |
| 1 | App bar, default surface |
| 3 | Cards, dialogs |
| 6 | FAB |
| 8 | Top app bar when scrolled |

**Banned:** decorative drop shadows. Use elevation tones.

---

## 8. Motion

| Token | Duration | Easing | Use |
|---|---|---|---|
| `motion-fast` | 150ms | standard | Hover/press state |
| `motion-standard` | 250ms | standard | Default screen transitions |
| `motion-emphasized` | 400ms | emphasized | State changes (e.g. dispatching → accepted) |
| `motion-long-press` | 2000ms | linear | REQUEST button progress ring |

**Rules:**
- Respect `prefers-reduced-motion` system setting.
- No looping decorative animation. Animation must convey progress, state, or attention — never decoration.
- Long-press progress ring: linear fill clockwise, haptic feedback at 1000ms (medium intensity), fires at 2000ms (strong intensity).

---

## 9. Iconography

- **Library:** Material Symbols (Outlined) — Apache 2.0 license.
- **Sizes:** 16dp small, 24dp default, 32dp primary action, 48dp emergency icon (REQUEST button center).
- **Treatment:** Monochrome. Use color tokens, not painted icons.
- **Banned:** icons in colored circles (SaaS starter look), painted/illustrated icons, emoji-as-icon except `📍 location` and `📞 phone` (universally understood).

---

## 10. Maps

- **Provider (v1):** **MapLibre Native + OpenStreetMap tiles** (locked decision, reverted from Google Maps SDK after user clarification: any paid-subscription or billing-card requirement disqualifies a default for the OSS reference impl, even if the per-deploy cost is zero in the free tier). MapLibre is BSD-2 license — Apache-compatible. OSM tiles are ODbL-licensed and free.
- **Pluggable architecture:** wrap behind `MapProvider` interface so downstream operators with existing Google Cloud relationships can swap in **Google Maps SDK** if they prefer polished maps + offline tile caching. Reference adapter for Google Maps ships in the same repo as a second example.
- **Tile server:** v1 uses the public OpenStreetMap tile server. For production deploys, operators self-host a tile server (e.g. `osm-tile-server` Docker image) — Helm chart documents this option. Document polite tile-usage policy (caching, attribution).
- **Marker design:** seeker = red pin with pulsing halo; responder = blue chevron pointing in direction of travel; AED = green AED glyph in circle.
- **Map style:** OSM Bright or Liberty style with custom JSON to mute saturation 20% (reduce visual noise during incident).
- **Trade-off accepted:** Nepal OSM coverage is good in Kathmandu Valley + improving in Kavrepalanchok (KUDH catchment); rural areas like Dolpa have sparser data. v2 may need community OSM contributions for full coverage.

---

## 11. Accessibility

- **Touch targets:** ≥48dp (Material), ≥56dp for emergency CTAs.
- **Color contrast:** WCAG AA. Automated CI check via `axe-core` (RN equivalent).
- **TalkBack:** all interactive elements have `contentDescription` (in user's language).
- **Dynamic font scaling:** support system text size up to 200%.
- **No timing-dependent UI** except the 2s long-press, which uses haptic + visual progress as equivalent-alternative channels.
- **Audio prompts:** captioned + visual equivalent.
- **High-contrast mode:** respect Android "Color correction" + "Dark theme."

---

## 12. Layout patterns

**Seeker home screen:** Single dominant CTA centered, banner top, secondary links bottom. No bottom nav (single-screen app). No hero image.

**Provider home screen:** Top: on-call toggle. Middle: today's stats. Bottom: diagnostics strip (battery, GPS, service status). Hamburger top-left for settings.

**Active incident (both apps):** Full-screen map. Bottom sheet with incident details, status, primary action (Accept, On scene, Resolved). Top bar with seeker name + chief complaint.

**Admin web:** Standard left-nav web app, table-heavy. Material 3 web tokens.

---

## 13. Microcopy guidelines

- Nepali strings MUST be reviewed by ≥2 native speakers before shipping (tracked as TODO-39).
- English strings: plain, direct, no jargon. Reading level ~grade 6.
- No "Welcome to Rescue-Squad" hero text. No "Loading…" without progress indicator.
- Empty states: warmth + primary action + context.
- Error messages: what happened + what to try + alternate path.

Examples:
- ✅ "No responders nearby — call 102 now." / "जवाफदाता उपलब्ध छैन — १०२ कल गर्नुहोस्।"
- ❌ "Error: ENOENT" / "Something went wrong"

---

## 14. AI-slop blacklist (active enforcement)

Banned patterns. CI lint will flag violations:

1. Purple/violet/indigo gradients.
2. 3-column feature grids.
3. Icons in colored circles as decoration.
4. Centered everything (use left-align except CTAs).
5. Uniform bubbly border-radius on every element.
6. Decorative blobs, floating circles, wavy SVG dividers.
7. Emoji as design elements (except `📍 📞` for universal utility).
8. Colored left-border on cards.
9. Generic hero copy ("Welcome to…", "Unlock the power of…").
10. Cookie-cutter section rhythm.

---

## 15. Open questions (tracked in TODOS.md)

- TODO-39: Nepali microcopy native-speaker review pipeline.
- TODO-40: Full `/design-consultation` pass to expand DESIGN.md (brand identity, illustration style, motion principles).
- TODO-41: Mockup generation for ≥3 screens (deferred until OpenAI API key + brand direction firms up).
- TODO-42: Real brand name (deferred to post-pilot per design review).
- TODO-43: App icon (deferred until brand name lands).
