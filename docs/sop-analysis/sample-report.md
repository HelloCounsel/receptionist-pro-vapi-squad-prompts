# Sample SOP Analysis Report

Example output demonstrating the 4-category format for SOP gap analysis.

---

# SOP Analysis Report: [Sample Firm Name]

**Generated:** 2026-01-29
**Analyzed By:** Claude Code SOP Analysis Skill
**SOP Document:** sample_firm_sop.pdf

---

## Summary

| Category | Count |
|----------|-------|
| Clear & Aligned | 8 |
| Needs Work | 3 |
| Needs Clarity | 2 |
| Missing Data | 5 |

---

## 1. CLEAR AND ALIGNED

*Workflows that match existing agent capabilities exactly. No changes needed.*

---

### 1.1 New Client Intake Routing

**SOP Requirement:** "All new potential clients calling about accidents should be transferred to the intake team during business hours. After hours, take a message with callback number."

**Agent:** New Client (06)
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Greeter routes callers who say "I had an accident" or "need a lawyer" to New Client agent
- New Client agent transfers to intake using `caller_type="new_case"` during hours
- After hours: Takes message with callback number

---

### 1.2 Insurance Adjuster Case Lookup

**SOP Requirement:** "Insurance adjusters calling about claimants should be able to get case status and case manager contact information."

**Agent:** Insurance Adjuster (04)
**Tools:** search_case_details, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- search_case_details looks up case by client name
- Agent provides case status (translated to plain English) and case manager phone/email
- Can transfer to insurance department or case manager

---

### 1.3 Spanish Language Support

**SOP Requirement:** "Spanish-speaking callers should be quickly transferred to the Spanish line."

**Agent:** Spanish Speaker (10)
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Greeter detects Spanish spoken or "Habla espanol?" request
- Quick transfer to Spanish line (no name collection required first)
- Uses `caller_type="spanish"`

---

### 1.4 Staff Direct Dial Requests

**SOP Requirement:** "Callers asking for a specific staff member by name should be transferred directly to that person."

**Agent:** Direct Staff Request (08)
**Tools:** staff_directory_lookup, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Greeter captures `target_staff_name` from caller
- Direct Staff agent looks up staff_id via directory
- Transfers directly using staff_id

---

### 1.5 Existing Client Case Manager Transfer

**SOP Requirement:** "Existing clients wanting to speak with their case manager should be verified and transferred."

**Agent:** Existing Client (03)
**Tools:** search_case_details, transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Collects DOB for verification (strict variant)
- Looks up case to find assigned case manager
- Transfers using staff_id for direct routing

---

### 1.6 Vendor Billing Inquiries

**SOP Requirement:** "Vendors calling about invoices or payments should be routed to the finance department."

**Agent:** Vendor (07)
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Greeter detects billing/invoice keywords
- Vendor agent transfers to finance using `caller_type="vendor"`

---

### 1.7 Frustrated Caller Escalation

**SOP Requirement:** "Callers who express frustration about unreturned calls should be escalated to customer success."

**Agent:** Existing Client, Insurance Adjuster
**Tools:** transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- Agents detect frustration signals ("I've been calling", "no one calls back")
- Escalate to customer_success with acknowledgment
- After hours: Mark as urgent, take message

---

### 1.8 After Hours Message Taking

**SOP Requirement:** "When the office is closed, take messages with callback numbers."

**Agent:** All agents with transfer_call
**Status:** FULLY SUPPORTED

**Current Implementation:**
- All agents check `is_open` and `intake_is_open` variables
- When false, switch to message-taking mode
- Collect callback number and message content

---

## 2. CLEAR BUT NEEDS WORK

*Requirements that are understood but need prompt/config changes to implement.*

---

### 2.1 Medical Provider Case Information Sharing

**SOP Says:** "Medical providers calling about patients should receive case manager name and direct phone number."

**Current Gap:** Medical Provider agent (05) follows a third-party privacy policy that ONLY provides fax number. No case lookup or case manager info is shared.

**Implementation Needed:**
1. Update Medical Provider prompt to allow case lookup
2. Add `search_case_details` tool to Medical Provider agent
3. Update information sharing policy to include case manager name/phone
4. Remove or modify fax-redirect-only behavior

**Effort:** Medium

**Considerations:**
- May require legal review of privacy implications
- Current policy is intentionally restrictive for compliance
- Need to confirm firm's comfort level with medical provider info sharing

---

### 2.2 Attorney Name Requests

**SOP Says:** "When clients ask 'Who is my attorney?', provide the assigned attorney name."

**Current Gap:** Existing Client agent only provides case manager information. Attorney assignment is in the case data but not exposed in the response flow.

**Implementation Needed:**
1. Confirm `attorney_name` field is available in case_details
2. Add handling for "Who is my attorney?" question in Existing Client prompt
3. Add attorney info to [What You CAN Share] section

**Effort:** Low

**Considerations:**
- Need to verify attorney data is reliably populated in CMS
- Should only share if attorney is assigned (not all cases have one)

---

### 2.3 Appointment Rescheduling

**SOP Says:** "Clients calling to reschedule appointments should be transferred to the case manager's assistant."

**Current Gap:** No appointment scheduling logic exists. Purpose "reschedule appointment" triggers fallback to customer_success.

**Implementation Needed:**
1. Add "appointment" detection to Existing Client agent
2. Define routing path for appointment requests (new staff role?)
3. Configure transfer destination for scheduling team
4. Add to greeter handoff logic if needed

**Effort:** Medium

**Considerations:**
- Requires new `caller_type` value (e.g., "scheduling")
- Need staff directory entry for scheduling contact
- Consider: Should this be case manager specific or centralized scheduling?

---

## 3. NEEDS CLARITY

*Ambiguous SOP sections requiring client clarification before implementation.*

---

### 3.1 "VIP Client" Handling

**Quote:** "VIP clients should receive priority treatment and be connected immediately."

**Ambiguity:** The SOP does not define:
1. What qualifies a client as "VIP"?
2. How is VIP status indicated in the system?
3. What does "priority treatment" mean operationally?
4. Who should VIP clients be connected to?

**Questions to Resolve:**
1. Is VIP status a flag in the case management system?
2. Should VIPs bypass verification steps?
3. Is there a specific person or queue for VIP calls?
4. Should VIP status be mentioned to the caller?

---

### 3.2 Referral Fee Inquiries

**Quote:** "Referral partners calling about their referrals should be helped promptly."

**Ambiguity:** The SOP does not specify:
1. What information can be shared about referred cases?
2. Can referral status or fee amounts be discussed?
3. Who handles referral partner relationships?

**Questions to Resolve:**
1. Can the receptionist confirm a referral was received?
2. Can case status of referred clients be shared?
3. Should referral partners be transferred to a specific person (business development)?
4. Is there a referral partner database to verify callers?

---

## 4. MISSING DATA POINTS

*Required configuration/data not provided in the SOP document.*

---

### 4.1 Firm Configuration

Missing:
- [ ] **Firm timezone** - Needed for: hours calculation, `is_open` logic
- [ ] **Office hours schedule** - Needed for: 7-day working hours configuration
- [ ] **Intake team hours** - Needed for: `intake_is_open` logic (may differ from office hours)

---

### 4.2 Staff Directory

Missing:
- [ ] **Complete staff list** - Needed for: staff_directory_lookup knowledge base
- [ ] **Staff phone numbers** - Needed for: direct transfers, contact info sharing
- [ ] **Staff email addresses** - Needed for: contact info sharing
- [ ] **Staff roles** - Needed for: routing context (case_manager, paralegal, etc.)

---

### 4.3 Transfer Destinations

Missing:
- [ ] **Intake team phone number** - Needed for: `caller_type="new_case"` transfers
- [ ] **Customer success phone number** - Needed for: fallback and escalation routing
- [ ] **Insurance department phone number** - Needed for: insurance adjuster routing
- [ ] **Spanish line phone number** - Needed for: Spanish speaker transfers
- [ ] **Fax number** - Needed for: medical provider redirect (if applicable)

---

### 4.4 Case Management Integration

Missing:
- [ ] **Case status values used** - Needed for: status translation to plain English
- [ ] **CMS system type** - Needed for: integration path (SmartAdvocate, Filevine, other)
- [ ] **API credentials (if SmartAdvocate)** - Needed for: case lookup integration
- [ ] **Filevine email (if Filevine)** - Needed for: case note routing

---

### 4.5 Email Routing

Missing:
- [ ] **New case summary recipients** - Needed for: post-call email routing
- [ ] **Existing client summary recipients** - Needed for: post-call email routing
- [ ] **Escalation alert recipients** - Needed for: frustrated caller notifications

---

## Recommended Next Steps

1. **Schedule onboarding call** to collect missing data points (Section 4)
2. **Send clarification questions** for ambiguous SOP sections (Section 3)
3. **Review "Needs Work" items** with client to confirm approach before implementation (Section 2)
4. **Document approval** for any changes to current privacy/compliance policies

---

## Appendix: Files to Modify

If all "Needs Work" items are approved:

| File | Changes |
|------|---------|
| `prompts/squad/strict/assistants/05_medical_provider.md` | Add case lookup, update info sharing |
| `prompts/squad/strict/assistants/03_existing_client.md` | Add attorney info handling |
| `docs/new_firm_onboarding.md` | Add appointment scheduling questions |
| Backend: transfer destinations | Add scheduling department |

---

*Report generated by /analyze-sop skill*
