# McCraw Law Group - AI Receptionist Configuration

**Date:** January 30, 2026
**Purpose:** Capture all information needed to configure AI receptionist

---

## 1. Firm Details

| Field | Value |
|-------|-------|
| Firm Name | McCraw Law Group |
| Practice Area | Personal Injury (Texas only) |
| Address | *[To be provided]* |
| Main Phone | *[To be provided]* |
| Fax | 972-332-2361 |
| Central Email | *[To be provided]* |
| Working Hours | *[To be provided]* |
| Timezone | *[To be provided]* |
| After-Hours Protocol | *[Voicemail / Answering Service / Other?]* |

**Open Questions:**
- What are your office hours? (Mon-Fri? Weekends?)
- What happens after hours? (Voicemail? Answering service? AI takes messages?)

---

## 2. Caller Types & Routing

### 2.1 Routing by Caller Type

| Caller Type | How to Identify | Route To | What AI Can Share | What AI Cannot Share |
|-------------|-----------------|----------|-------------------|---------------------|
| Existing Client | States they have a case, asks about status | Based on case phase (see 2.2) | Case manager name, confirmation of file | Case details, settlement info, legal advice |
| New Potential Client | Mentions accident, injury, needs lawyer | Intake Team | Firm handles personal injury in Texas | Whether we'll take their case |
| Insurance Adjuster | Identifies as adjuster, mentions insurance company, "on a recorded line" | Case Manager | Confirmation of representation, case manager contact | Internal status codes, settlement amounts, medical details |
| Medical Provider / Third Party | Identifies as hospital, clinic, medical provider | Fax redirect: 972-332-2361 | Fax number for inquiries | Representation verification over phone |
| Vendor / Billing | Mentions invoice, payment, accounts payable | Finance / Accounting | — | — |
| Specific Staff Request | Asks for someone by name | That staff member | — | — |
| Spanish Speaker | Speaks Spanish, requests Spanish | Spanish Line | — | — |
| Administrative | Mentions HR, accounting, office matters | Administration | — | — |
| Marketing | Mentions marketing, advertising | Marketing | — | — |

**Open Questions:**
- Is any verification required for insurance adjusters before sharing case info or transferring? (e.g., confirm adjuster name on file, verify claim number?)

---

### 2.2 Routing by Case Phase (Existing Clients)

| Case Phase | Primary Contact | Backup if Unavailable |
|------------|-----------------|----------------------|
| Pre-Lit 00: Under Consideration | Case Manager | — |
| Pre-Lit 01: Process New Client Intake | Case Manager | — |
| Pre-Lit 03: Investigations | Case Manager | — |
| Pre-Lit 04: Client Treating | Case Manager | — |
| Pre-Lit 05: Record Retrieval | Case Manager | — |
| Pre-Lit 06: Demand to be drafted | Case Manager | — |
| Pre-Lit 07: Demand Issued | Attorney | Case Manager |
| Pre-Lit 08: Negotiations | Attorney | Case Manager |
| Pre-Lit 09: Attorney Under Review | Attorney | — |
| Pre-Lit 10: Reject/Withdrawal/Refer Out | Attorney | — |
| Pre-Lit 11: Transfer to Litigation | Attorney | — |
| Settled 01-05 | Case Manager / Paralegal | — |
| Settled 06-07 | April VanHoose | — |
| Settled 08 (GAL - Minor Prove Up) | Attorney assigned to case | — |
| Litigation | Attorney or Paralegal | — |

**Open Questions:**
- For Pre-Lit 07-11, if attorney unavailable, should AI always try case manager next, or go straight to message?
- For Litigation, should AI try attorney first then paralegal, or either?

---

### 2.3 Client Identity Verification

| Step | Action |
|------|--------|
| 1 | Collect full name (first and last) |
| 2 | *[Verify date of birth?]* |
| 3 | Look up case in SmartAdvocate |

**Open Question:**
- Should AI verify client identity with date of birth before discussing anything, or is name sufficient?

---

## 3. Escalation Protocols

### 3.1 Upset / Frustrated Clients

| Scenario | Escalate To |
|----------|-------------|
| Upset Pre-Lit client (attorney & case manager unavailable) | Kyra or Janet |
| Upset Litigation client (attorney & paralegal unavailable) | Vickie Crabb or Emma Burgess |
| Client calling repeatedly without callback | Contact attorney directly |
| Crisis requiring attorney intervention | Task attorney + email summary, offer callback within 1-2 hours or next day |

**Open Questions:**
- How many calls = "repeatedly"? (2? 3? Within what timeframe?)
- Do we have access to call history to detect repeat callers?

---

### 3.2 Transfer Failures

| Scenario | Action |
|----------|--------|
| Transfer doesn't connect | Take detailed message (name, callback number, reason, case reference) |
| After hours | Take message, promise follow-up next business day |

---

## 4. Restricted Actions

### 4.1 Never Do

| Action | Instead Say |
|--------|-------------|
| Verify representation over the phone | "We are unable to verify representation over the phone. Please send your inquiry to our fax: 972-332-2361." |
| Verify for non-verified lienors | Check lien tracking tab first. If not listed, redirect to fax. |

### 4.2 Blocked Entities (Always Redirect to Fax)

- Any hospital or ER room
- American Medical Response
- Optum
- Elevate Financial
- Rawlings
- Intellivo
- Medcap
- Movedocs
- Gain or Gain Servicing

---

## 5. Staff Directory

| Name | Role/Department | Extension | Email |
|------|-----------------|-----------|-------|
| Kyra | Pre-Lit Escalation | *[Needed]* | *[Needed]* |
| Janet | Pre-Lit Escalation | *[Needed]* | *[Needed]* |
| Vickie Crabb | Litigation Escalation | *[Needed]* | *[Needed]* |
| Emma Burgess | Litigation Escalation | *[Needed]* | *[Needed]* |
| April VanHoose | Settlements (06-07) | *[Needed]* | *[Needed]* |
| *[Record Clerk]* | Medical Provider Inquiries | *[Needed]* | *[Needed]* |
| *[Add staff...]* | | | |

---

## 6. Department Phone Numbers

| Department | Phone/Extension |
|------------|-----------------|
| Intake Team | *[Needed]* |
| Finance / Accounting | *[Needed]* |
| Administration | *[Needed]* |
| Marketing | *[Needed]* |
| Spanish Line | *[Needed]* |
| Customer Success / General | *[Needed]* |

---

## 7. Email Recipients

| Purpose | Email Address |
|---------|---------------|
| New PNC Notifications | *[Needed]* |
| Intake Manager Escalation | *[Needed]* |
| Client Crisis / Urgent Issues | *[Needed]* |
| General Inquiries | *[Needed]* |

---

## 8. Software Integrations

| System | Purpose | Access Details |
|--------|---------|----------------|
| SmartAdvocate | Case management, case lookup | *[API Key / URL needed]* |
| Salesforce | Lead/PNC intake | *[If applicable]* |
| RingCentral | Phone system | *[If applicable]* |
| DocuSign | Intake form signing | *[If applicable]* |

**Open Question:**
- How does your team currently check staff availability? (Teams status? Phone system? Calendar?)

---

## 9. Open Questions Summary

| # | Question | Section |
|---|----------|---------|
| 1 | What are your office hours? | 1 |
| 2 | What happens after hours? | 1 |
| 3 | Is verification required for insurance adjusters? | 2.1 |
| 4 | For Pre-Lit 07-11, if attorney unavailable, try case manager or take message? | 2.2 |
| 5 | For Litigation, try attorney first then paralegal, or either? | 2.2 |
| 6 | Should AI verify client identity with DOB? | 2.3 |
| 7 | How many calls = "repeatedly"? | 3.1 |
| 8 | Do we have access to call history? | 3.1 |
| 9 | How does your team check staff availability? | 8 |

---

*Document prepared by Receptionist Pro*
