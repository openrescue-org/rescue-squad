# Real-Time Location Sharing and Time-Critical Dispatch Architecture

## Overview

The Rescue Squad application has two roles that must be served by the same runtime: trained on-call EMS providers, and end users in a medical emergency. When a help request is filed it must fan out to **all** on-shift providers within seconds, and once a provider accepts, both phones must stream GPS continuously until the user marks the incident resolved. Because the application sits on the critical path of a medical event, every layer — location acquisition, transport protocol, push notification, background execution, geo-indexed search — has to be evaluated against a **life-critical latency and accuracy budget**, not the looser budgets that ride-hail, food-delivery, or fitness apps target.

This document collects primary-source evidence on each layer and proposes a defensible architecture. Where authoritative measurement data does not exist (notably end-to-end APNs/FCM tail latency from a trusted academic source), the gap is called out explicitly.

---

## 1. Mobile GPS Accuracy in Practice

Both iOS and Android expose a **fused** location stack: the OS combines GNSS (GPS, GLONASS, Galileo, BeiDou), Wi-Fi-based positioning, cell-tower trilateration, and inertial sensors before handing a position to the application. The accuracy of this fused estimate is highly environment-dependent.

On iOS, `CLLocation.horizontalAccuracy` is defined as "the radius of uncertainty for the location, measured in meters," with the location's latitude/longitude as the centre of that circle; **negative values mean the position is invalid and must be discarded** [Apple, *CLLocation.horizontalAccuracy*](#ref-1). Apple's `CLLocationAccuracy` constants — `kCLLocationAccuracyBestForNavigation`, `kCLLocationAccuracyBest`, `kCLLocationAccuracyNearestTenMeters`, `kCLLocationAccuracyHundredMeters`, `kCLLocationAccuracyKilometer`, `kCLLocationAccuracyThreeKilometers` — set the *desired* accuracy; Apple notes that Core Location may deliver more accurate data than requested, but raising desired accuracy "powers up additional hardware and wastes power" if not needed [Apple, *kCLLocationAccuracyBest*](#ref-2).

Android's analogue is `FusedLocationProviderClient`. Setting a `LocationRequest` with `Priority.PRIORITY_HIGH_ACCURACY` "requests a tradeoff that favors highly accurate locations at the possible expense of additional power usage" and is the documented choice for "clients showing the user's location in realtime" [Google, *LocationRequest*](#ref-3). The fused provider blends GNSS with network-derived position and IMU dead reckoning [Google, *FusedLocationProviderClient*](#ref-4).

Real-world accuracy in urban environments is substantially worse than the marketing numbers. A peer-reviewed PLOS ONE measurement study using iPhone 6 in a downtown setting reported **average horizontal accuracy of 7–13 m**, with degradation correlated to building density and tree canopy [Merry & Bettinger, 2019](#ref-5). A 2025 *Sensors* / MDPI paper on urban-canyon positioning measured baseline horizontal RMSE of **13.0 m**, dropping to 4.7 m only after applying a new signal-strength weighting model [Zhang et al., 2025](#ref-6). Multipath — GNSS signals reflecting off building façades before reaching the receiver — is the dominant error source and cannot be fully removed without dual-frequency receivers that most consumer phones do not yet ship [Xie & Petovello, 2014](#ref-7).

**Implication for Rescue Squad:** the application must treat any `horizontalAccuracy` worse than ~30 m with caution, display the uncertainty radius to the receiving provider, and **never auto-route based on a single fix**. Both clients should reject fixes with negative horizontal accuracy [Apple, *CLLocation.horizontalAccuracy*](#ref-1) and prefer fixes with `horizontalAccuracy ≤ 20 m` for routing decisions while still showing the user's best-known position.

---

## 2. Live-Location Transport Protocols

Streaming "both parties' position every N seconds" is a small, frequent, bidirectional message workload. Four candidates:

**HTTP polling.** A client repeatedly issues HTTP requests. Each request carries full TLS+HTTP overhead, so a 1 Hz update burns ~1–2 KB of headers per direction per second per client, plus TCP/TLS connection re-establishment cost when keep-alive expires. There is no standard "push" — the server cannot wake the client. Latency floor is the polling interval. **Unsuitable** for a life-critical dispatch flow.

**Server-Sent Events (SSE)** is one-way (server → client) over a long-lived HTTP/1.1 or HTTP/2 connection. Useful for streaming the provider's position to a user, but the client cannot stream back over the same channel; a separate uplink is required.

**WebSocket (RFC 6455)** provides full-duplex, framed communication over a single TCP connection initiated via an HTTP `Upgrade: websocket` handshake; ports 80 and 443 are reused so the protocol traverses corporate proxies and TLS-terminating load balancers cleanly [Fette & Melnikov, 2011](#ref-8). After the handshake, framing overhead is **2 to 14 bytes per message** depending on payload length and masking, versus hundreds of bytes per HTTP request. The protocol is an IETF Proposed Standard (STD 35 status pending) and is supported natively by both URLSession (iOS 13+) and OkHttp / Ktor on Android.

**MQTT v5 (OASIS standard, March 2019)** is a publish/subscribe broker-mediated protocol designed for "lightweight, open, simple, and designed to be easy to implement" telemetry over unreliable networks [Banks et al., 2019](#ref-9). It offers three QoS levels (0 = at most once, 1 = at least once, 2 = exactly once), session persistence across reconnects, last-will messages (useful for "provider went offline" semantics), and topic-based fan-out — naturally fitting the "broadcast to all on-shift providers" pattern [Banks et al., 2019](#ref-9). MQTT v3.1.1 is also ratified as ISO/IEC 20922:2016 [OASIS, 2018](#ref-10). The downside vs. WebSocket is the operational burden of running an HA broker (HiveMQ, EMQX, AWS IoT Core, etc.); the upside is that the broker handles fan-out, retained messages, and reconnect-with-session-state for you.

**Tradeoff summary:**

| Concern | WebSocket | MQTT v5 (over TCP or WSS) | HTTP polling |
|---|---|---|---|
| Battery (radio wakeups) | Low (1 long-lived TCP) | Low (1 long-lived TCP) | High (per-poll wake) |
| Server fan-out semantics | App-level | Built-in pub/sub | App-level |
| Reconnect / session state | App-level | Built-in (clean-start=false) | N/A |
| NAT / proxy traversal | Excellent (443/TLS) | Excellent over MQTT-over-WSS (443) | Excellent |
| Delivery guarantees | App-level | QoS 0/1/2 | At-most-once |
| Standard | RFC 6455 [Fette & Melnikov, 2011](#ref-8) | OASIS [Banks et al., 2019](#ref-9) | RFC 9110 |

**Recommendation:** MQTT v5 over WSS (MQTT framed inside a TLS WebSocket on port 443) for the live-location channel. It gives the fan-out semantics free, traverses cellular NATs and corporate proxies, and reuses the same TCP socket used for the initial accept/reject control plane.

---

## 3. Push Notification Reliability for Time-Critical Alerts

The first 911-style "fan-out" is the most sensitive moment in the system: the request must reach a provider's phone **even if the app is suspended or the screen is locked**.

**Apple Push Notification service (APNs).** The `apns-priority` header on the HTTP/2 POST to `api.push.apple.com` controls scheduling. A value of **10** means "send the push message immediately" and "is appropriate for notifications that trigger an alert, play a sound, or badge the app's icon"; a value of 5 conserves power; a value of 1 prioritises power and prevents device wake [Apple, *Sending notification requests to APNs*](#ref-11). For iOS 15+, the JSON payload's `interruption-level` of `time-sensitive` (and, separately, `critical` for life-safety) allows the notification to break through Focus and Notification Summary; the `critical` alert level requires a special entitlement granted by Apple [Apple, *UNNotificationInterruptionLevel*](#ref-12). **Apple does not publish a numeric SLA for delivery time.** The service is "best effort"; in practice, third-party benchmarks measure provider-acceptance times of ~60 ms median over the open internet, but those numbers measure only Knock-to-Apple-edge, not edge-to-device [Knock, 2024](#ref-13).

**Firebase Cloud Messaging (FCM).** FCM offers `normal` and `high` priority. "FCM attempts to deliver high priority messages immediately even if the device is in Doze mode," with "temporary access to network services and partial wakelocks" granted to the app [Google, *Doze and App Standby*](#ref-14); high priority is documented as appropriate for "time-sensitive, user visible content" such as chat messages or food-delivery updates [Google, *Set the priority of a message*](#ref-15). However, high-priority delivery is **still subject to Android's App Standby Buckets** (active / working_set / frequent / rare / restricted) on Android 9+; an app in the `restricted` bucket can have high-priority pushes deferred [Google, *Doze and App Standby*](#ref-14). Knock's benchmark measured FCM provider-acceptance at 80 ms p50 / 92 ms peak-p50 and a daily error rate of 0.00–0.06% [Knock, 2024](#ref-13).

**Are these guarantees sufficient for an emergency-alert use case?** No, not on their own. Both APNs and FCM are explicitly best-effort cloud services; neither vendor publishes an end-to-end delivery SLO, and tail latency under cellular contention can be tens of seconds. The architecturally correct response is **defence in depth**:

1. Send APNs with `apns-priority: 10` and `interruption-level: time-sensitive` (or `critical` if Apple grants the entitlement) [Apple, *Sending notification requests to APNs*](#ref-11).
2. Send FCM with `priority: "high"` and a notification payload [Google, *Set the priority of a message*](#ref-15).
3. **In parallel**, if the provider's app has an active foreground service or VoIP background mode, deliver the alert over the persistent MQTT/WSS channel — the push notification is then only a fallback to wake a backgrounded app.
4. Require the provider's client to **acknowledge receipt** within a configured timeout (e.g., 8 s). If no ack, escalate: re-push, fan out to the next ring of providers, and surface the failure to a human dispatcher.

---

## 4. Background Location on iOS and Android

An on-call provider's phone will spend most of its time with the app backgrounded. The OS rules for "wake on push, then continue streaming location" are strict.

**iOS.** The app declares the `location` and `remote-notification` `UIBackgroundModes` in `Info.plist`, and may additionally use the `voip` background mode (for a VoIP-style persistent socket, but Apple has tightened restrictions on this since iOS 13 and prefers PushKit + CallKit). With `location` mode and `allowsBackgroundLocationUpdates = true`, Core Location continues to deliver updates while the app is suspended, but the OS may throttle them to conserve battery [Apple, *Handling location updates in the background*](#ref-16). Background App Refresh must be enabled by the user; if it is off, "the system doesn't relaunch the app for any location events" [Apple, *Handling location updates in the background*](#ref-16). **Always-allow** location permission is required for true background continuation; "While Using" only delivers updates while the app is foreground.

**Android.** Continuous background location requires a **foreground service** with `foregroundServiceType="location"`, declared in the manifest, plus the `FOREGROUND_SERVICE` and `FOREGROUND_SERVICE_LOCATION` manifest permissions [Google, *Foreground service types*](#ref-17). At runtime, the user must grant `ACCESS_FINE_LOCATION` (or `ACCESS_COARSE_LOCATION`) and **separately** `ACCESS_BACKGROUND_LOCATION` — the latter is shown in a second permission prompt and is the most common failure point in production [Google, *Foreground service types*](#ref-17). On Android 14+ the foreground service type must also be declared in Play Console with a demonstration video [Google, *Foreground service types required*](#ref-18). The foreground service must show a persistent notification while running.

**Implication:** the provider onboarding flow must explicitly walk the user through enabling "Always" (iOS) and "Allow all the time" (Android) location, plus Background App Refresh on iOS, before they can go on-shift. Without those, the architecture's wake-on-push assumption breaks.

---

## 5. Existing Dispatch Architectures (Ride-Hail Reference)

Ride-hail platforms solve a structurally similar problem (request → fan-out → accept → live tracking) at planetary scale, so their published architectures are informative.

Uber's matching engine is called **DISCO** (Dispatch Optimization). The geographic substrate is **H3**, Uber's open-source hexagonal hierarchical spatial index: "H3 is a geospatial indexing system using a hexagonal grid that can be (approximately) subdivided into finer and finer hexagonal grids" with 16 resolutions; cells at resolution 9 average ~0.1 km² [Uber Engineering, 2018](#ref-19) [H3 Project, n.d.](#ref-20). For dispatch, a pickup point is mapped to its H3 cell, the **k-ring** of neighbouring cells defines the candidate set, and the matching algorithm scores candidates by ETA, repositioning cost, and acceptance probability [Uber Engineering, 2018](#ref-19).

Lyft's dispatch team has published on bipartite-matching formulations of the rider–driver assignment problem (LP relaxations, Hungarian-style assignment) and on real-time map-matching to clean noisy GPS traces before dispatch decisions [Hanguir, Lyft Engineering, 2020](#ref-21) [Douriez, Lyft Engineering, 2018](#ref-22). Lyft is reported to use Google's S2 library for spatial indexing rather than H3 [Douriez, Lyft Engineering, 2018](#ref-22).

**Implication for Rescue Squad:** EMS dispatch is a much smaller-scale problem (dozens to low-hundreds of on-shift providers in a region, not millions of drivers), but the same primitives apply: cell-index the supply (providers), k-ring around the incident to form a candidate set, fan out, and use the live-location channel for tracking after accept.

---

## 6. Geofencing and Geo-Indexed Search

Three serious options for "find all on-shift providers within X km of incident":

- **PostGIS** — the canonical open-source GIS. `ST_DWithin(geog, point, radius)` works on the `geography` type with units in **metres** and uses a GiST spatial index automatically; "ST_DWithin uses a spatial index (if available)" and an expanded bounding box internally [PostGIS Project, *ST_DWithin*](#ref-23). The right choice when the source of truth is already a relational database and the query set is "find Postgres rows within R metres of point P."
- **H3 (Uber)** — open-source C library with bindings to Python, Java, JavaScript, Go, and a PostgreSQL extension (`h3-pg`); hexagonal cells with uniform neighbour distance (every cell has 6 neighbours at equal distance, vs. 4-and-4-and-corner for squares) [Uber Engineering, 2018](#ref-19) [H3 Project, n.d.](#ref-20). Lookups become "convert incident to H3 index, take k-ring, query providers by cell ID." This is O(k) and indexed by a 64-bit integer — extremely fast.
- **S2 (Google)** — open-source C++ library with Go/Java/Python ports; projects the Earth onto a cube and recursively subdivides each face into a quadtree, yielding 64-bit Cell IDs at up to 30 levels of refinement (~1 cm at level 30) [Google Open Source, 2017](#ref-24).

For Rescue Squad's scale (on the order of hundreds of providers per metropolitan region, not millions), **PostGIS with `ST_DWithin` on a `geography(Point, 4326)` column** is the lowest-operational-burden choice: provider locations are already in the relational store of truth, the GiST index makes radius queries sub-millisecond at this cardinality, and there is no second system to operate. If the system later needs to do offline analytics (heatmaps, demand prediction), introduce H3 cell IDs as a denormalised column alongside the geography.

---

## 7. End-to-End Latency Budget

A realistic budget for "user presses HELP → first provider's phone vibrates":

| Stage | Realistic budget | Source |
|---|---|---|
| App → server, TCP/TLS already warm (LTE) | 30–70 ms | [Rohde & Schwarz, 2021](#ref-25) |
| App → server, on 5G sub-6 | 15–30 ms | [Rohde & Schwarz, 2021](#ref-25) |
| Server processing (auth, persist, query candidates by H3/PostGIS k-ring) | 20–80 ms | Engineering estimate |
| Server → APNs/FCM ingest (HTTP/2 POST) | 50–100 ms p50 | [Knock, 2024](#ref-13) |
| APNs/FCM → device (depends on connectivity, Doze, app standby bucket) | 100 ms to many seconds (no published SLA) | [Apple, *Sending notification requests to APNs*](#ref-11); [Google, *Set the priority of a message*](#ref-15) |
| Device wake + notification render | 100–500 ms | OS-dependent |
| **Total p50, both endpoints on good cellular** | **~0.5–1.0 s** | — |
| **Total p99 / pathological** | **5–30+ s** | — |

The p99 tail is the operational risk. Push services do not guarantee delivery time; in poor coverage or Doze, delays of tens of seconds are documented. **The architecture must not rely on push alone for life-critical wake-up**: persistent MQTT-over-WSS while the provider is on-shift, plus a human-dispatcher fallback when no ack arrives within ~8 s, is the only defensible design.

---

## Recommended Architecture Summary

1. **Location capture:** `CLLocationAccuracy.kCLLocationAccuracyBest` (iOS) and `Priority.PRIORITY_HIGH_ACCURACY` (Android) only after a provider accepts an incident; reject negative `horizontalAccuracy` [Apple, *CLLocation.horizontalAccuracy*](#ref-1); display uncertainty radius in the UI.
2. **Transport:** MQTT v5 over WSS to a managed broker (e.g., AWS IoT Core, HiveMQ Cloud) for both control plane (accept/reject, resolved) and live-location streams. QoS 1 for control messages, QoS 0 for the high-frequency position stream.
3. **Wake-up:** APNs with `apns-priority: 10` + `interruption-level: time-sensitive` (apply for `critical` entitlement) and FCM with `priority: "high"`, both fired in parallel. Push is the wake-up *only*; payload data is fetched from the MQTT channel after the app wakes.
4. **Background continuity:** iOS `location` + `remote-notification` background modes with "Always" permission; Android foreground service with `foregroundServiceType="location"` and `ACCESS_BACKGROUND_LOCATION`. Onboarding must walk the user through both.
5. **Geo-index:** PostGIS `geography` column with GiST index; `ST_DWithin` for radius candidate queries.
6. **Fan-out + ack:** broadcast over MQTT topic `dispatch/onshift/<region>`; require client ack within 8 s; on ack-timeout, escalate to a human dispatcher and re-broadcast to a wider ring. **Do not trust the cloud push for the life-critical guarantee.**
7. **Observability:** instrument every hop (server enqueue → APNs accept → app wake → ack) and alert on p99 ack latency above 10 s.

---

## Sources

<a id="ref-1"></a>
1. Apple Inc. *horizontalAccuracy — CLLocation | Apple Developer Documentation*. [https://developer.apple.com/documentation/corelocation/cllocation/horizontalaccuracy](https://developer.apple.com/documentation/corelocation/cllocation/horizontalaccuracy)

<a id="ref-2"></a>
2. Apple Inc. *kCLLocationAccuracyBest | Apple Developer Documentation*. [https://developer.apple.com/documentation/corelocation/kcllocationaccuracybest](https://developer.apple.com/documentation/corelocation/kcllocationaccuracybest)

<a id="ref-3"></a>
3. Google. *LocationRequest | Google Play services*. [https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest](https://developers.google.com/android/reference/com/google/android/gms/location/LocationRequest)

<a id="ref-4"></a>
4. Google. *FusedLocationProviderClient | Google Play services*. [https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderClient](https://developers.google.com/android/reference/com/google/android/gms/location/FusedLocationProviderClient)

<a id="ref-5"></a>
5. Merry, K. & Bettinger, P. (2019). "Smartphone GPS accuracy study in an urban environment." *PLOS ONE*, 14(7): e0219890. [https://doi.org/10.1371/journal.pone.0219890](https://doi.org/10.1371/journal.pone.0219890)

<a id="ref-6"></a>
6. Zhang, Y. et al. (2025). "Satellite Positioning Accuracy Improvement in Urban Canyons Through a New Weight Model Utilizing GPS Signal Strength Variability." *Sensors*, 25(15): 4678. [https://doi.org/10.3390/s25154678](https://doi.org/10.3390/s25154678)

<a id="ref-7"></a>
7. Xie, P. & Petovello, M. G. (2014). "Measuring GNSS Multipath Distributions in Urban Canyon Environments." *IEEE Transactions on Instrumentation and Measurement*. [https://doi.org/10.1109/TIM.2014.2342452](https://doi.org/10.1109/TIM.2014.2342452)

<a id="ref-8"></a>
8. Fette, I. & Melnikov, A. (2011). *RFC 6455 — The WebSocket Protocol*. Internet Engineering Task Force. [https://datatracker.ietf.org/doc/html/rfc6455](https://datatracker.ietf.org/doc/html/rfc6455)

<a id="ref-9"></a>
9. Banks, A., Briggs, E., Borgendale, K. & Gupta, R. (eds.) (2019). *MQTT Version 5.0. OASIS Standard*, 07 March 2019. [https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html](https://docs.oasis-open.org/mqtt/mqtt/v5.0/os/mqtt-v5.0-os.html)

<a id="ref-10"></a>
10. OASIS Open. *MQTT Version 5.0 is an approved OASIS Committee Specification* (2018). [https://www.oasis-open.org/2018/01/03/mqtt-v5-0-is-an-approved-oasis-committee-specification/](https://www.oasis-open.org/2018/01/03/mqtt-v5-0-is-an-approved-oasis-committee-specification/); MQTT v3.1.1 ratified as ISO/IEC 20922:2016.

<a id="ref-11"></a>
11. Apple Inc. *Sending notification requests to APNs | Apple Developer Documentation*. [https://developer.apple.com/documentation/usernotifications/sending-notification-requests-to-apns](https://developer.apple.com/documentation/usernotifications/sending-notification-requests-to-apns)

<a id="ref-12"></a>
12. Apple Inc. *UNNotificationInterruptionLevel | Apple Developer Documentation*. [https://developer.apple.com/documentation/usernotifications/unnotificationinterruptionlevel](https://developer.apple.com/documentation/usernotifications/unnotificationinterruptionlevel)

<a id="ref-13"></a>
13. Knock (2024). *Apple Push (APNS) vs Firebase FCM — Push API benchmarks*. Note: third-party benchmark, included for comparative latency measurement only. [https://knock.app/push-api-benchmarks/compare/apns-vs-fcm](https://knock.app/push-api-benchmarks/compare/apns-vs-fcm)

<a id="ref-14"></a>
14. Google. *Optimize for Doze and App Standby | Android Developers*. [https://developer.android.com/training/monitoring-device-state/doze-standby](https://developer.android.com/training/monitoring-device-state/doze-standby)

<a id="ref-15"></a>
15. Google. *Set the priority of a message | Firebase Cloud Messaging*. [https://firebase.google.com/docs/cloud-messaging/customize-messages/setting-message-priority](https://firebase.google.com/docs/cloud-messaging/customize-messages/setting-message-priority)

<a id="ref-16"></a>
16. Apple Inc. *Handling location updates in the background | Apple Developer Documentation*. [https://developer.apple.com/documentation/corelocation/handling-location-updates-in-the-background](https://developer.apple.com/documentation/corelocation/handling-location-updates-in-the-background)

<a id="ref-17"></a>
17. Google. *Foreground service types | Android Developers*. [https://developer.android.com/develop/background-work/services/fgs/service-types](https://developer.android.com/develop/background-work/services/fgs/service-types)

<a id="ref-18"></a>
18. Google. *Foreground service types are required (Android 14) | Android Developers*. [https://developer.android.com/about/versions/14/changes/fgs-types-required](https://developer.android.com/about/versions/14/changes/fgs-types-required)

<a id="ref-19"></a>
19. Uber Engineering (2018). *H3: Uber's Hexagonal Hierarchical Spatial Index*. [https://www.uber.com/us/en/blog/h3/](https://www.uber.com/us/en/blog/h3/)

<a id="ref-20"></a>
20. H3 Project. *H3 Documentation*. [https://h3geo.org/docs/](https://h3geo.org/docs/) and source repository [https://github.com/uber/h3](https://github.com/uber/h3)

<a id="ref-21"></a>
21. Hanguir, O. (2020). *Solving Dispatch in a Ridesharing Problem Space*. Lyft Engineering. [https://eng.lyft.com/solving-dispatch-in-a-ridesharing-problem-space-821d9606c3ff](https://eng.lyft.com/solving-dispatch-in-a-ridesharing-problem-space-821d9606c3ff)

<a id="ref-22"></a>
22. Douriez, M. (2018). *A New Real-Time Map-Matching Algorithm at Lyft*. Lyft Engineering. [https://eng.lyft.com/a-new-real-time-map-matching-algorithm-at-lyft-da593ab7b006](https://eng.lyft.com/a-new-real-time-map-matching-algorithm-at-lyft-da593ab7b006)

<a id="ref-23"></a>
23. PostGIS Project. *ST_DWithin — PostGIS Reference*. [https://postgis.net/docs/ST_DWithin.html](https://postgis.net/docs/ST_DWithin.html)

<a id="ref-24"></a>
24. Google Open Source (2017). *Announcing the S2 Library: Geometry on the Sphere*. [https://opensource.googleblog.com/2017/12/announcing-s2-library-geometry-on-sphere.html](https://opensource.googleblog.com/2017/12/announcing-s2-library-geometry-on-sphere.html); reference site [https://s2geometry.io/](https://s2geometry.io/); source [https://github.com/google/s2geometry](https://github.com/google/s2geometry)

<a id="ref-25"></a>
25. Rohde & Schwarz (2021). *5G NR and LTE latency analysis in a public network*. Measurement study using QualiPoc on commercial 5G NSA networks. Vendor-published measurement study; included for empirical RTT figures; cross-reference with Hassan et al., "An In-Depth Measurement Analysis of 5G mmWave PHY Latency," *ACM PAM 2023*, [https://www-users.cse.umn.edu/~fengqian/paper/5glatency_pam23.pdf](https://www-users.cse.umn.edu/~fengqian/paper/5glatency_pam23.pdf). [https://www.rohde-schwarz.com/uk/solutions/critical-infrastructure/mobile-network-testing/stories-insights/5g-nr-and-lte-latency-analysis-in-a-public-network_255963.html](https://www.rohde-schwarz.com/uk/solutions/critical-infrastructure/mobile-network-testing/stories-insights/5g-nr-and-lte-latency-analysis-in-a-public-network_255963.html)
