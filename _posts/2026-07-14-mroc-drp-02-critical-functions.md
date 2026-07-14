---
title: "MROC Disaster Recovery Plan — Part 2: Critical Business Functions"
date: 2026-07-14 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-02
---

> Part 2 of a 6-part Disaster Recovery Plan built for **Midwest Regional
> Outpatient Clinic (MROC)**, a fictional healthcare clinic created for
> IS 4600: Disaster Recovery Planning at Indiana Tech. All staff names are
> fictional.
{: .prompt-info }

[Part 1: Company Overview](/posts/mroc-drp-01-company-overview/)

## Overview

The following critical business functions were identified through a needs
assessment conducted as part of this disaster recovery plan. Each function
was evaluated based on its sensitivity to downtime, revenue impact,
regulatory impact, and its effect on the clinic's other functions. Of the
five functions identified, three were classified High criticality, one
Medium, and one Low.

## Critical Function Summary

| Function | Criticality | Max Downtime | Primary Owner |
|---|---|---|---|
| EHR System Access | High | 8 Hours | Bill Techman (IT Coordinator) |
| Insurance Billing & Claims Submissions | High | 48 Hours | Ted Green (Billing Manager) |
| Patient Scheduling & Check-In | High | 8 Hours | Mindy Wren (Front Desk Supervisor) |
| Occupational Health Employer Reporting | Medium | 3 Business Days | Clark Bigsby (Occ. Health Coordinator) |
| Supply & Inventory Ordering | Low | 1 Week | Sarah Smith (Office Manager) |

## Function Detail

### EHR System Access
**Criticality:** High &nbsp;|&nbsp; **Max Downtime:** 8 Hours
**Owner:** Bill Techman, IT Coordinator — Alt 1: Susan Bree (Clinic Manager) — Alt 2: EHR vendor support

- **Required resources:** Internet connectivity, staff workstations, EHR login credentials, downtime paper forms, backup browser-capable devices
- **Impacted functions:** Patient scheduling, clinical documentation, e-prescribing, billing, lab order tracking
- **Downtime process:** Activate downtime paper forms for all clinic documentation. Contact EHR vendor support and troubleshoot internet connectivity. Once restored, enter all paper documentation into the EHR system and reconcile.

### Insurance Billing & Claims Submissions
**Criticality:** High &nbsp;|&nbsp; **Max Downtime:** 48 Hours
**Owner:** Ted Green, Billing Manager — Alt 1: Sally Klame (Senior Billing Specialist) — Alt 2: Third-party billing vendor

- **Required resources:** EHR access, billing platform login, internet connectivity, paper bills, printer/fax machine
- **Impacted functions:** Accounts receivable, cash flow, approval/denial management, patient invoicing
- **Downtime process:** Clinical staff complete paper bills. Billing team batches claims for submission upon system recovery. Third-party vendors are notified. All denied claims are reviewed and resubmitted.

### Patient Scheduling & Check-In
**Criticality:** High &nbsp;|&nbsp; **Max Downtime:** 8 Hours
**Owner:** Mindy Wren, Front Desk Supervisor — Alt 1: Susan Bree (Clinic Manager)

- **Required resources:** EHR access, phone systems, paper appointment logs, insurance verification tool, patient sign-in sheets
- **Impacted functions:** Patient flow, clinical operations, billing initiation
- **Downtime process:** Use paper sign-in sheets and appointment logs. Verify insurance by phone with carriers. Update the EHR with all manual entries upon system recovery.

### Occupational Health Employer Reporting
**Criticality:** Medium &nbsp;|&nbsp; **Max Downtime:** 3 Business Days
**Owner:** Clark Bigsby, Occupational Health Coordinator — Alt 1: Susan Bree (Clinic Manager)

- **Required resources:** EHR access, employer contact list, fax machine, email, paper result forms
- **Impacted functions:** Employer billing, employer relations, contractual compliance
- **Downtime process:** Communicate time-sensitive results via phone, email, or fax as a temporary measure. Document all communications. Transmit all formal results upon system recovery.

### Supply & Inventory Ordering
**Criticality:** Low &nbsp;|&nbsp; **Max Downtime:** 1 Week
**Owner:** Sarah Smith, Office Manager — Alt 1: Susan Bree (Clinic Manager)

- **Required resources:** Inventory tracking system, vendor contact list, purchase order forms, clinic credit account
- **Impacted functions:** Clinical operations, patient care
- **Downtime process:** Perform a manual inventory of all on-hand supplies. Contact vendors by phone to place critical orders. Use existing credit accounts until payment methods are restored. Document all transactions.

## Criticality Legend

- **High** — Maximum sensitivity to downtime. Immediate revenue, patient safety, or regulatory impact.
- **Medium** — Moderate impact. Disruption affects contractual or reputational obligations within days.
- **Low** — Minimal immediate impact. On-site resources or manual workarounds can sustain operations for 1+ weeks.

---

**Next: [Part 3 — Risk Register Summary](/posts/mroc-drp-03-risk-register/)**
