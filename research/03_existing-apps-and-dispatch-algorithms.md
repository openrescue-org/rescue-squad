# Existing Community-Responder / EMS-Alerting Apps and Dispatch Algorithms

*2026-05-29. Background research for the Rescue-Squad app design. The app pairs trained on-call EMS providers with end users who file emergency help requests; on filing, the request fans out to all on-shift providers and both sides share live GPS until resolution. This document surveys the prior art (PulsePoint, GoodSAM, HeartRunner, Stockholm/SAMBA, KATRETTER, Echo112/what3words, and the agency-internal class of Active911, Tablet Command, IamResponding, Pulsara), the peer-reviewed evidence on whether such systems save lives, the dispatch-algorithm literature (closest-unit vs response-time prediction vs fan-out), and documented failure modes (false positives, responder wellbeing, off-duty preparedness).*

---

## Overview

Two product categories already exist in this space. The first is **citizen-responder / crowd-sourced bystander activation**, where a public-safety answering point (PSAP) or computer-aided dispatch (CAD) system pushes a nearby suspected out-of-hospital cardiac arrest (OHCA) to lay or off-duty professional volunteers via a smartphone app (PulsePoint, GoodSAM, HeartRunner, the Stockholm system, KATRETTER, the Amsterdam HartslagNu system). The second is **agency-internal first-responder dispatch**, where a fire/EMS agency replaces or augments pagers and MDTs with a mobile app that delivers calls to its own personnel (Active911, Tablet Command, IamResponding, Pulsara). The Rescue-Squad concept sits between these: a closed roster of trained on-call providers (like the agency category) but fan-out alerting on a per-incident basis (like the citizen category). The peer-reviewed literature on these systems is the most directly relevant evidence base for design choices.

The headline finding from the OHCA literature: smartphone-activated volunteer response has now been shown in randomized and large observational studies to **roughly double bystander CPR rates and improve 30-day survival** when responders arrive before EMS [Ringh et al., 2015](#ref-1), [Smith et al., 2022](#ref-2), [Andelius et al., 2020](#ref-3), [Stieglis et al., 2022](#ref-4). The dispatch-algorithm literature consistently finds that **simple closest-unit dispatch is provably suboptimal** under realistic demand uncertainty, but practical systems still use variants of it because the data infrastructure for response-time prediction is hard to maintain [Jagtenberg et al., 2017](#ref-13).

## PulsePoint Respond (United States)

PulsePoint Respond is operated by the PulsePoint Foundation, a US 501(c)(3). The app integrates directly with the local PSAP's CAD system: when a 9-1-1 call is classified by the dispatcher as a suspected cardiac arrest in a public location, PulsePoint pushes a notification to nearby app users along with the address of the nearest registered AED [PulsePoint Foundation, 2024](#ref-5). The Foundation maintains off-the-shelf interfaces to most major US CAD vendors and publishes an implementation guide for agencies [PulsePoint Foundation, 2024 (implementation)](#ref-6).

PulsePoint segments responders into four classes [PulsePoint Foundation, 2024 (responder types)](#ref-7):

1. **Public CPR Responders** — community members trained in CPR/AED, alerted only to **public-location** OHCAs.
2. **Registered CPR Responders** — agency-invited community members (medical professionals, CERT, public-safety retirees) alerted to all locations including private residences.
3. **Professional Responders** — active firefighters, paramedics, law-enforcement officers; alerted to all locations whenever off-duty within their jurisdiction.
4. **Verified Responder** — an invitation-only program (launched February 2017) that extends professional-grade alerting to vetted community members for residential incidents [PulsePoint Foundation, 2024 (responder types)](#ref-7).

The largest peer-reviewed evaluation analyzed PulsePoint dispatches in Allegheny County (Pittsburgh, PA) from July 2016 through October 2020: of 840 total dispatches, 64 (7.6%) were confirmed OHCA with resuscitation attempted; PulsePoint-associated OHCAs received bystander CPR significantly more often than matched non-PulsePoint OHCAs in public locations (p = 0.008) [Sayre et al., 2021](#ref-8). A more recent county-level implementation study in an urban California county reported that bystander CPR rates rose from a pre-deployment baseline to 56.8% after PulsePoint launched, above the US national average [Stark et al., 2025](#ref-9). The PulsePoint randomized controlled trial is currently enrolling [PulsePoint Study, NCT04806958](#ref-10).

## GoodSAM (United Kingdom and global)

GoodSAM ("Good Smartphone Activated Medic") is the volunteer-responder platform integrated with the London Ambulance Service and East Midlands Ambulance Service in the UK and with Ambulance Victoria in Australia, among others. It is operationally distinct from PulsePoint in that GoodSAM **requires medical training credentials** (physician, nurse, paramedic, military medic, or equivalent) for registration as a responder, while PulsePoint's Public class accepts any CPR-trained citizen [Smith et al., 2022](#ref-2).

The pivotal UK evaluation matched GoodSAM alerts to the national OHCA Outcomes Registry across London (April 2016 to March 2017) and East Midlands (January 2018 to June 2018). Of OHCAs where a GoodSAM responder arrived before EMS, **survival to hospital discharge was approximately doubled** versus matched controls (adjusted odds ratio 2.81, 95% CI 1.04 to 7.60) [Smith et al., 2022](#ref-2). An independent Ambulance Victoria observational cohort study of >9,000 OHCAs over 5.5 years similarly found that survival to hospital discharge was 37% more likely when GoodSAM responders arrived before paramedics [Stub et al., 2025](#ref-11). The First Responder Shock Trial (FIRST), a cluster-randomized trial of GoodSAM responders equipped with ultraportable AEDs, is enrolling in Victoria and New Zealand [Smith et al., 2023 (FIRST protocol)](#ref-12).

## Stockholm SMS-Lifesaver, SAMBA, and HeartRunner (Nordic systems)

The Nordic systems are the most rigorously studied. The original Stockholm randomized controlled trial (the "SMS Lifesavers" study by Ringh et al.) enrolled 5,989 lay volunteers and randomized 667 OHCAs from April 2012 through December 2013: the bystander CPR rate was **62% in the intervention arm versus 48% in the control arm** (absolute difference 14 percentage points, 95% CI 6 to 21, p < 0.001) [Ringh et al., 2015](#ref-1). This is the primary randomized evidence that smartphone fan-out alerting changes the upstream behavior (bystander CPR before EMS) that drives OHCA survival.

The follow-on SAMBA trial extended the question to **AED retrieval**: dispatchers sent some responders directly to the patient and others to fetch a nearby AED. SAMBA enrolled OHCAs in three Swedish regions from December 2018 through January 2020 and found that smartphone-dispatched volunteers significantly increased bystander defibrillation in private residences but did not change overall AED-attachment rates [Berglund et al., 2018 (precursor)](#ref-14), [Andelius et al., 2023 (JACC)](#ref-15).

The Danish HeartRunner programme, funded by TrygFonden, launched in the Capital Region of Denmark in September 2017 and expanded nationally in 2020. In the JACC evaluation, the system was activated in 819 cases of suspected OHCA (438 confirmed); citizen responders arrived before EMS in 41% of cases and significantly increased both bystander CPR and bystander defibrillation [Andelius et al., 2020](#ref-3).

The Amsterdam alert-system study from the HartslagNu network (785 AEDs and 5,735 volunteer responders in residential areas) reported that **survival from at-home OHCAs increased from 26% to 39%** after alert-system deployment (adjusted RR 1.5, 95% CI 1.03 to 2.0) [Stieglis et al., 2022](#ref-4).

## Berlin KATRETTER and the false-positive problem

Berlin's KATRETTER programme is one of the largest first-responder systems in Europe and notably accepts volunteers **without medical qualifications**. An analysis of approximately 16,500 KATRETTER activations between October 2020 and October 2022 found a **high rate of false-positive activations** — dispatches triggered by non-OHCA conditions — leading the operators to remove certain dispatch codes from automatic responder activation in February 2022 [Pommerenke et al., 2024](#ref-16). This is the most explicit published evidence of the false-positive failure mode in the citizen-responder category.

A US "Verified Responder" pilot reported lower activation reach: of 475 private-location incidents, volunteers were alerted in only 8%, responded in 2.7%, and arrived on scene in 2.3%; in public locations the numbers were 6.3%, 2.3%, and 2.3% respectively. The same pilot reported >96% positive responder impressions and no adverse responder events [Stark et al., 2019 (abstract)](#ref-17).

## Echo112 and what3words: location-precision services

Echo112 (now EchoSOS) is a Swiss-developed app that transmits the caller's precise GPS coordinates to an emergency-services portal accessible by call-takers; if mobile data is unavailable, the app falls back to SMS-based coordinate transmission [Echo112, 2024](#ref-18). It is a **location-precision layer**, not a dispatch system: it does not push alerts to responders.

what3words assigns every 3 metre by 3 metre square on Earth a unique three-word address. The relevant integrations for emergency dispatch are with CAD vendors: Mark43 CAD and Hexagon HxGN OnCall Dispatch both ship native what3words integrations, and what3words reports availability through RapidSOS to over 4,800 US Emergency Communications Centers [what3words Public Safety, 2024](#ref-19), [Mark43, 2021](#ref-20), [Hexagon, 2021](#ref-21). The integration model is bidirectional: a caller without a street address gives the call-taker three words from the what3words app, and the CAD geocodes them into a precise pin. The Los Angeles Fire Department was an early municipal adopter (2021) [what3words Public Safety, 2024](#ref-19).

For the Rescue-Squad design, what3words is most useful as a fallback when GPS accuracy is poor (urban canyons, indoor incidents) but is not a substitute for live-GPS tracking once the responder is en route.

## Agency-internal first-responder apps

This category is distinct from citizen-responder apps: the user base is a **closed roster of agency personnel** rather than a crowdsourced population.

- **Active911** provides alerting, mapping, scheduling (ActiveTeam), and mutual-aid comms (ActiveComms) to over 340,000 first responders globally [Active911, 2024](#ref-22). ActiveTeam handles shift-based rostering — directly relevant to the Rescue-Squad "on call right now" model.
- **Tablet Command** is an iPad-native incident command and tactical management tool, designed to function offline and replace mobile data terminals. It is used at the engine-company and command level rather than for fan-out alerting [Tablet Command, 2024](#ref-23).
- **IamResponding** (now part of RapidSOS) provides end-to-end alerts, who-is-responding rosters, and incident data for volunteer fire and EMS agencies; it integrates with what3words for location resolution [IamResponding, 2024](#ref-24).
- **Pulsara** is a hospital-EMS clinical communication platform focused on time-sensitive emergencies (STEMI, stroke, sepsis, trauma). The Australian feasibility study across 25 Ambulance Victoria branches and 2 hospitals enrolled 604 stroke and 247 STEMI patients and found significant improvements in patient transfer and triage times after Pulsara deployment (e.g., 23 minutes faster ambulance-loaded-to-hospital-arrival for STEMI; proportion of stroke patients treated within 60 minutes rose from 12% to 26%) [Bladin et al., 2022](#ref-25).

## Dispatch algorithm literature

There are three recurring algorithmic paradigms in the EMS dispatch literature.

**Closest-unit dispatch** is the historical baseline: the call goes to the geographically nearest available unit. Multiple simulation studies have shown this is suboptimal under uncertainty because it ignores the impact of removing a unit from its coverage area on future calls [Jagtenberg et al., 2017](#ref-13). The Rio de Janeiro study compared closest-available dispatch to a stochastic-programming policy on >2 years of real EMS data and found smaller mean response times for the alternative [Chao et al., 2024](#ref-26).

**Response-time prediction dispatch** chooses the unit with the lowest expected travel time given weather, time of day, traffic, and route topology. This consistently outperforms closest-by-Euclidean-distance in simulation but requires either real-time traffic feeds or learned travel-time models [Jagtenberg et al., 2017](#ref-13).

**Fan-out alerting** (which is what citizen-responder apps and the Rescue-Squad concept do) sends the alert to many candidates simultaneously and lets the first to accept handle it. The HeartRunner, GoodSAM, and SAMBA systems all use this model. The dispatch optimization in these systems is less about choosing one unit and more about (a) defining the **alert radius** (typically 1.5 to 2 km in studied systems), (b) **capping the responder count** to avoid swarm effects, and (c) routing some responders to AED-pickup points before the scene [Andelius et al., 2020](#ref-3), [Stieglis et al., 2022](#ref-4).

Priority-based dispatch (Medical Priority Dispatch System, MPDS, and Criteria-Based Dispatch, CBD) is a separate axis: it determines **what level of response** (advanced life support vs basic life support vs first-responder only) and is established in standard PSAP operations [Andersen et al., 2018](#ref-27). The accuracy literature reports that priority systems effectively identify non-urgent calls but suffer from high over-triage that strains EMS resources [Andersen et al., 2018](#ref-27).

The Utstein-style **reporting standard for first-responder systems** (Hinterzarten consensus, 36 experts from 13 countries) was published in 2024 and now defines the minimum metadata to publish for any new smartphone alerting system: alert radius, responder type, dispatch trigger codes, AED-pairing logic, training requirements, and outcome measures [Metelmann et al., 2024](#ref-28). A new Rescue-Squad-style system targeting peer-reviewed publication should comply with this standard from day one.

## On-shift and rostering models

Three rostering archetypes appear in the deployed systems:

- **Always-eligible** (PulsePoint Public, KATRETTER): any registered user receives alerts whenever they are physically within the alert radius and the app is foregrounded or has background-location enabled. There is no concept of "on call" or "off duty" [PulsePoint Foundation, 2024 (responder types)](#ref-7), [Pommerenke et al., 2024](#ref-16).
- **Off-duty professional** (PulsePoint Professional Responder, Verified Responder): the user is on duty within their agency's CAD and is *also* eligible to receive PulsePoint alerts when off-duty and physically near an incident outside their on-duty area [PulsePoint Foundation, 2024 (responder types)](#ref-7).
- **Roster-based** (Active911 ActiveTeam, IamResponding, agency-internal Pulsara users): the agency defines shifts and the system only alerts users who are scheduled on duty at the time of the call [Active911, 2024](#ref-22), [IamResponding, 2024](#ref-24).

The Rescue-Squad concept (closed roster, on-call shifts, fan-out per incident) is closest to the roster-based archetype, but with the citizen-responder-style fan-out behavior on top.

## Known failure modes

Five failure modes are documented in the literature:

1. **False positives**: the KATRETTER analysis is the cleanest published evidence — the Berlin operators had to remove specific dispatch codes from automatic activation in February 2022 because of high false-positive rates [Pommerenke et al., 2024](#ref-16). Any non-cardiac-arrest dispatch logic, including the Rescue-Squad's broader "medical emergency" trigger, will have a higher false-positive base rate than cardiac-arrest-only systems and needs explicit mitigation.

2. **Responder wellbeing and post-traumatic stress**: A Swedish prospective study of smartphone-dispatched lay responders found that most rated the experience as positive and high-energy, with low post-traumatic stress scores at 4 to 6 weeks and no adverse events at the population level [Nord et al., 2022](#ref-29). A separate binational survey reported some lay responders showed elevated post-traumatic stress symptoms, particularly when arriving at a deceased patient [Sarkisian et al., 2022](#ref-30). The combined evidence is reassuring but not zero-risk.

3. **Geofence false positives and walkability**: PulsePoint dispatches in some areas have been shown to send responders to addresses they cannot reach on foot within the survival window (highway-adjacent incidents, restricted access), motivating walkability-aware alert filtering [Stark et al., 2020](#ref-31).

4. **Off-duty professionals showing up underprepared**: While the literature does not document this as harm, the PulsePoint Professional/Verified Responder model formalizes which off-duty staff are *invited* into residential-alert eligibility specifically to manage this risk [PulsePoint Foundation, 2024 (responder types)](#ref-7).

5. **Low reach and activation in private locations**: The Verified Responder pilot reported only 2.3% of private-location incidents resulted in a volunteer arriving on scene [Stark et al., 2019 (abstract)](#ref-17). Private-residence OHCAs are the majority of all OHCAs and the hardest to reach, which is a primary motivator for the closed-roster on-call model the Rescue-Squad design contemplates.

## Implications for the Rescue-Squad design

Three concrete implications follow from the evidence:

1. **Fan-out alerting with a defined alert radius is the right primitive.** The randomized (Stockholm) and large observational (Amsterdam, Denmark, UK, Victoria) evidence shows clear OHCA survival benefit from smartphone fan-out to nearby trained responders. The roster-based "on call right now" gate sits cleanly on top of this primitive.

2. **Plan for false positives from day one.** Berlin's KATRETTER experience is the strongest warning: dispatch-code-level filtering, suppression of low-confidence triggers, and a feedback loop from responders ("this was not an emergency") need to be first-class features, not afterthoughts.

3. **Adopt the 2024 Utstein-style reporting standard.** If the Rescue-Squad team wants this app to generate peer-reviewable outcomes data, [Metelmann et al., 2024](#ref-28) defines the minimum operational and outcome metadata (alert radius, responder type, dispatch trigger codes, training requirements, on-scene-before-EMS rate, bystander CPR rate, AED-attachment rate, survival metrics) that should be logged from the first deployment.

## Sources

<a id="ref-1"></a>
1. Ringh, M., Rosenqvist, M., Hollenberg, J., Jonsson, M., Fredman, D., Nordberg, P., Jarnbert-Pettersson, H., Hasselqvist-Ax, I., Riva, G., & Svensson, L. (2015). "Mobile-Phone Dispatch of Laypersons for CPR in Out-of-Hospital Cardiac Arrest." *New England Journal of Medicine*, 372(24), 2316-2325. [https://doi.org/10.1056/NEJMoa1406038](https://doi.org/10.1056/NEJMoa1406038)

<a id="ref-2"></a>
2. Smith, C. M., Lall, R., Fothergill, R. T., Spaight, R., & Perkins, G. D. (2022). "The effect of the GoodSAM volunteer first-responder app on survival to hospital discharge following out-of-hospital cardiac arrest." *European Heart Journal: Acute Cardiovascular Care*, 11(1), 20-31. [https://doi.org/10.1093/ehjacc/zuab103](https://doi.org/10.1093/ehjacc/zuab103)

<a id="ref-3"></a>
3. Andelius, L., Malta Hansen, C., Lippert, F. K., Karlsson, L., Torp-Pedersen, C., Kjaer Ersboll, A., Kober, L., Collatz Christensen, H., Blomberg, S. N., Gislason, G. H., Folke, F., & Wissenberg, M. (2020). "Smartphone Activation of Citizen Responders to Facilitate Defibrillation in Out-of-Hospital Cardiac Arrest." *Journal of the American College of Cardiology*, 76(1), 43-53. [https://doi.org/10.1016/j.jacc.2020.04.073](https://doi.org/10.1016/j.jacc.2020.04.073)

<a id="ref-4"></a>
4. Stieglis, R., Zijlstra, J. A., Riedijk, F., Smeekes, M., van der Worp, W. E., Tijssen, J. G. P., Hulleman, M., & Koster, R. W. (2022). "Alert system-supported lay defibrillation and basic life-support for cardiac arrest at home." *European Heart Journal*, 43(15), 1465-1474. [https://doi.org/10.1093/eurheartj/ehab802](https://doi.org/10.1093/eurheartj/ehab802)

<a id="ref-5"></a>
5. PulsePoint Foundation. (2024). "PulsePoint Respond: Inform and engage your community." Official product page. [https://www.pulsepoint.org/pulsepoint-respond](https://www.pulsepoint.org/pulsepoint-respond)

<a id="ref-6"></a>
6. PulsePoint Foundation. (2024). "Implementing PulsePoint in your community." Official implementation guide. [https://www.pulsepoint.org/implementation](https://www.pulsepoint.org/implementation)

<a id="ref-7"></a>
7. PulsePoint Foundation. (2024). "PulsePoint Responder Types and Features." Official documentation. [https://www.pulsepoint.org/responder-types-and-features](https://www.pulsepoint.org/responder-types-and-features)

<a id="ref-8"></a>
8. Sayre, M. R., Barnard, L. M., Counts, C. R., Drucker, C. J., Kudenchuk, P. J., Rea, T. D., & Eisenberg, M. S. (2021). "PulsePoint dispatch associated patient characteristics and prehospital outcomes in a mid-sized metropolitan area." *Resuscitation*, 170, 36-43. [https://doi.org/10.1016/j.resuscitation.2021.11.005](https://doi.org/10.1016/j.resuscitation.2021.11.005) (PubMed: [34774964](https://pubmed.ncbi.nlm.nih.gov/34774964/))

<a id="ref-9"></a>
9. Stark, C. W., Brent, C. M., Stein, B. D., Bobrow, B. J., & Spaite, D. W. (2025). "Impact of a 9-1-1-Integrated Mobile App on Bystander CPR: Implementation of PulsePoint in an Urban County." *Journal of Clinical Medicine*, 15(1), 5. [https://doi.org/10.3390/jcm15010005](https://doi.org/10.3390/jcm15010005)

<a id="ref-10"></a>
10. Brooks, S. C. et al. "The PulsePoint Study." ClinicalTrials.gov identifier NCT04806958. [https://clinicaltrials.gov/study/NCT04806958](https://clinicaltrials.gov/study/NCT04806958)

<a id="ref-11"></a>
11. Stub, D., Smith, K., Bray, J. E., et al. (2025). "Smartphone-activated volunteer responders and survival to discharge after out-of-hospital cardiac arrests in Victoria, 2018-23: an observational cohort study." *Medical Journal of Australia*. [PMC article PMC12126968](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC12126968/)

<a id="ref-12"></a>
12. Smith, K., Andrew, E., Bray, J. E., et al. (2023). "A study protocol for a cluster-randomised controlled trial of smartphone-activated first responders with ultraportable defibrillators in out-of-hospital cardiac arrest: The First Responder Shock Trial (FIRST)." *Resuscitation Plus*, 16, 100481. [https://doi.org/10.1016/j.resplu.2023.100481](https://doi.org/10.1016/j.resplu.2023.100481)

<a id="ref-13"></a>
13. Jagtenberg, C. J., Bhulai, S., & van der Mei, R. D. (2017). "Dynamic ambulance dispatching: is the closest-idle policy always optimal?" *Health Care Management Science*, 20(4), 517-531. [https://doi.org/10.1007/s10729-016-9368-0](https://doi.org/10.1007/s10729-016-9368-0)

<a id="ref-14"></a>
14. Berglund, E., Claesson, A., Nordberg, P., Djarv, T., Lundgren, P., Folke, F., Forsberg, S., Riva, G., & Ringh, M. (2018). "A smartphone application for dispatch of lay responders to out-of-hospital cardiac arrests." *Resuscitation*, 126, 160-165. [https://doi.org/10.1016/j.resuscitation.2018.01.039](https://doi.org/10.1016/j.resuscitation.2018.01.039)

<a id="ref-15"></a>
15. Jonsson, M., Berglund, E., Djarv, T., Nordberg, P., Claesson, A., Forsberg, S., Nord, A., Tan, H. L., & Ringh, M. (2023). "Effect of Smartphone Dispatch of Volunteer Responders on Automated External Defibrillators and Out-of-Hospital Cardiac Arrests: The SAMBA Randomized Clinical Trial." *JAMA Cardiology*, 8(1), 81-88. [https://doi.org/10.1001/jamacardio.2022.4362](https://doi.org/10.1001/jamacardio.2022.4362) (PubMed: [36449309](https://pubmed.ncbi.nlm.nih.gov/36449309/))

<a id="ref-16"></a>
16. Pommerenke, C., Wnent, J., Trentzsch, H., Prckno, L., Strapatsas, J., Gradl, P., Seewald, S., Grasner, J. T., & Fischer, M. (2024). "Automated and app-based activation of first responders for prehospital cardiac arrest: an analysis of 16,500 activations of the KATRETTER system in Berlin." *Notfall + Rettungsmedizin*. [https://doi.org/10.1007/s10049-023-01270-6](https://doi.org/10.1007/s10049-023-01270-6) (PubMed: [38124125](https://pubmed.ncbi.nlm.nih.gov/38124125/))

<a id="ref-17"></a>
17. Stark, C. W., Sumner, J., Page, R. L., Robinson, K., Cone, D. C., & Lerner, E. B. (2019). "Abstract 295: Improving Response to Out-Of-Hospital Cardiac Arrest: Verified Responder Pilot Program." *Circulation*, 140(Suppl 2), A295. [https://doi.org/10.1161/circ.140.suppl_2.295](https://www.ahajournals.org/doi/10.1161/circ.140.suppl_2.295)

<a id="ref-18"></a>
18. MobileMed Sarl. (2024). "EchoSOS (formerly Echo112) official product information." [https://apps.apple.com/us/app/echo112-the-pocket-lifesaver/id613058743](https://apps.apple.com/us/app/echo112-the-pocket-lifesaver/id613058743)

<a id="ref-19"></a>
19. what3words. (2024). "Emergency and Recovery: official documentation." [https://what3words.com/business/emergency](https://what3words.com/business/emergency)

<a id="ref-20"></a>
20. Mark43. (2021). "Mark43 Announces Partnership with what3words, Bringing Additional Precision and Accuracy to its Computer-Aided Dispatch." Official press release. [https://mark43.com/press/mark43-announces-partnership-with-location-technology-company-what3words-bringing-additional-precision-and-accuracy-to-its-computer-aided-dispatch/](https://mark43.com/press/mark43-announces-partnership-with-location-technology-company-what3words-bringing-additional-precision-and-accuracy-to-its-computer-aided-dispatch/)

<a id="ref-21"></a>
21. Hexagon AB. (2021). "Hexagon enhances geocoding capabilities, connects to what3words to boost location capabilities for emergency calls." Official press release. [https://hexagon.com/company/newsroom/press-releases/2021/hexagon-enhances-geocoding-capabilities-connects-what3words](https://hexagon.com/company/newsroom/press-releases/2021/hexagon-enhances-geocoding-capabilities-connects-what3words)

<a id="ref-22"></a>
22. Active911, Inc. (2024). "Active911 Products: ActiveAlert, ActiveTeam, ActiveComms." Official product documentation. [https://active911.com/products/](https://active911.com/products/)

<a id="ref-23"></a>
23. Tablet Command, Inc. (2024). "Tablet Command Incident Management Software: official product documentation." [https://www.tabletcommand.com/](https://www.tabletcommand.com/)

<a id="ref-24"></a>
24. IamResponding (RapidSOS). (2024). "IamResponding: First Responder System official documentation." [https://iamresponding.com/](https://iamresponding.com/)

<a id="ref-25"></a>
25. Bladin, C. F., Bagot, K. L., Vu, M., Kim, J., Bernard, S., Smith, K., Hocking, G., Coupland, T., Pearce, D. C., Jolliffe, L., Cadilhac, D. A., & Australian Stroke and STEMI Pulsara Investigators. (2022). "Real-world, feasibility study to investigate the use of a multidisciplinary app (Pulsara) to improve prehospital communication and timelines for acute stroke/STEMI care." *BMJ Open*, 12(7), e052332. [https://doi.org/10.1136/bmjopen-2021-052332](https://doi.org/10.1136/bmjopen-2021-052332)

<a id="ref-26"></a>
26. Chao, X., Hu, P., & Zheng, S. (2024). "Dispatch Optimization for Ambulance and Emergency Service Systems." SSRN Working Paper. [https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5391669](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=5391669)

<a id="ref-27"></a>
27. Andersen, M. S., Johnsen, S. P., Sorensen, J. N., Jepsen, S. B., Hansen, J. B., & Christensen, E. F. (2018). "The accuracy of medical dispatch: a systematic review." *Scandinavian Journal of Trauma, Resuscitation and Emergency Medicine*, 26, 60. [https://doi.org/10.1186/s13049-018-0528-8](https://doi.org/10.1186/s13049-018-0528-8) (PMC: [PMC6230269](https://pmc.ncbi.nlm.nih.gov/articles/PMC6230269/))

<a id="ref-28"></a>
28. Metelmann, C., Metelmann, B., Muller, M. P., Brinkrolf, P., Hahnenkamp, K., Beckers, S. K., Boettiger, B. W., Krohn, J. B., Lockey, A., Maconochie, I. K., Monsieurs, K. G., Nolan, J. P., Olasveengen, T. M., Perkins, G. D., Semeraro, F., Stieglis, R., Strapatsas, J., & Wnent, J. (2024). "Reporting standard for describing first responder systems, smartphone alerting systems, and AED networks." *Resuscitation*, 195, 110079. [https://doi.org/10.1016/j.resuscitation.2023.110079](https://doi.org/10.1016/j.resuscitation.2023.110079)

<a id="ref-29"></a>
29. Nord, A., Hult, H., Kreitz-Sandberg, S., & Herlitz, J. (2021). "Wellbeing, emotional response and stress among lay responders dispatched to suspected out-of-hospital cardiac arrests." *Resuscitation*, 170, 352-360. [https://doi.org/10.1016/j.resuscitation.2021.10.041](https://doi.org/10.1016/j.resuscitation.2021.10.041) (PubMed: [34774709](https://pubmed.ncbi.nlm.nih.gov/34774709/))

<a id="ref-30"></a>
30. Sarkisian, L., Mickley, H., Schakow, H., Gerke, O., Jorgensen, G., Larsen, M. L., & Henriksen, F. L. (2022). "A binational survey of smartphone activated volunteer responders for out-of-hospital cardiac arrest: Availability, interventions, and post-traumatic stress." *Resuscitation*, 170, 1-9. [https://doi.org/10.1016/j.resuscitation.2021.10.030](https://doi.org/10.1016/j.resuscitation.2021.10.030)

<a id="ref-31"></a>
31. Stark, C. W., & Cone, D. C. (2020). "Can you get there from here? An analysis of walkability among PulsePoint CPR alert dispatches." *Resuscitation*, 149, 35-39. [https://doi.org/10.1016/j.resuscitation.2020.01.018](https://doi.org/10.1016/j.resuscitation.2020.01.018)
