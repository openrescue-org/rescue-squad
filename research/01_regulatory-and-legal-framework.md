# Regulatory and Legal Framework for a Smartphone-Based EMS Dispatch App

## Overview

This document maps the regulatory and legal surface for a smartphone application that lets end-users summon help during a medical emergency and fans the request out to on-call, trained EMS providers, with bidirectional live GPS shared until the user marks the call resolved. The product sits at the intersection of three overlapping regimes: (a) **public-safety telecommunications law** (911/E911/NG911 in the United States; equivalent regimes for 112 in the EU, 999 in the UK, 102/108/112 in South Asia); (b) **health-information privacy law** (chiefly HIPAA in the US, plus state analogs); and (c) **EMS practice law** (state licensure rules that distinguish a community-alerting app from a licensed dispatching agency, plus Good Samaritan immunity for off-duty responders). Each regime is jurisdiction-specific. The dominant design constraint is that **no third-party app in the United States may lawfully substitute for the public 911/PSAP system** — it can only complement it. Almost every other regulatory question (HIPAA scope, dispatcher liability, NEMSIS reporting) is downstream of where the product positions itself on the "alerter vs. dispatcher" spectrum.

## 1. U.S. 911 / E911 Architecture and Third-Party App Integration

The Federal Communications Commission's 911 rules live in **47 CFR Part 9** [eCFR, 47 CFR Part 9](#ref-1). Subpart E (sections 9.10 and 9.14) imposes obligations on Commercial Mobile Radio Service (CMRS) providers and Interconnected VoIP providers to (i) transmit all 911 calls to the appropriate Public Safety Answering Point (PSAP) regardless of subscriber validation status, and (ii) deliver Automatic Location Information (ALI) and Automatic Number Identification (ANI) with each call [47 CFR § 9.14](#ref-2). A PSAP is the facility "designated to receive 911 calls and route them to emergency services personnel" — typically a county or municipal communications center — and only a PSAP, not a private intermediary, is the legal terminus for a 911 call in the United States [eCFR, 47 CFR Part 9](#ref-1).

The FCC's Wireless E911 Location Accuracy Rules (47 CFR § 9.10) require CMRS providers to deliver, where technically feasible, a **"dispatchable location"** — defined as "the street address of the calling party, plus additional information such as suite, apartment or similar information necessary to adequately identify the location" — or a z-axis estimate within plus-or-minus 3 meters of the handset for 80% of E911 calls [47 CFR § 9.10](#ref-3). By the April 3, 2025 benchmark, nationwide CMRS providers must provide either dispatchable location or z-axis nationwide [47 CFR § 9.10](#ref-3).

**Next Generation 911 (NG911)** is governed by 47 CFR Part 9 Subpart J (§§ 9.27–9.34). The FCC's rules set a two-phase transition initiated by a "911 Authority" submitting a valid request to originating service providers (OSPs) to deliver 911 traffic in IP/SIP format to designated NG911 Delivery Points routed via an Emergency Services IP Network (ESInet) [eCFR, 47 CFR Part 9 Subpart J](#ref-4). The technical architecture for NG911 is the **NENA i3 Standard** (NENA-STA-010), which standardizes the software services, databases, network elements, and SIP-based interfaces that process IP-delivered emergency calls and associated multimedia [NENA i3 Standard](#ref-5). The FCC's 2024 NG911 Report and Order codified the i3-aligned transition requirements [Federal Register, 2024](#ref-6).

**Implications for a third-party app:**
- A consumer app cannot lawfully act as the PSAP. It must either (a) place a standard 911 call through the device's native dialer (so the carrier delivers the call to the PSAP), or (b) route the request to a parallel "community alerting" channel that does not replace 911. Most production apps (PulsePoint, GoodSAM) take approach (b) — alerting volunteer responders **in addition to**, never in lieu of, a 911 call already initiated by the user or by the PSAP itself [IAFC PulsePoint White Paper](#ref-7).
- To consume the PSAP's CAD (Computer Aided Dispatch) feed and learn about active cardiac arrests, integration must be negotiated with each PSAP/911 Authority individually; NG911 i3 contemplates such third-party data subscribers but does not entitle them [NENA i3 Standard](#ref-5).
- The FCC's location-accuracy regime applies to **carriers**, not to apps. A consumer app that captures its own GPS for routing does not displace the carrier's 911 location duty.

## 2. HIPAA Applicability

The Health Insurance Portability and Accountability Act (HIPAA) Privacy and Security Rules are codified at **45 CFR Parts 160 and 164** [eCFR, 45 CFR Part 160](#ref-8) [eCFR, 45 CFR Part 164](#ref-9). Two categories of regulated entity exist:

1. **Covered Entity (CE):** health plans, health care clearinghouses, and health care providers who transmit any health information in electronic form in connection with a covered transaction [45 CFR § 160.103](#ref-10).
2. **Business Associate (BA):** a person or entity that "creates, receives, maintains, or transmits" protected health information (PHI) on behalf of a covered entity to perform a function regulated by HIPAA [45 CFR § 160.103](#ref-10).

**Protected Health Information (PHI)** is individually identifiable health information that a CE or BA creates, receives, maintains, or transmits in any form, relating to past, present, or future physical or mental health, the provision of health care, or payment for it [45 CFR § 164.501](#ref-11). Geolocation tied to a specific medical event (e.g., "this user requested EMS at these coordinates") is PHI when the requester is identifiable.

HHS's Office for Civil Rights has explicitly addressed mobile health apps. In its 2016 guidance "Health App Use Scenarios & HIPAA," OCR distinguishes apps that operate **on behalf of a covered entity** (BA — full HIPAA Security/Privacy Rule obligations, Business Associate Agreement required) from consumer-direct apps that "are not covered entities unless [they act] on behalf of a covered entity" [HHS OCR, 2016](#ref-12). The companion FTC/HHS Interactive Tool walks developers through whether the FTC Act, the FTC Health Breach Notification Rule, HIPAA, and/or the FD&C Act apply [FTC, 2016](#ref-13).

**Implications for the EMS app:**
- A pure peer-to-peer alerting model where the user transmits a request directly to nearby volunteers, with no covered-entity intermediary, may sit **outside HIPAA's scope** — but it remains squarely inside the FTC Health Breach Notification Rule and any state PHI/biometric statutes [FTC, 2016](#ref-13).
- The moment the app integrates with a licensed EMS agency, hospital, or PSAP that is a covered entity (e.g., the agency uses the app to dispatch its own units, or the app ingests CAD data), the developer becomes a Business Associate and a BAA is required [45 CFR § 164.504(e)](#ref-9) [HHS OCR, 2016](#ref-12).
- The data items most likely to constitute PHI on this platform: (i) the requester's identity + GPS coordinates + timestamp of medical event, (ii) any free-text symptoms entered into the request, (iii) responder-side notes or chief-complaint codes, (iv) call-resolution status.

## 3. NEMSIS Data Standard

The National EMS Information System (NEMSIS) is the NHTSA-funded national EMS data standard; the current production version is **v3.5.0** with v3.5.1 in transition [NEMSIS Data Dictionary v3.5.1](#ref-14). The standard is structured into three datasets: **EMS (the ePCR / patient care report), Demographic (Agency) — DEMDataSet, and StateDataSet (SDS)** [NEMSIS DEMDataSet Guide, 2022](#ref-15). NEMSIS v3.5 contains 22 National Mandatory and Required elements for the agency DEM file and a much larger set of required elements per EMS event for the ePCR [NEMSIS v3.5 Requisite National Elements](#ref-16).

The EMS dataset captures, at minimum: incident location, dispatch and en-route/on-scene/transport/at-destination timestamps, chief complaint, primary impression, patient demographics, vital signs, procedures, medications, and disposition. NEMSIS is the system EMS agencies use to **report to the state**, which then forwards de-identified data to the national repository [NEMSIS Public Release RDS v3.5.0](#ref-17).

**Implications:** a pure community-alerting app does not have to emit NEMSIS records; only a **licensed EMS agency** generating an ePCR for a billable encounter must. However, if the app aspires to integrate with state EMS systems, generate runtime metrics interoperable with state databases, or become an ePCR vendor for licensed agencies, it must conform to NEMSIS v3.5 schema and pass NEMSIS compliance testing [NEMSIS Data Dictionary v3.5.1](#ref-14).

## 4. Good Samaritan Laws and Volunteer Liability

Good Samaritan immunity is governed at the state level. In Virginia, the controlling statute is **Va. Code § 8.01-225**, which provides civil immunity for any person who "in good faith, renders emergency care or assistance, without compensation, to any ill or injured person … at the scene of an accident, fire, or any life-threatening emergency" — protecting them from civil damages for acts or omissions resulting from such care, with the exception of conduct constituting gross negligence or willful misconduct [Va. Code § 8.01-225](#ref-18). A companion statute, **Va. Code § 8.01-225.3**, extends immunity to volunteer firefighters and volunteer EMS personnel for injuries arising from operation of emergency vehicles en route to a call [Va. Code § 8.01-225.3](#ref-19). "Compensation" is defined to exclude the regular salary of public officials, so paid EMTs do not lose immunity solely by virtue of being paid for their day job; however, on-duty paid responders are usually governed by separate sovereign-immunity and ministerial-duty doctrines.

PulsePoint and GoodSAM operate explicitly within state Good Samaritan frameworks: GoodSAM in the UK requires volunteer responders to hold in-date CPR training before they can register [Smith et al., 2021](#ref-20), and the IAFC's PulsePoint white paper documents that the app activates **off-duty, CPR-trained personnel** in jurisdictions where Good Samaritan protections apply [IAFC PulsePoint White Paper](#ref-7). The first randomized controlled trial of mobile-phone dispatch of CPR-trained laypersons (Ringh et al., NEJM 2015) found a statistically significant increase in bystander-CPR rates (62% vs 48%, P<0.001) but no statistically significant survival benefit in that single trial [Ringh et al., 2015](#ref-21); a subsequent GoodSAM cohort study (Smith et al., 2021) reported comparable findings and characterized survival evidence as "limited" [Smith et al., 2021](#ref-20).

**Implication:** structuring the responder pool as **volunteer, trained, uncompensated, opt-in** maximizes the chance that state Good Samaritan immunity attaches to responder actions, but cannot insulate the **platform operator** from claims of negligent design, negligent recruitment of volunteers, or failure-to-warn.

## 5. State EMS Licensure: Alerting vs. Dispatching

State EMS regulators draw a sharp line between **community alerting** (notifying lay or off-duty responders that an event has occurred) and **EMS dispatching** (directing a licensed EMS agency's units to a scene under medical direction). In Virginia, the EMS Regulations are codified at **12VAC5-31**. Section 12VAC5-31-300 requires that "no person may establish, operate, maintain, advertise or represent themselves or any service or organization as an EMS agency or as EMS personnel without a valid license or certification issued by the Office of EMS" [12VAC5-31-300](#ref-22). Agency licensure (12VAC5-31-410 through -430) is valid for up to two years and is granted by classification [12VAC5-31-410](#ref-23) [12VAC5-31-430](#ref-24).

Emergency Medical Dispatch (EMD) is a separate regulated function in Virginia. The Office of EMS maintains an accreditation program for PSAPs/emergency dispatch centers; 100% of communications personnel whose job description includes call-taking must become certified Emergency Medical Dispatchers within two years of employment, through an OEMS-recognized program meeting NHTSA standards [VDH, Virginia Office of EMS Accreditation for PSAPs](#ref-25).

**Implications:**
- A PulsePoint-style app that merely **alerts** trained, off-duty responders to a nearby cardiac arrest does not itself become an "EMS agency" or an "EMD." It is a notification channel parallel to PSAP dispatch.
- An app that begins **directing licensed EMS units** to scenes — assigning calls, giving pre-arrival instructions to patients, providing chief-complaint coding — likely triggers EMD certification and/or EMS agency licensure in Virginia and most other states.
- Marketing language matters: 12VAC5-31-300 forbids holding the service out as "EMS personnel" or an "EMS agency" without a license. App copy must avoid implying licensed status.

## 6. International Ambulance Numbering and Regulators

Emergency-number assignment is governed nationally, with international harmonization recommendations from the **ITU-T E.161.1** ("Guidelines to select Emergency Number for public telecommunications networks") [ITU-T E.161.1](#ref-26) and inventory at ITU-T E.129 [ITU-T E.129](#ref-27). ITU recommends member states adopt 911, 112, or both [ITU-T E.161.1](#ref-26). Salient regional baselines:

- **United States and Canada:** 911 (universal), governed by FCC Part 9 (US) and CRTC equivalents (Canada).
- **European Union:** 112 is the mandated pan-European emergency number under **Directive 2002/22/EC** (Universal Service Directive) and continued under the European Electronic Communications Code (Directive 2018/1972) [EUR-Lex, Directive 2002/22/EC](#ref-28) [EUR-Lex, EECC summary](#ref-29). Member states must ensure free 112 access and provide caller-location information to emergency services.
- **United Kingdom:** 999 (original) and 112 (EU-aligned), both routed to the same call-handling centers; regulated by Ofcom.
- **India:** historically 100 (police), 101 (fire), and **102/108 (ambulance)**; the Telecom Regulatory Authority of India (TRAI) recommended 112 as a unified emergency number in May 2015, implemented by the Ministry of Home Affairs as the **Emergency Response Support System (ERSS-112)** [MHA, ERSS](#ref-30) [112.gov.in, About](#ref-31). The Supreme Court of India in 2026 directed all States and UTs to integrate 100, 101, 102, 108, and other helplines with 112 within three months [Republic World, 2026](#ref-32).
- **Nepal and Bangladesh:** 102 continues as the dedicated ambulance number; Nepal Telecommunications Authority and Bangladesh Telecommunication Regulatory Commission set numbering policy nationally.

**Implication:** any cross-border launch must treat the emergency-number-to-PSAP mapping as a per-country configuration table and must not assume a single dial behavior. The product also has to comply with each country's location-disclosure regime (EU "push" location under Commission Recommendation 2003/558/EC, US FCC 47 CFR § 9.10, India MHA ERSS specifications) [EUR-Lex, 2003 Recommendation](#ref-33).

## 7. Dispatcher Liability and Insurance Exposure

EMS dispatcher liability is a long-recognized hazard. The National Library of Medicine's *StatPearls* clinical reference summarizes that "harm resulting from a dispatching error exposes both the dispatcher and employer to liability for negligence" — typical errors include underestimating call urgency, mis-coding chief complaints, and mishandling caller information [Cooney, StatPearls, 2024](#ref-34). The standard of care is what an "ordinarily prudent, competent dispatcher" would do under similar circumstances, but conformity with peer practice does not insulate a dispatcher if the entire system standard is itself inadequate [Cooney, StatPearls, 2024](#ref-34). The International Academies of Emergency Dispatch (IAED) argues that priority medical dispatch protocols are now the standard of care in EMS [Clawson, 2004](#ref-35).

Medical direction is also a vector of liability. Under most state EMS frameworks, an EMS Medical Director is accountable for protocols and the clinical conduct of EMS personnel acting under those protocols; courts have generally distinguished negligent supervision (a high bar) from protocol-design negligence [Cooney, StatPearls, 2024](#ref-34).

**Implications for the app:**
- Even a "pure alerter" product faces theories of liability premised on (i) negligent design of the alerting algorithm, (ii) negligent vetting of registered responders, (iii) failure to warn users that the app does not replace 911, and (iv) data-breach exposure under HIPAA, state PHI laws, and the FTC Act.
- The platform should carry **technology errors-and-omissions** and **media/cyber** coverage in addition to general liability; if the platform employs medical direction (e.g., publishes pre-arrival CPR instructions), professional medical liability is also implicated.
- A best-in-class design replicates the PulsePoint architecture: (a) the user's 911 call is dialed by the device's native dialer, (b) the app is a parallel community-alerting channel, (c) the PSAP retains primary dispatch authority, and (d) pre-arrival instructions are sourced from established, evidence-based protocols (e.g., AHA Guidelines) rather than original platform content.

## Sources

<a id="ref-1"></a>
1. Electronic Code of Federal Regulations. *47 CFR Part 9 — 911 Requirements.* U.S. Government, current edition. [https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9)

<a id="ref-2"></a>
2. Electronic Code of Federal Regulations. *47 CFR § 9.14 — Emergency calling requirements.* [https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-E/section-9.14](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-E/section-9.14)

<a id="ref-3"></a>
3. Electronic Code of Federal Regulations. *47 CFR § 9.10 — 911 Service.* [https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-C/section-9.10](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-C/section-9.10)

<a id="ref-4"></a>
4. Electronic Code of Federal Regulations. *47 CFR Part 9 Subpart J — Next Generation 911.* [https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-J](https://www.ecfr.gov/current/title-47/chapter-I/subchapter-A/part-9/subpart-J)

<a id="ref-5"></a>
5. National Emergency Number Association. *NENA i3 Standard for Next Generation 9-1-1 (NENA-STA-010).* NENA, current versions 1–3. [https://www.nena.org/page/i3_Stage3](https://www.nena.org/page/i3_Stage3)

<a id="ref-6"></a>
6. Federal Communications Commission. *Facilitating Implementation of Next Generation 911 Services (NG911); Location-Based Routing for Wireless 911 Calls — Report and Order.* 89 Fed. Reg. (Sept. 24, 2024). [https://www.federalregister.gov/documents/2024/09/24/2024-18603/facilitating-implementation-of-next-generation-911-services-ng911-location-based-routing-for](https://www.federalregister.gov/documents/2024/09/24/2024-18603/facilitating-implementation-of-next-generation-911-services-ng911-location-based-routing-for)

<a id="ref-7"></a>
7. International Association of Fire Chiefs. *Using Mobile Technology to Activate Citizens and First Responders (PulsePoint White Paper).* IAFC, n.d. [https://www.iafc.org/docs/default-source/1assoc/white-paper-pulsepoint_final.pdf](https://www.iafc.org/docs/default-source/1assoc/white-paper-pulsepoint_final.pdf)

<a id="ref-8"></a>
8. Electronic Code of Federal Regulations. *45 CFR Part 160 — General Administrative Requirements.* [https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-160](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-160)

<a id="ref-9"></a>
9. Electronic Code of Federal Regulations. *45 CFR Part 164 — Security and Privacy.* [https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164)

<a id="ref-10"></a>
10. Electronic Code of Federal Regulations. *45 CFR § 160.103 — Definitions.* [https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-160/subpart-A/section-160.103](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-160/subpart-A/section-160.103)

<a id="ref-11"></a>
11. Electronic Code of Federal Regulations. *45 CFR § 164.501 — Definitions.* [https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-E/section-164.501](https://www.ecfr.gov/current/title-45/subtitle-A/subchapter-C/part-164/subpart-E/section-164.501)

<a id="ref-12"></a>
12. U.S. Department of Health and Human Services, Office for Civil Rights. *Health App Use Scenarios & HIPAA.* HHS, February 2016. [https://www.hhs.gov/sites/default/files/ocr-health-app-developer-scenarios-2-2016.pdf](https://www.hhs.gov/sites/default/files/ocr-health-app-developer-scenarios-2-2016.pdf)

<a id="ref-13"></a>
13. Federal Trade Commission. *Mobile Health App Interactive Tool* and *FTC Releases New Guidance For Developers of Mobile Health Apps* (April 2016). [https://www.ftc.gov/business-guidance/resources/mobile-health-apps-interactive-tool](https://www.ftc.gov/business-guidance/resources/mobile-health-apps-interactive-tool); [https://www.ftc.gov/news-events/news/press-releases/2016/04/ftc-releases-new-guidance-developers-mobile-health-apps](https://www.ftc.gov/news-events/news/press-releases/2016/04/ftc-releases-new-guidance-developers-mobile-health-apps)

<a id="ref-14"></a>
14. National Emergency Medical Services Information System (NEMSIS) Technical Assistance Center. *NEMSIS Data Dictionary, NHTSA v3.5.1.* NEMSIS / NHTSA. [https://nemsis.org/media/nemsis_v3/release-3.5.1/DataDictionary/PDFHTML/EMSDEMSTATE/NEMSISDataDictionary.pdf](https://nemsis.org/media/nemsis_v3/release-3.5.1/DataDictionary/PDFHTML/EMSDEMSTATE/NEMSISDataDictionary.pdf)

<a id="ref-15"></a>
15. NEMSIS Technical Assistance Center. *NEMSIS v3 DEMDataSet Guide.* November 8, 2022. [https://nemsis.org/wp-content/uploads/2022/11/NEMSIS_V3_DEMDataset_Guide.pdf](https://nemsis.org/wp-content/uploads/2022/11/NEMSIS_V3_DEMDataset_Guide.pdf)

<a id="ref-16"></a>
16. NEMSIS Technical Assistance Center. *NHTSA/NEMSIS Version 3.5 EMS (ePCR/Event) DataSet Requisite Elements.* [https://nemsis.org/wp-content/uploads/2026/03/NEMSIS-v3.5-Requisite-National-Elements-EMS-DataSet_.pdf](https://nemsis.org/wp-content/uploads/2026/03/NEMSIS-v3.5-Requisite-National-Elements-EMS-DataSet_.pdf)

<a id="ref-17"></a>
17. NEMSIS Technical Assistance Center. *National EMS Database — NEMSIS Public Release Research Data Set v3.5.0 User Manual.* September 2025. [https://nemsis.org/wp-content/uploads/2025/09/2024-NEMSIS-RDS-350-User-Manual-v3.pdf](https://nemsis.org/wp-content/uploads/2025/09/2024-NEMSIS-RDS-350-User-Manual-v3.pdf)

<a id="ref-18"></a>
18. Code of Virginia. *§ 8.01-225 — Persons rendering emergency care, obstetrical services exempt from liability.* [https://law.lis.virginia.gov/vacode/title8.01/chapter3/section8.01-225/](https://law.lis.virginia.gov/vacode/title8.01/chapter3/section8.01-225/)

<a id="ref-19"></a>
19. Code of Virginia. *§ 8.01-225.3 — Immunity for volunteer first responders en route to an emergency.* [https://law.lis.virginia.gov/vacode/title8.01/chapter3/section8.01-225.3/](https://law.lis.virginia.gov/vacode/title8.01/chapter3/section8.01-225.3/)

<a id="ref-20"></a>
20. Smith, C. M., Lall, R., Fothergill, R. T., et al. (2022). *The effect of the GoodSAM volunteer first-responder app on survival to hospital discharge following out-of-hospital cardiac arrest.* *European Heart Journal – Acute Cardiovascular Care*, 11(1), 20–31. PMC8757292. [https://pmc.ncbi.nlm.nih.gov/articles/PMC8757292/](https://pmc.ncbi.nlm.nih.gov/articles/PMC8757292/)

<a id="ref-21"></a>
21. Ringh, M., Rosenqvist, M., Hollenberg, J., et al. (2015). *Mobile-Phone Dispatch of Laypersons for CPR in Out-of-Hospital Cardiac Arrest.* *New England Journal of Medicine*, 372(24), 2316–2325. doi:10.1056/NEJMoa1406038. [https://www.nejm.org/doi/full/10.1056/NEJMoa1406038](https://www.nejm.org/doi/full/10.1056/NEJMoa1406038)

<a id="ref-22"></a>
22. Virginia Administrative Code. *12VAC5-31-300 — Requirement for EMS agency licensure and EMS certification.* [https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section300/](https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section300/)

<a id="ref-23"></a>
23. Virginia Administrative Code. *12VAC5-31-410 — EMS agency licensure classifications.* [https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section410/](https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section410/)

<a id="ref-24"></a>
24. Virginia Administrative Code. *12VAC5-31-430 — Issuance of an EMS agency license.* [https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section430/](https://law.lis.virginia.gov/admincode/title12/agency5/chapter31/section430/)

<a id="ref-25"></a>
25. Virginia Department of Health, Office of EMS. *Virginia Office of EMS Accreditation for PSAPs and 911 Centers.* [https://www.vdh.virginia.gov/emergency-medical-services/other-ems-programs-and-links/communications/virginia-office-of-ems-accreditation-for-psaps-and-911-centers/](https://www.vdh.virginia.gov/emergency-medical-services/other-ems-programs-and-links/communications/virginia-office-of-ems-accreditation-for-psaps-and-911-centers/)

<a id="ref-26"></a>
26. International Telecommunication Union, Telecommunication Standardization Sector. *Recommendation ITU-T E.161.1: Guidelines to select Emergency Number for public telecommunications networks.* [https://www.itu.int/rec/T-REC-E.161.1](https://www.itu.int/rec/T-REC-E.161.1)

<a id="ref-27"></a>
27. International Telecommunication Union. *ITU-T E.129 Important numbers — country inventory of emergency and important numbers.* [https://www.itu.int/net/itu-t/inrdb/e129_important_numbers.aspx](https://www.itu.int/net/itu-t/inrdb/e129_important_numbers.aspx)

<a id="ref-28"></a>
28. European Parliament and Council. *Directive 2002/22/EC of 7 March 2002 on universal service and users' rights relating to electronic communications networks and services (Universal Service Directive).* EUR-Lex CELEX:32002L0022. [https://eur-lex.europa.eu/LexUriServ/LexUriServ.do?uri=CELEX%3A32002L0022%3AEN%3AHTML](https://eur-lex.europa.eu/LexUriServ/LexUriServ.do?uri=CELEX%3A32002L0022%3AEN%3AHTML)

<a id="ref-29"></a>
29. European Union. *Summary: The 2018 European Electronic Communications Code (Directive 2018/1972).* EUR-Lex. [https://eur-lex.europa.eu/EN/legal-content/summary/european-electronic-communications-code.html](https://eur-lex.europa.eu/EN/legal-content/summary/european-electronic-communications-code.html)

<a id="ref-30"></a>
30. Ministry of Home Affairs, Government of India. *Emergency Response Support System (ERSS).* [https://www.mha.gov.in/en/commoncontent/emergency-response-support-system-erss](https://www.mha.gov.in/en/commoncontent/emergency-response-support-system-erss)

<a id="ref-31"></a>
31. Government of India. *About — Emergency Response Support System in India (112.gov.in).* [https://112.gov.in/about](https://112.gov.in/about)

<a id="ref-32"></a>
32. Republic World. *Supreme Court Orders 112 As One Unified Emergency Response Number, Trauma Rescue Protocol.* May 28, 2026. [https://www.republicworld.com/india/supreme-court-orders-112-as-one-unified-emergency-response-number-trauma-rescue-protocol-2026-05-28-126036](https://www.republicworld.com/india/supreme-court-orders-112-as-one-unified-emergency-response-number-trauma-rescue-protocol-2026-05-28-126036)

<a id="ref-33"></a>
33. European Commission. *Commission Recommendation 2003/558/EC of 25 July 2003 on the processing of caller location information in electronic communication networks for the purpose of location-enhanced emergency call services.* EUR-Lex CELEX:32003H0558. [https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32003H0558](https://eur-lex.europa.eu/legal-content/EN/TXT/?uri=CELEX:32003H0558)

<a id="ref-34"></a>
34. Cooney, D. R. *EMS Medical Director Legal Issues and Liability.* In: *StatPearls* [Internet]. Treasure Island (FL): StatPearls Publishing, last updated 2024. NCBI Bookshelf NBK597344. [https://www.ncbi.nlm.nih.gov/books/NBK597344/](https://www.ncbi.nlm.nih.gov/books/NBK597344/)

<a id="ref-35"></a>
35. Clawson, J. J. *Priority Medical Dispatch Is the Standard of Care.* *JEMS — Journal of Emergency Medical Services*, April 2004. [https://cdn.emergencydispatch.org/iaed/pdf/Priority-Medical-Dispatch-is-the-standard-of-Care-04-JEMS.pdf](https://cdn.emergencydispatch.org/iaed/pdf/Priority-Medical-Dispatch-is-the-standard-of-Care-04-JEMS.pdf)
