---
title: "MROC Disaster Recovery Plan — Part 5: Pre-Event & Post-Event Mitigations"
date: 2026-07-17 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-05
---

> Part 5 of a 6-part Disaster Recovery Plan built for **Midwest Regional
> Outpatient Clinic (MROC)**, a fictional healthcare clinic created for
> IS 4600: Disaster Recovery Planning at Indiana Tech. All staff names are
> fictional.
{: .prompt-info }

[Part 4: Risk Response Strategies](/posts/mroc-disaster-recovery-plan-part-4-risk-response-strategies/)

## Pre-Event Mitigations

This section identifies operational and infrastructure investments MROC
should make *before* a disaster to reduce likelihood or lessen impact.
Unlike the ongoing actions in Part 4's response strategies, these are
capital investments, policy changes, or improvements requiring deliberate
decisions and resource allocation.

### R-001: Fire or Significant Destructive Event
- Install a fully automatic fire suppression system throughout the facility if not already present, prioritizing the server room and electrical closet
- Relocate the on-site server to a fire-rated enclosure/cabinet to protect it from fire and smoke damage
- Transition critical data to nightly automatic cloud backups; establish a quarterly backup test
- Establish a formal lease or memorandum of understanding (MOU) with a partner clinic as an alternate care site; review annually
- Implement a business continuity insurance policy to cover lost revenue during a facility displacement

### R-002: Ransomware Attack on the EHR System
- Implement endpoint detection on all workstations for real-time threat monitoring beyond standard antivirus
- Adopt a zero-trust network architecture to limit lateral movement in the event of a breach
- Establish a formal patch management schedule with maximum time-to-patch requirements based on CVE severity
- Test backup restorations regularly to confirm recovery objectives can be met

### R-003: Extended Billing Platform or Internet Outage
- Procure a 4G/5G mobile hotspot device kept in a ready state at the clinic; train the IT coordinator, billing manager, and clinic manager on activation
- Negotiate an SLA with the third-party billing vendor defining maximum downtime and including provisions for claim extensions during documented downtime
- Establish a second ISP account with a different provider, activatable within minutes of a primary failure
- Print and maintain a 30-day supply of paper billing forms

### R-004: Extended Power Outage
- Purchase and install a gas or propane standby generator with automatic transfer capability, sized for all critical functions
- Install UPS devices on the on-site server and front desk workstations, providing a minimum of 30 minutes of power
- Establish a connection with the utility provider or a monitoring service for timely outage updates
- Designate and train two or more staff members to maintain and operate the standby generator

### R-005: Unavailability of the Occupational Health Coordinator
- Formalize a written cross-training program for the occupational health coordinator role; designate at least one staff member able to function independently in the position
- Create and maintain reference guides covering all critical functions — account contacts, paperwork formatting, drug/injury screening procedures
- Execute a formal written mutual coverage agreement with a nearby occupational health clinic; review annually
- Require one week's notice for all planned absences of the coordinator

### R-006: Disruption to Primary Medical Supply Vendor
- Formalize a secondary vendor relationship, with periodic orders placed to keep the account active
- Establish a tier list of items to be ordered, with clear criticality indicators and minimum inventory levels
- Negotiate preferred-customer or priority status with the primary vendor for crucial items during a disruption

---

## Post-Incident Restoration

This section expands on Part 4's recovery overview with greater detail on
roles, staff re-integration, stakeholder/customer communication, and
formally closing out each type of incident.

### R-001: Fire or Significant Destructive Event
- Once the facility is cleared for re-entry, the clinic manager and IT coordinator conduct a formal damage assessment, categorizing losses as structural, equipment, data, or supply
- IT coordinator recovers all EHR and administrative data from offsite backups onto replacement/temporary hardware at the alternate care site
- Billing manager resumes claims submissions from the alternate site, reconciling visits recorded on paper during the disruption
- Clinic manager coordinates with contractors for repairs or relocation, keeping staff, patients, and employer accounts notified
- Staff return to the restored/new facility once functionality is confirmed by the clinic manager, IT coordinator, and building inspectors
- A formal after-action review is conducted within 30 days of the event; findings are incorporated into an updated DRP

### R-002: Ransomware Attack on the EHR System
- Once the incident response vendor confirms all systems are clean, the IT coordinator restores the EHR and administrative systems from the last verified clean backups
- IT coordinator and vendor conduct a full vulnerability scan of restored systems before reconnecting them to the network
- Clinical and billing staff reconcile paper records accumulated during the event, prioritizing critical/urgent records
- Compliance officer completes required HIPAA breach notifications within the 60-day window, coordinating with legal counsel as needed
- A post-incident report is issued to the clinic manager and compliance officer within 14 days of restoration, documenting the attack vector, affected systems, data impact, and corrective actions

### R-003: Extended Billing Platform or Internet Outage
- Billing manager confirms full system access is restored, verifying connectivity to both the EHR and third-party billing vendor
- Paper bills collected during the outage are reconciled in chronological order, prioritizing claims nearing insurance filing deadlines
- Billing manager reviews paper bills against EHR visit records to ensure no missed entries
- Late claims resulting from the outage are appealed with supporting documentation citing the outage
- Billing manager documents outage duration, cause, revenue impact, and recovery steps; clinic manager reviews and updates the DRP as needed

### R-004: Extended Power Outage
- Once power is restored, IT coordinator confirms the automatic switch back to grid power occurred and the generator was safely shut down
- IT coordinator performs a full systems check on workstations, the on-site server, and networked equipment to confirm no damage occurred
- Front desk staff enter all paper check-ins, appointment notes, and schedule changes into the EHR; front desk supervisor reconciles paper documents against the EHR
- Clinic manager reviews cancelled/rescheduled appointments and coordinates with the front desk supervisor on patient rescheduling priorities
- Generator fuel level is checked and refilled to ensure readiness for the next event

### R-005: Unavailability of the Occupational Health Coordinator
- Upon the coordinator's return, they and the clinic manager jointly review all cases handled by the backup coordinator, confirming reports were transmitted correctly
- Clinic manager contacts all employer accounts to confirm results were received and addresses any concerns
- If a backup clinic provided overflow coverage, the clinic manager follows up to confirm all cases are closed and reciprocal obligations fulfilled
- Clinic manager updates cross-training documentation and reference guides based on any gaps found during the absence

### R-006: Disruption to Primary Medical Supply Vendor
- Once the primary vendor confirms supply restoration, a restocking order is placed to return on-hand inventory to acceptable levels
- Office manager reconciles substitute orders placed during the disruption against standard purchase orders to account for cost discrepancies
- Office manager submits a report to the clinic manager documenting affected supplies, shortage duration, and costs incurred
- Clinic manager and office manager review the incident and determine whether the DRP needs adjustment

---

**Next: [Part 6 — Test Plan & Glossary](/posts/mroc-disaster-recovery-plan-part-6-test-plan-glossary/)**
