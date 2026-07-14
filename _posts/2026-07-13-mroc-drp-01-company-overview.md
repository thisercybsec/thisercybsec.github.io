---
title: "MROC Disaster Recovery Plan — Part 1: Company Overview"
date: 2026-07-13 09:00:00 -0400
categories: [Projects, Disaster Recovery]
tags: [grc, hipaa, business-continuity, is4600]
img_path: /assets/img/posts/mroc-drp-01
---

> This is Part 1 of a 6-part Disaster Recovery Plan built for **Midwest
> Regional Outpatient Clinic (MROC)**, a fictional healthcare clinic created
> for IS 4600: Disaster Recovery Planning at Indiana Tech. All staff names,
> the clinic itself, and its location are fictional and were created for
> coursework purposes only.
{: .prompt-info }

## Company Overview

Midwest Regional Outpatient Clinic is a for-profit medical practice located
in Pawnee, Indiana. The clinic has been in business for approximately 14
years and currently has a staff of forty-three people. This includes six
physicians, nine nurse practitioners, twelve medical assistants, and other
supporting staff. The clinic currently operates six days a week, Monday
through Saturday, and sees an average of 120 patients a day. This occurs
through two sources of care: general family medicine and occupational
health services.

The clinic operates out of a single 12,000 square foot facility, which it
owns outright. It maintains referral relationships with two local hospitals
and several smaller specialty practices in the area. The clinic's gross
annual revenue is approximately $6.2 million, mostly paid through insurance.

## Core Business Operations

MROC's day-to-day operations consist of scheduling, patient intake,
examination, billing, and post-visit follow-up. Every aspect of a patient's
visit either generates or requires updating a health record, which must
follow federal guidelines for security and storage. To meet this
requirement, the clinic uses a cloud-hosted Electronic Health Record (EHR)
system. Nearly every operational function — scheduling, documentation,
prescriptions, lab orders/results, billing, and insurance — interacts with
this EHR.

Billing and insurance claims are handled daily and are one of the clinic's
most time-sensitive functions. The billing staff (four members) can
operate independently of the rest of the clinic but depend on accurate
documentation being entered elsewhere. They also manage monthly billing
for fifteen local companies under contract with the clinic — covering
pre-employment health screenings, drug screenings, worker's compensation
assessments, and return-to-work assessments. These contracts represent a
significant, consistent portion of revenue.

## Technology Infrastructure

- **On-site server** for file storage and administrative functions
- **Single ISP** with no established backup connection
- **Workstations**: a mix of desktop and laptop devices running the current
  version of Windows
- **EHR access**: entirely web-browser based, hosted on the vendor's servers
  — meaning internet access is essential to clinic operations
- **Phone system**: standard business system with backup lines for primary
  line failure
- **Physical security**: keycard readers at critical access points, security
  cameras at entry points and the parking lot, and a separately locked
  server room accessible only to IT administration and clinic management

## Compliance

The clinic employs a single staff member responsible for HIPAA compliance
— ensuring all vendor relationships and communications with referral
partners meet applicable health information regulations.

---

**Next: [Part 2 — Critical Business Functions](#)** *(link once that post is published)*
