---
title: "MROC Disaster Recovery Plan — Part 6: Test Plan & Glossary"
date: 2026-07-18 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-06
---

> Part 6 (final) of a 6-part Disaster Recovery Plan built for **Midwest
> Regional Outpatient Clinic (MROC)**, a fictional healthcare clinic
> created for IS 4600: Disaster Recovery Planning at Indiana Tech. All
> staff names are fictional.
{: .prompt-info }

[Part 5: Pre-Event & Post-Event Mitigations](/posts/mroc-disaster-recovery-plan-part-5-pre-event-post-event-mitigations/)

## Overview

A disaster recovery plan is only valid once it has been tested and proven
to work. This section provides a validation scenario for each identified
risk, designed to test the response strategy, pre-event mitigations, and
restoration procedures under realistic conditions. Following initial
testing, an ongoing testing schedule is followed (below).

## R-001: Fire or Significant Destructive Event

**Scenario:** A fire originating in an electrical panel in the administrative wing is discovered by staff during business hours. It spreads to the server room before suppression activates, disabling the on-site server and three workstations. The fire marshal condemns the facility for a minimum of two weeks.

**Validation Objectives:** Confirm all staff can execute the evacuation plan and account for everyone within 10 minutes. Confirm the IT coordinator can restore EHR access from offsite backups onto temporary hardware within 24 hours. Confirm the alternate care site can be activated and accept patients within 48 hours. Confirm the clinic manager can complete all required notifications to personnel, patients, employer accounts, and regulators.

**Test Method:** Announced tabletop exercise with the clinic manager, IT coordinator, front desk supervisor, and billing manager, walking through each step of the response plan against the scenario.

**Pass Criteria:** All evacuation roles clearly understood and executed. EHR access restored properly in the test environment. Alternate site contact made and activation steps confirmed. Notification checklists completed.

## R-002: Ransomware Attack on the EHR System

**Scenario:** A staff member opens a phishing email and unknowingly installs ransomware, which begins encrypting files and attempting to spread to the on-site server. A ransom message appears mid-morning.

**Validation Objectives:** Confirm staff can correctly identify ransomware symptoms and immediately disconnect the device without waiting on IT instructions. Confirm the IT coordinator can isolate affected devices from the rest of the network in under 30 minutes. Confirm paper documentation can begin within 15 minutes of the disruption. Confirm systems can be restored within 8 hours.

**Test Method:** Tabletop exercise simulating the discovery of ransomware at 10:00 AM during business operations, including the IT coordinator, compliance officer, clinic manager, and at least two staff members. Backup procedures verified separately.

**Pass Criteria:** Infected device is isolated before the facilitator announces it has spread to the on-site server. Paper forms located and distributed successfully. IT coordinator demonstrates workstation restoration from backups. Compliance officer can identify whether the scenario would constitute a HIPAA breach.

## R-003: Extended Outage of Billing Platform or Internet Connectivity

**Scenario:** The clinic's primary ISP experiences an unplanned outage beginning at 8:00 AM, expected to last 6–8 hours, affecting access to the EHR and billing platform.

**Validation Objectives:** Confirm the billing manager can identify the outage source and activate the hotspot within 20 minutes. Confirm paper billing documents can be located and implemented. Confirm paper billing can be reconciled with the EHR within 24 hours of ISP restoration.

**Test Method:** Announced drill in which the IT coordinator simulates an ISP outage; billing staff use paper billing for a 2-hour window, then access is restored and reconciliation performed.

**Pass Criteria:** Hotspot activated and online access restored within 20 minutes. Paper billing forms completed correctly for visits within the test window. Post-reconciliation shows zero missing claims.

## R-004: Extended Power Outage

**Scenario:** A severe ice storm causes a grid-wide power failure affecting the clinic at 9:00 AM, with the utility estimating 12–24 hours of outage.

**Validation Objectives:** Confirm front desk staff can locate and implement paper check-in/appointment documentation within 10 minutes of power loss. Confirm the UPS provides adequate power for a safe server shutdown. Confirm the generator can restore power to critical systems (testable during non-business hours). Confirm the clinic manager can decide to remain open or close within 30 minutes and communicate that to staff and patients.

**Test Method:** Tabletop exercise with the front desk supervisor, clinic manager, and IT coordinator simulating the outage from discovery through the open/close decision; UPS and generator functionality verified separately.

**Pass Criteria:** Paper forms located and implemented within 10 minutes of the outage. Clinic manager demonstrates clear decision-making about remaining open or closing. IT coordinator confirms UPS and generator functionality.

## R-005: Unavailability of the Occupational Health Coordinator

**Scenario:** The occupational health coordinator calls in sick on a Monday with an illness expected to last several days. Three employer drug screenings and two return-to-work evaluations are scheduled during the timeframe.

**Validation Objectives:** Confirm the clinic manager can identify and assign the cross-trained backup staff member within one hour of notification. Confirm the backup staff member can locate reference guides for the screenings/evaluations. Confirm employers can be contacted and informed of scheduling changes. Confirm the partner clinic can be notified of any overflow and arrangements made the same business day.

**Test Method:** Tabletop exercise with the clinic manager and designated backup staff member, walking through identification of pending appointments, use of the reference guide, and escalation to partner clinic overflow.

**Pass Criteria:** Backup staff member identified and assigned within one hour. Reference guides found and contain all needed information. Partner clinic and employer contact information on file and up to date.

## R-006: Disruption to Primary Medical Supply Vendor

**Scenario:** The primary medical supply vendor notifies the clinic of a disruption that will delay pending orders by 7–10 business days. Current on-hand inventory of critical supplies is expected to last only 10 days.

**Validation Objectives:** Confirm the office manager can perform a complete inventory count and identify at-risk items within two hours of notification. Confirm the secondary vendor account is active and can fulfill a priority order. Confirm the clinic manager is informed and service adjustments made as needed.

**Test Method:** Announced drill in which the office manager performs a manual inventory count, identifies at-risk items, and contacts the secondary vendor to verify account status, pricing, and priority-order availability (without placing an actual order).

**Pass Criteria:** Manual inventory conducted within a 2-hour window. All at-risk items correctly identified. Secondary vendor account active and capable of a priority order. Clinic manager informed of the disruption and any needed service adjustments.

## Annual Test Schedule

| Risk ID | Test Type | Responsible Party | Frequency |
|---|---|---|---|
| R-001 | Tabletop Exercise | Susan Bree (Clinic Manager) | Annual |
| R-002 | Backup drill (Spring) + Tabletop (Fall) | Bill Techman (IT Coordinator) | Semi-Annual |
| R-003 | Functional Drill | Ted Green (Billing Manager) | Annual |
| R-004 | Tabletop Exercise + Equipment Check | Mindy Wren / Bill Techman | Annual |
| R-005 | Tabletop Exercise | Susan Bree (Clinic Manager) | Annual |
| R-006 | Functional Drill | Sarah Smith (Office Manager) | Annual |
| ALL | Full DRP Documentation Review | Susan Bree (Clinic Manager) | Annual |
| ALL | Backup Restoration Verification | Bill Techman (IT Coordinator) | Quarterly |
| ALL | Staff Downtime Procedure Training | Susan Bree (Clinic Manager) | Annual |

**Legend:**
- **Tabletop Exercise** — facilitated discussion-based scenario walkthrough with key personnel, no live systems affected
- **Functional Drill** — partial activation of procedures using real tools/processes in a controlled environment
- **Backup Restoration Drill** — technical test of data recovery from offline/offsite backups in an isolated environment
- **Equipment Inspection** — physical check of UPS units, generators, fire suppression, and other readiness hardware
- **Full Review** — annual review of the complete DRP to reflect operational or staff changes

---

## Glossary

| Term | Definition |
|---|---|
| Business Continuity | The capability of an organization to continue delivery of products or services at a pre-defined level following a disruption |
| Disaster Recovery Plan | A documented process or set of procedures to recover and protect business IT infrastructure in the event of a disaster |
| EHR (Electronic Health Record) | A digital version of a patient's paper chart, accessible in real time and shared across healthcare providers |
| HIPAA | Federal legislation providing data privacy and security provisions for safeguarding medical information |
| MOU (Memorandum of Understanding) | A formal agreement between parties outlining the terms and responsibilities of an agreement |
| Ransomware | Malicious software designed to block access to a system or encrypt data until a ransom is paid |
| Risk Register | A document that identifies, assesses, and prioritizes potential risks and documents a planned response |
| UPS (Uninterruptible Power Supply) | A device providing emergency power to connected systems when the main power source fails |

---

This closes out the six-part MROC Disaster Recovery Plan. Full source
documents (risk register, critical functions chart, test schedule matrix)
were developed as part of IS 4600: Disaster Recovery Planning at Indiana
Tech.
