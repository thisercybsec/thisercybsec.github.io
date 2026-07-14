---
title: "MROC Disaster Recovery Plan — Part 3: Risk Register Summary"
date: 2026-07-15 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-03
---

> Part 3 of a 6-part Disaster Recovery Plan built for **Midwest Regional
> Outpatient Clinic (MROC)**, a fictional healthcare clinic created for
> IS 4600: Disaster Recovery Planning at Indiana Tech. All staff names are
> fictional.
{: .prompt-info }

[Part 2: Critical Business Functions](/posts/mroc-disaster-recovery-plan-part-2-critical-business-functions/)

## Overview

The following risks were identified through the risk assessment process
conducted for this disaster recovery plan. Each risk was evaluated on
likelihood of occurrence, severity of impact, and controllability, and
classified as High, Medium, or Low on each dimension.

## Risk Summary

| ID | Function | Description | Likelihood | Severity | Controllability |
|---|---|---|---|---|---|
| R-001 | Facility / All Critical Functions | Fire or significant destructive event | Low | High | Medium |
| R-002 | EHR System Access | Ransomware attack | High | High | Medium |
| R-003 | Insurance Billing & Claims Submissions | ISP or billing platform outage | Medium | High | Low |
| R-004 | Patient Scheduling & Check-In | Extended power outage | Medium | High | Medium |
| R-005 | Occupational Health Employer Reporting | Key personnel unavailability | Medium | Medium | High |
| R-006 | Supply & Inventory Ordering | Supply vendor disruption | Low | Low | High |

## Risk Detail

### R-001 — Fire or Significant Destructive Event
**Owner:** Susan Bree, Clinic Manager
**Source:** Environmental/physical — accidental ignition from electrical wiring or equipment; weather events such as tornadoes or a fire at a nearby facility.

- **Likelihood (Low):** Commercial fires or major weather events are statistically uncommon for any single facility, though the risk is never zero.
- **Severity (High):** A significant fire or destructive event would disrupt all operations and could take weeks or months to recover from, effectively triggering every other risk in this register.
- **Controllability (Medium):** Preventive measures — fire suppression systems, regular electrical inspections, smoke detection, and employee training — meaningfully reduce likelihood. Offsite backups and a pre-identified alternate care site are essential for recovery.

### R-002 — Ransomware Attack on the EHR System
**Owner:** Bill Techman, IT Coordinator
**Source:** Internal/external — malicious actors exploiting email phishing, unpatched software, or compromised clinic credentials.

- **Likelihood (High):** Healthcare organizations are frequent ransomware targets, and a small clinic is unlikely to have extensive attack-prevention training in place.
- **Severity (High):** Complete loss of EHR access would disrupt all clinic functions, potentially for a prolonged period, with associated revenue loss and legal exposure.
- **Controllability (Medium):** Technical and operational controls (patching, email filtering, staff training) reduce likelihood, but modern attacks evolve constantly and can bypass controls. Offline backups provide the primary recovery path.

### R-003 — Extended Billing Platform or Internet Outage
**Owner:** Ted Green, Billing Manager
**Source:** External — third-party billing vendor, ISP, or other infrastructure failures.

- **Likelihood (Medium):** MROC operates on a single ISP with no redundancy. Outages aren't routine, but do occur annually. Third-party billing platform reliability is outside the clinic's control and represents a second point of failure.
- **Severity (High):** Unsubmitted claims compound into significant revenue delays, and insurance providers enforce strict filing timelines.
- **Controllability (Low):** ISP and vendor downtime are outside the clinic's direct control. A backup ISP connection or mobile hotspot can mitigate part of the risk; a paper billing workaround is possible but not sustainable long-term.

### R-004 — Extended Power Outage
**Owner:** Mindy Wren, Front Desk Supervisor
**Source:** Environmental/infrastructure — severe weather (tornadoes, high winds, snow/ice storms) common to the region, capable of damaging the electrical grid.

- **Likelihood (Medium):** Indiana experiences a significant and growing number of severe weather events annually, some capable of causing multi-day outages.
- **Severity (High):** A power loss halts all clinic operations, including scheduling and check-in, and can cause significant appointment backlogs beyond a day.
- **Controllability (Medium):** A backup generator or UPS can restore power quickly to critical functions — MROC currently has neither at its facility. Weather itself is not controllable.

### R-005 — Unavailability of the Occupational Health Coordinator
**Owner:** Susan Bree, Clinic Manager
**Source:** Internal — unplanned staff absence due to illness, personal emergency, or unplanned departure without a cross-trained backup.

- **Likelihood (Medium):** Staff illness and turnover are normal parts of business operations. A single coordinator creates a single point of failure, and flu season/illness outbreaks are a recurring threat.
- **Severity (Medium):** Employer accounts operate under contractual timelines; unexpected delays can lead to dissatisfaction or non-renewal of contracts.
- **Controllability (High):** This risk is controllable with proper administrative/HR measures — cross-training for critical roles with documented backup staffing, plus a coverage agreement with nearby health partners.

### R-006 — Disruption to Primary Medical Supply Vendor
**Owner:** Sarah Smith, Office Manager
**Source:** External/supply chain — regional disruptions from weather, transportation issues, infrastructure failures, vendor inventory shortfalls, or broader supply chain issues.

- **Likelihood (Low):** Supply chain disruptions are possible but relatively uncommon with a single reliable supplier. Disruptions are usually short-lived, and on-hand inventory is typically sufficient.
- **Severity (Low):** The clinic maintains several days to weeks of on-hand inventory to mitigate disruptions, and secondary vendors are available for larger disruptions.
- **Controllability (High):** A solid inventory system helps spot shortages early and track supply turnover. Establishing a secondary vendor relationship substantially reduces this risk.

## Rating Legend

- **High** — Significant probability of occurrence or major operational/financial/regulatory impact
- **Medium** — Moderate probability or impact with defined responses
- **Low** — Unlikely to occur or limited impact, addressable with standard operational procedures

---

**Next: [Part 4 — Risk Response Strategies](/posts/mroc-disaster-recovery-plan-part-4-risk-response-strategies/)**
