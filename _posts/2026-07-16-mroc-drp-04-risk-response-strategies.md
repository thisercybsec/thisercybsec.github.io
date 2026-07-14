---
title: "MROC Disaster Recovery Plan — Part 4: Risk Response Strategies"
date: 2026-07-16 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-04
---

> Part 4 of a 6-part Disaster Recovery Plan built for **Midwest Regional
> Outpatient Clinic (MROC)**, a fictional healthcare clinic created for
> IS 4600: Disaster Recovery Planning at Indiana Tech. All staff names are
> fictional.
{: .prompt-info }

[Part 3: Risk Register Summary](/posts/mroc-drp-03-risk-register/)

## Overview

This section outlines a response strategy for each risk identified in the
MROC risk register. Each risk is broken into three stages:

- **Stage 1 — Preventive/Preparatory:** actions taken *before* an event
- **Stage 2 — Immediate Response:** actions taken *during* an event
- **Stage 3 — Recovery/Restoration:** actions taken *after* an event to return to normal operations

## R-001: Fire or Significant Destructive Event Affecting Facility

**Risk Owner:** Susan Bree, Clinic Manager

**Stage 1 — Preventive/Preparatory**
- Maintain fire suppression systems (smoke detectors, extinguishers) with scheduled inspections from a certified vendor
- Conduct an annual electrical safety inspection
- Maintain offsite encrypted backups of EHR data, billing records, and administrative files on a regular basis
- Establish and document alternate care sites (partner clinics, leased medical spaces) that can be activated on short notice
- Conduct fire and severe weather drills with predefined routes on a regular basis
- Maintain adequate insurance for property and business operations

**Stage 2 — Immediate Response**
- Activate the building evacuation/severe weather plan
- Account for all staff and patients
- Front desk or first available manager contacts 911 and notifies the clinic manager
- Clinic manager initiates the emergency communication plan
- Once the immediate event is over and it's safe to do so, the manager assesses if any assets can be recovered

**Stage 3 — Recovery/Restoration**
- Clinic manager and IT coordinator assess the extent of damage to the facility, on-site server, and records with insurance adjusters; file applicable claims
- Activate the alternate care site
- IT coordinator restores EHR access via offsite backups to new or backup hardware
- Clinic manager notifies patients, employer accounts, insurance carriers, and regulatory bodies of operational changes
- Conduct a post-incident review to evaluate the plan's effectiveness

## R-002: Ransomware Attack on the EHR System

**Risk Owner:** Bill Techman, IT Coordinator

**Stage 1 — Preventive/Preparatory**
- Maintain up-to-date patches and antivirus/antimalware software on all workstations and the on-site server
- Implement email filtering and staff phishing-awareness training
- Enforce multi-factor authentication
- Maintain regular encrypted backups of EHR data, air-gapped and offsite
- Maintain paper documents needed if the system becomes inaccessible (printed and organized)
- Establish and maintain a relationship with a cybersecurity incident response vendor

**Stage 2 — Immediate Response**
- The staff member who notices ransomware indicators immediately disconnects the system from the network and notifies the IT coordinator
- IT coordinator isolates affected systems to prevent further spread
- Notify the clinic manager and activate paper documentation procedures
- IT coordinator contacts the cybersecurity incident response vendor if needed to investigate and contain the attack
- Clinic manager and compliance officer assess whether the incident qualifies as a HIPAA breach

**Stage 3 — Recovery/Restoration**
- IT coordinator restores systems from offline backups after confirming the threat has been removed
- Staff enter all paper documentation used during the outage once systems are restored, checking for accuracy
- Compliance officer gathers information required for HIPAA breach notifications to patients and regulatory bodies
- Clinic manager and IT coordinator conduct a root-cause investigation to identify how the attack occurred and close any vulnerabilities
- Update staff training and controls based on lessons learned
- Conduct a post-incident review to evaluate the plan's effectiveness

## R-003: Extended Outage of Billing Platform or Internet Connectivity

**Risk Owner:** Ted Green, Billing Manager

**Stage 1 — Preventive/Preparatory**
- Investigate and budget for a secondary internet connection or mobile hotspot
- Maintain a supply of paper billing forms
- Document a manual claims procedure and train staff on it

**Stage 2 — Immediate Response**
- Billing manager determines the source of the outage (ISP, internal network, or third-party billing vendor)
- If the outage is with the ISP, the IT coordinator activates the hotspot or secondary connection if available
- If the outage is with the internal network, the IT coordinator leads the investigation
- If the outage is with the third-party billing vendor, they are notified
- Clinic staff complete paper billing/claims forms for all visits

**Stage 3 — Recovery/Restoration**
- Billing manager and staff batch and submit all backlogged claims as soon as possible, prioritizing claims nearing deadlines
- Billing manager monitors claims denied due to late filing caused by the outage and appeals where necessary
- Billing manager compares paper billing to EHR records to ensure all bills were issued
- Document the outage's duration and cost to justify a secondary connection, if still needed
- Conduct a post-incident review to evaluate the plan's effectiveness

## R-004: Extended Power Outage Affecting Clinic Operations

**Risk Owner:** Mindy Wren, Front Desk Supervisor

**Stage 1 — Preventive/Preparatory**
- Evaluate and budget for a standby generator capable of powering critical functions
- Acquire and maintain UPS units for critical workstations and the on-site server
- Maintain a supply of paper check-in and appointment forms
- Print the day's appointments ahead of time for offline access

**Stage 2 — Immediate Response**
- Front desk supervisor switches to paper forms for check-ins and appointments
- Staff verify insurance information by phone or defer until systems are restored
- Clinic manager determines whether the clinic can safely remain open or should close early and reschedule remaining appointments
- If a generator is available, the IT coordinator or a trained staff member activates it to restore power
- Front desk contacts patients with upcoming appointments to notify of delays or rescheduling

**Stage 3 — Recovery/Restoration**
- Once power is restored, all manually logged appointments, check-ins, and notes are entered into the EHR system
- Clinic manager reviews affected appointments and makes arrangements to accommodate patients
- IT coordinator confirms all systems and equipment are working correctly without data loss or damage
- Document the outage's duration and cost to justify a generator or UPS purchase, if still needed
- Conduct a post-incident review to evaluate the plan's effectiveness

## R-005: Unavailability of the Occupational Health Coordinator

**Risk Owner:** Susan Bree, Clinic Manager

**Stage 1 — Preventive/Preparatory**
- Cross-train at least one staff member on the core functions of the occupational health reporting role
- Maintain clear documentation of all protocols, procedures, and backup staffing
- Establish a mutual coverage relationship with nearby clinics for overflow or emergency coverage

**Stage 2 — Immediate Response**
- Clinic manager identifies the cross-trained staff member and assigns them to occupational health reporting duties
- Clinic manager identifies any time-sensitive cases in the backlog and prioritizes them for the backup staff member
- If the absence is expected to be prolonged, the clinic manager contacts the backup clinic for overflow arrangements

**Stage 3 — Recovery/Restoration**
- The occupational health coordinator and clinic manager review all cases handled for completeness and accuracy
- Conduct a post-incident review to evaluate the plan's effectiveness and whether additional cross-training is needed

## R-006: Disruption to Primary Medical Supply Vendor

**Risk Owner:** Sarah Smith, Office Manager

**Stage 1 — Preventive/Preparatory**
- Maintain a minimum on-hand inventory buffer of one to two weeks of high-use supplies
- Identify and establish an account with at least one secondary/backup medical supply vendor prior to any disruption
- Maintain an inventory tracking system to monitor usage rates and enable timely orders

**Stage 2 — Immediate Response**
- Office manager performs a manual inventory count to determine current supply levels and identify potential shortages
- Office manager contacts the secondary vendor to order critical or low-inventory items
- Clinic manager is notified of any supplies expected to run low or deplete before restock, so services can be adjusted as needed

**Stage 3 — Recovery/Restoration**
- Office manager documents all transactions and substitute orders placed, for accounting and reconciliation
- Office manager reviews the cause of the disruption and works with the vendor on measures to prevent recurrence
- Clinic manager and office manager review whether the inventory buffer, primary vendor, or secondary vendor need adjustment based on the event

---

**Next: [Part 5 — Pre-Event & Post-Event Mitigations](/posts/mroc-drp-05-mitigations/)**
