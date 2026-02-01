# SOP Analysis Report: McCraw Law Group

**Generated:** 2026-01-29
**Analyzed By:** Claude Code SOP Analysis Skill
**SOP Documents:**
- Intake Call Routing Cheat Sheet.pdf
- SOP_-_Client_Crisis_Management_Communication.pdf
- SOP_-_Inbound_Caller_Intake_and_Routing_Workflow.pdf

---

## Summary

| Category | Count |
|----------|-------|
| Clear & Aligned | 8 |
| Needs Work | 7 |
| Needs Clarity | 6 |
| Missing Data | 12 |

---

## 1. CLEAR AND ALIGNED

*Workflows that match existing agent capabilities exactly.*

### 1.1 Standard Greeting and Caller Identification

**SOP Requirement:** "Intake Specialist answers the call using the firm's standard greeting. Obtain the following from the caller: Full name, Subject or reason for the call, Person or department they are trying to reach"

**Agent:** Greeter Classifier (#1)
**Tools:** handoff_tool, staff_directory_lookup
**Status:** FULLY SUPPORTED

**Current Implementation:**
The Greeter Classifier agent collects caller name and purpose, then routes to the appropriate specialist agent based on caller type classification. This matches the SOP's intake workflow exactly.

---

### 1.2 Case-Related Calls Routing to Case Handler

**SOP Requirement:** "Case-Related Calls: Route to the appropriate Case Handler or team member assigned to the case."

**Agent:** Existing Client (#3), Pre-Identified Client (#2)
**Tools:** search_case_details, staff_directory_lookup, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
The system looks up the case via name/DOB verification, retrieves the assigned case manager, and offers transfer to that specific staff member. This aligns with the SOP's case routing requirement.

---

### 1.3 New Client (PNC) Routing

**SOP Requirement:** "PNC or New Client Calls: Follow the workflow in the SOP: Screening a Potential New Client (PNC)."

**Agent:** New Client (#6)
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
The New Client agent handles potential new clients, collects basic case type information, and transfers to the intake team. This matches the SOP's requirement to follow the PNC screening workflow.

---

### 1.4 Insurance Adjuster Routing to Case Manager

**SOP Requirement:** "If an insurance adjustor is calling for a case status update, the call will go to case manager."

**Agent:** Insurance Adjuster (#4)
**Tools:** search_case_details, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
The Insurance Adjuster agent performs case lookup, can share limited status information in plain English, and routes to the case manager. This directly matches the SOP requirement.

---

### 1.5 Fax Redirect for Third-Party Inquiries

**SOP Requirement:** "We are unable to verify representation or provide updates over the phone. We ask that you place an inquiry through our fax number. 972-332-2361."

**Agent:** Medical Provider (#5)
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
The Medical Provider agent follows a strict third-party policy with fax redirect only. This aligns perfectly with McCraw's "DO NOT verify representation over the phone" policy.

---

### 1.6 Direct Staff Transfer Requests

**SOP Requirement:** "Person or department they are trying to reach" + routing to specific staff members

**Agent:** Direct Staff Request (#8)
**Tools:** staff_directory_lookup, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
The Direct Staff Request agent handles callers asking for specific staff members by name, performs directory lookup, and transfers directly. This supports the SOP's staff-specific routing needs.

---

### 1.7 Upset Client Escalation

**SOP Requirement:** "If a client is extremely upset and neither the attorney nor case manager is available, please call Kyra or Janet so they can speak to the client."

**Agent:** Existing Client (#3), Pre-Identified Client (#2)
**Tools:** transfer_call (escalation destination)
**Status:** PARTIALLY SUPPORTED

**Current Implementation:**
The system supports escalation routing for frustrated callers via the `escalation` caller_type transfer destination. The specific escalation contacts (Kyra, Janet for Pre-Lit; Vickie Crabb, Emma Burgess for Litigation) need to be configured.

---

### 1.8 Message Taking When Unavailable

**SOP Requirement:** "If staff is unavailable: Take a detailed message including all caller info and the subject of the call."

**Agent:** All agents (when `is_open = false`)
**Tools:** N/A (message mode)
**Status:** FULLY SUPPORTED

**Current Implementation:**
When office hours are closed or transfers fail, agents switch to message-taking mode, collecting caller information for follow-up. This matches the SOP's unavailability handling.

---

## 2. CLEAR BUT NEEDS WORK

*Requirements that are understood but need prompt/config changes.*

### 2.1 Status-Based Routing Logic (Pre-Lit vs. Attorneys)

**SOP Says:**
- "Pre-Lit 00 through Pre-Lit 06: Case Managers"
- "Pre-Lit 07 through Pre-Lit 11: Attorneys"
- "Settled 01 to Settled 05: Case manager/paralegal"
- "Settled 06 and Settled 07: April VanHoose"
- "Settled 08 - GAL - Minor Prove Up: Attorney assigned"

**Current Gap:** The system currently routes all existing client calls to the case manager assigned in the CRM. It does not have status-based routing logic that differentiates between case manager vs. attorney vs. specific staff (like April VanHoose).

**Implementation Needed:**
1. Add case status field to `search_case_details` API response
2. Implement status-based routing logic in Existing Client agent prompt
3. Create status-to-destination mapping:
   - Pre-Lit 00-06 → Case Manager
   - Pre-Lit 07-11 → Attorney
   - Settled 01-05 → Case Manager/Paralegal
   - Settled 06-07 → April VanHoose (hardcoded staff_id)
   - Settled 08 → Attorney
   - Lit → Attorney or Paralegal
4. Update transfer logic to use attorney field when status indicates attorney routing

**Effort:** High

**Considerations:**
- Requires SmartAdvocate API to return case status
- May need new staff lookup logic for attorney vs. case manager
- Status codes must match exactly what SmartAdvocate returns

---

### 2.2 Attorney Transfer with Case Manager Fallback

**SOP Says:** "If the client asks to speak to the attorney, transfer to the attorney if they are available. If they are not available call the case manager."

**Current Gap:** The system routes to case manager by default. It doesn't have explicit "attorney request" handling with fallback logic.

**Implementation Needed:**
1. Add attorney detection to Existing Client agent ("I want to speak to my attorney", "Can I talk to the lawyer")
2. Retrieve attorney field from case lookup (in addition to case manager)
3. Implement attempt-attorney-first-then-fallback logic
4. Add prompt instructions for explaining the fallback: "The attorney is unavailable, but I can connect you with your case manager..."

**Effort:** Medium

**Considerations:**
- Requires real-time availability check OR sequential transfer attempts
- SmartAdvocate API must return attorney assignment
- Fallback messaging must be handled gracefully

---

### 2.3 Repeated Caller Detection and Attorney Escalation

**SOP Says:** "If you see a client, call repeatedly and have not been given a call back, call the attorney to see if they can take the call."

**Current Gap:** No mechanism exists to detect repeat callers or track callback status. The system treats each call independently.

**Implementation Needed:**
1. Add call frequency tracking (requires backend/CRM integration)
2. Define "repeated caller" threshold (e.g., 3+ calls in 48 hours)
3. Add escalation path to attorney for repeat callers
4. Pass repeat-caller flag via call metadata

**Effort:** High

**Considerations:**
- Requires integration with call history data
- May need SmartAdvocate task/note query
- Privacy considerations around tracking call patterns

---

### 2.4 Verified Lienor vs. Unverified Third-Party Distinction

**SOP Says:** "DO NOT verify representation anyone who is not a verified lienor in the lien tracking tab"

**Current Gap:** The Medical Provider agent treats ALL medical providers/third parties the same way (fax redirect). The SOP indicates that VERIFIED lienors in the lien tracking tab should receive different treatment - specifically, they can be transferred to case manager or record clerk.

**Implementation Needed:**
1. Add lienor verification lookup to SmartAdvocate API
2. Create bifurcated Medical Provider flow:
   - Unknown/unverified → Fax redirect only (current behavior)
   - Verified lienor → Transfer to case manager or record clerk
3. Update prompt to ask qualifying questions to distinguish

**Effort:** High

**Considerations:**
- Requires new API endpoint or field from SmartAdvocate
- List of blocked entities (Optum, Rawlings, etc.) should be configurable
- Legal review needed on info sharing with verified lienors

---

### 2.5 Administrative and Marketing Call Routing

**SOP Says:**
- "Administrative Calls: Route to the Administration team (e.g., HR, Accounting, Office Management)"
- "Marketing Calls: Route to the Marketing department"

**Current Gap:** The current squad handles Vendors (finance/billing), but doesn't have specific agents or routing for HR, Accounting, Office Management, or Marketing departments.

**Implementation Needed:**
1. Add "Administrative" caller classification to Greeter
2. Add "Marketing" caller classification to Greeter
3. Create transfer destinations:
   - `administrative` → Administration team number
   - `marketing` → Marketing department number
4. Update Greeter routing logic

**Effort:** Low

**Considerations:**
- May be able to handle via Fallback agent with department selection
- Need phone numbers for Administration and Marketing

---

### 2.6 Litigation Case Routing

**SOP Says:** "Lit calls can be routed to the handling attorney or paralegal. If a lit client is upset and neither the attorney nor paralegal is available, please call Vickie Crabb or Emma Burgess."

**Current Gap:** System doesn't distinguish Pre-Lit from Litigation cases. Lit cases have different escalation contacts than Pre-Lit.

**Implementation Needed:**
1. Add case type (Pre-Lit vs. Lit) detection from SmartAdvocate
2. Route Lit cases to attorney OR paralegal (not case manager)
3. Configure separate escalation contacts:
   - Pre-Lit escalation → Kyra, Janet
   - Lit escalation → Vickie Crabb, Emma Burgess
4. Update upset client detection to use appropriate escalation path

**Effort:** Medium

**Considerations:**
- Requires case type field from API
- Paralegal may not be a current transfer option (need staff role data)
- Two separate escalation phone trees needed

---

### 2.7 Blocked Third-Party Entity List

**SOP Says:** Lists specific entities that should NEVER receive representation verification:
- Any hospital or ER room
- American Medical Response
- Optum, Elevate Financial, Rawlings, Intellivo, Medcap, Movedocs, Gain or Gain Servicing

**Current Gap:** Medical Provider agent uses blanket fax-redirect for all third parties. There's no explicit blocklist or differentiated handling.

**Implementation Needed:**
1. Create configurable blocklist of entity names
2. Add organization name detection to Medical Provider agent
3. Ensure blocklisted entities ALWAYS get fax redirect, regardless of other factors
4. Add these to prompt as explicit examples for pattern matching

**Effort:** Low

**Considerations:**
- Blocklist should be configurable (not hardcoded in prompt)
- Caller may not always identify their organization
- Consider fuzzy matching for variations (e.g., "Gain Servicing" vs "Gain")

---

## 3. NEEDS CLARITY

*Ambiguous SOP sections requiring client clarification.*

### 3.1 "Record Clerk" Role and Routing

**Quote:** "If a medical facility that is in the medical providers tab in SA is calling regarding a client, transfer over to case manager or record clerk depending on the nature of the call."

**Ambiguity:** What determines "the nature of the call" that would route to record clerk vs. case manager? What types of inquiries go to each?

**Questions to Resolve:**
1. What specific call purposes should route to the record clerk?
2. Is there a designated record clerk, or is this a role multiple people fill?
3. What is the record clerk's phone number or extension?

---

### 3.2 Intake Manager Role in Escalation

**Quote:** "Communicate with the intake manager, assigned attorney, or a firm administrator (if the attorney is unavailable) to ensure timely follow-up"

**Ambiguity:** The crisis SOP mentions three escalation paths (intake manager, attorney, firm administrator) but doesn't clarify when to use each.

**Questions to Resolve:**
1. Who is the Intake Manager? (Name and contact)
2. Who are the Firm Administrators? (Names and contacts)
3. What is the escalation priority order? (Intake Manager → Attorney → Firm Admin?)
4. Should the AI receptionist escalate to Intake Manager, or always to designated escalation contacts (Kyra/Janet)?

---

### 3.3 Callback Preference Collection

**Quote:** "Confirm with the client whether they prefer: A call back within 1–2 hours, A scheduled call for the following day"

**Ambiguity:** Should the AI receptionist collect callback preferences? This implies scheduling capability.

**Questions to Resolve:**
1. Should the AI offer callback timing preferences to upset clients?
2. If so, how should this preference be communicated to staff? (Message content? Specific field?)
3. Does the firm want appointment scheduling capability in the future?

---

### 3.4 Staff Availability Checking

**Source 1 (Universal Policy):**
- **Document:** SOP_-_Inbound_Caller_Intake_and_Routing_Workflow.pdf, Page 2, Section "3. Transferring Calls"
- **Quote:** "Before transferring: ... Confirm that the staff member is available to take the call. If staff is unavailable: Take a detailed message..."
- **Scope:** ALL transfers, regardless of caller type or staff role

**Source 2 (Attorney-Request Specific):**
- **Document:** Intake Call Routing Cheat Sheet.pdf, Page 1, Section "Attorneys"
- **Quote:** "If the client asks to speak to the attorney, transfer to the attorney if they are available. If they are not available call the case manager."
- **Scope:** ONLY when existing client explicitly requests to speak to their attorney

**Ambiguity:**
- The universal policy (Source 1) implies real-time availability checking before ANY transfer, with message-taking as fallback
- The attorney-specific policy (Source 2) implies a different fallback: try case manager instead of taking a message
- The AI system cannot perform real-time availability checks

**Questions to Resolve:**
1. For the **universal policy**: Should the AI attempt transfer and handle failure with message-taking, or should it always offer to leave a message first?
2. For **attorney requests specifically**: Should the AI attempt attorney transfer first, then case manager on failure, then message? Or is this a human-only workflow?
3. How does the firm currently check staff availability? (Teams status? Calendar? Phone system presence?)
4. Is there a presence/availability system that could be integrated with the AI?

---

### 3.5 Department Lead Escalation for Unreturned Callbacks

**Source:**
- **Document:** SOP_-_Inbound_Caller_Intake_and_Routing_Workflow.pdf, Page 2, Section "4. Handling Missed Calls and Follow-Ups"
- **Quote:** "If a caller is returning a call that was not returned: Attempt to connect them directly with the intended staff member. If the staff member is still unavailable, escalate the call to a department lead for resolution."
- **Scope:** ONLY applies to callers who are returning a call that was never returned to them (i.e., they're following up on an unreturned callback)

**Ambiguity:**
- This escalation path is specific to "unreturned callback" scenarios, not a general escalation policy
- The department leads are not identified
- It's unclear how the AI should detect "returning a call that was not returned" vs. a regular repeat caller

**Questions to Resolve:**
1. Does this policy apply to all caller types (existing clients, new clients, insurance adjusters, vendors, etc.) or only existing clients?
2. How should the AI detect this scenario? (Caller says "I'm returning a call" vs "No one called me back"?)
3. Who are the department leads for each department?
   - Intake department lead?
   - Case Management (Pre-Lit) department lead?
   - Litigation department lead?
4. Is this different from the existing escalation contacts (Kyra/Janet for Pre-Lit, Vickie Crabb/Emma Burgess for Lit)?
5. Should the AI use a general "customer success" line instead of trying to identify specific department leads?

---

### 3.6 Verification Level Preference

**Quote:** Not explicitly stated in SOP documents.

**Ambiguity:** The SOPs don't specify caller verification requirements. The system supports both "strict" (DOB verification) and "lenient" (name only) modes.

**Questions to Resolve:**
1. Should clients be verified by Date of Birth before discussing case details?
2. Should verification be required for all callers, or only certain scenarios?
3. Are there specific pieces of information that require verification before sharing?

---

## 4. MISSING DATA POINTS

*Required configuration/data not provided in SOP.*

### 4.1 Firm Configuration

Missing:
- [ ] **Receptionist/AI Name** - Needed for: Agent identity in greetings
- [ ] **Timezone** - Needed for: Hours calculation
- [ ] **Office hours (7-day schedule)** - Needed for: Open/closed routing logic
- [ ] **Intake hours** - Needed for: New case transfer availability (may differ from office hours)
- [ ] **Main phone number** - Needed for: Configuration
- [ ] **Office address** - Needed for: Caller questions
- [ ] **Website URL** - Needed for: Caller redirects
- [ ] **General email** - Needed for: Message taking

---

### 4.2 Staff Directory

Missing:
- [ ] **Complete staff roster** - Needed for: Directory lookup
  - Names, roles, phone numbers/extensions, emails
  - Must include: Attorneys, Case Managers, Paralegals, Record Clerk(s)
- [ ] **Staff IDs** - Needed for: Direct transfer mapping
- [ ] **Escalation contacts with direct numbers:**
  - [ ] Kyra (Pre-Lit escalation)
  - [ ] Janet (Pre-Lit escalation)
  - [ ] Vickie Crabb (Litigation escalation)
  - [ ] Emma Burgess (Litigation escalation)
  - [ ] April VanHoose (Settled 06-07 specialist)

---

### 4.3 Transfer Destinations

Missing:
- [ ] **Intake team number** - Needed for: New case transfers
- [ ] **Customer success / fallback number** - Needed for: General escalations
- [ ] **Insurance department number** - Needed for: Insurance adjuster transfers
- [ ] **Finance / Accounting number** - Needed for: Vendor/billing calls
- [ ] **Spanish line number** - Needed for: Spanish speaker routing
- [ ] **Administration team number** - Needed for: HR/Admin calls
- [ ] **Marketing department number** - Needed for: Marketing calls
- [ ] **Fax number** - Provided in SOP: **972-332-2361** ✓

---

### 4.4 CRM/Case Data Configuration

Missing:
- [ ] **SmartAdvocate instance URL** - Needed for: API integration
- [ ] **SmartAdvocate API credentials** - Needed for: Case lookup
- [ ] **Case status values mapping** - Partially provided:
  - Pre-Lit 00-11 (provided)
  - Settled 01-08 (provided)
  - Lit statuses (not detailed)
- [ ] **Case type field** - Needed for: Pre-Lit vs. Lit distinction
- [ ] **Attorney assignment field** - Needed for: Status-based attorney routing

---

### 4.5 Post-Call Notifications

Missing:
- [ ] **Email recipients per call category:**
  - [ ] New case / intake notifications
  - [ ] Existing client message notifications
  - [ ] Escalation notifications
  - [ ] Insurance adjuster call notifications
  - [ ] Upset client / crisis notifications

---

## 5. ADDITIONAL OBSERVATIONS

### 5.1 SmartAdvocate Integration Depth

The SOPs reference several SmartAdvocate features:
- "Medical providers tab in SA"
- "Lien tracking tab"
- "Notes tab"
- Case status codes (Pre-Lit 00-11, Settled 01-08)

**Recommendation:** The firm's SmartAdvocate integration needs to be deeper than the standard case lookup. Consider:
- Lienor verification API
- Status-based routing fields
- Medical provider verification

### 5.2 Crisis Management Workflow

The Client Crisis Management SOP is primarily about human staff handling crises, not AI routing. However, two elements are relevant:
1. **Upset caller detection** → Route to escalation contacts
2. **Callback preference collection** → May need message enhancement

### 5.3 Policy Alignment

McCraw's "DO NOT verify representation over the phone" policy aligns well with the Medical Provider agent's strict third-party handling. The fax redirect approach (972-332-2361) is a direct match.

### 5.4 Referenced but Missing SOP

The Inbound Caller Intake SOP references: "SOP: Screening a Potential New Client (PNC)" - This document was not provided but may contain additional requirements for new client intake.

---

## Recommended Next Steps

1. **Schedule client discovery call** to resolve the 6 clarity questions, especially around:
   - Verification requirements (strict vs. lenient)
   - Department lead contacts
   - Record clerk role definition

2. **Request firm data package:**
   - Complete staff directory with roles and contact info
   - Transfer destination phone numbers
   - SmartAdvocate API credentials
   - Office hours schedule

3. **Request additional SOPs:**
   - SOP: Screening a Potential New Client (PNC)
   - Any SOPs for specific departments (HR, Marketing, Finance)

4. **Plan technical implementation** for high-effort items:
   - Status-based routing logic (requires SmartAdvocate API enhancement)
   - Lienor verification lookup
   - Attorney vs. case manager field availability

5. **Configure blocklist** with provided entities:
   - Optum, Rawlings, Medcap, Gain/Gain Servicing
   - Elevate Financial, Intellivo, Movedocs
   - American Medical Response
   - Generic "hospital or ER room" patterns

6. **Create firm-specific prompt variants** addressing:
   - Pre-Lit vs. Litigation distinction
   - Two-tier escalation contacts
   - Status-based routing instructions
