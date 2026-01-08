# VAPI Variable Extraction Plans for Handoff Destinations

Copy these variable extraction configurations into VAPI dashboard for each handoff destination.

---

## 1. Pre-Identified Client (`handoff_to_pre_identified_client`)

**Variables Expected by Agent:**
- `case_details` (auto-populated from phone lookup)
- `caller_name` (auto from case_details.client_full_name)
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `purpose` | string | What the caller is calling about. Examples: "case update", "speak to case manager", "get status", "ask a question" |

> Note: `case_details` is pre-populated from phone lookup, not extracted during handoff.

---

## 2. Existing Client (`handoff_to_existing_client`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Full name of the caller (first and last name). This is REQUIRED - the destination agent uses this to search for the case. Extract exactly as stated by the caller. |
| `caller_type` | string | Set to "existing_client" |
| `purpose` | string | What the caller wants. Examples: "case update", "speak with case manager", "get case status", "question about case" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 3. Insurance Adjuster (`handoff_to_insurance_adjuster`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `organization_name`
- `client_name`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the insurance adjuster calling. REQUIRED - must collect before handoff. |
| `caller_type` | string | Set to "insurance" |
| `organization_name` | string | Name of the insurance company. Examples: "State Farm", "Progressive", "GEICO", "Allstate", "USAA", "Farmers", "Liberty Mutual", "Nationwide" |
| `client_name` | string | Name of the client/claimant/patient the adjuster is calling about. May be empty if not provided yet. |
| `purpose` | string | Why they're calling. Examples: "case status", "payment status", "case manager contact", "speak with case manager" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 4. Medical Provider (`handoff_to_medical_provider`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `organization_name`
- `client_name` (the patient)
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the person calling from the medical facility. REQUIRED - must collect before handoff. |
| `caller_type` | string | Set to "medical_provider" |
| `organization_name` | string | Name of the hospital, clinic, doctor's office, rehab center, or medical facility they're calling from. |
| `client_name` | string | Name of the patient whose case they're calling about. May be empty if not provided yet. |
| `purpose` | string | Why they're calling. Examples: "case manager contact", "case status", "speak with case manager", "treatment coordination" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 5. New Client (`handoff_to_new_client`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the person calling. REQUIRED - must collect before handoff. |
| `caller_type` | string | Set to "new_case" |
| `purpose` | string | Type of accident or what they're asking about. Examples: "car accident", "slip and fall", "truck accident", "motorcycle accident", "workplace injury", "medical malpractice", "I was in an accident", "need a lawyer" |
| `frustration_level` | number | Caller frustration level from 0-3. Often elevated due to recent accident. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 6. Vendor (`handoff_to_vendor`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `organization_name`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the person calling. REQUIRED - must collect before handoff. |
| `caller_type` | string | Set to "vendor" |
| `organization_name` | string | Name of the company or medical facility (for billing). |
| `purpose` | string | What they're calling about. Examples: "invoice", "billing", "payment status", "outstanding balance", "accounts payable" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 7. Direct Staff Request (`handoff_to_direct_staff_request`)

**Variables Expected by Agent:**
- `caller_name`
- `target_staff_name`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the person calling. REQUIRED - must collect before handoff. |
| `target_staff_name` | string | The specific staff member's name the caller asked for. REQUIRED - this is the primary reason for this handoff. Examples: "Sarah Johnson", "Mr. Thompson", "John". May be first name only, last name only, or full name. |
| `purpose` | string | Why they want to speak with this person. Optional - don't block handoff if not provided. |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 8. Family Member (`handoff_to_family_member`)

**Variables Expected by Agent:**
- `caller_name` (the family member calling)
- `caller_type`
- `client_name` (the actual client whose case it is)
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the family member who is calling. REQUIRED - must collect before handoff. This is NOT the client - this is the third party calling about someone else's case. |
| `caller_type` | string | Set to "family_member" |
| `client_name` | string | Name of the actual client whose case they're calling about. REQUIRED - must collect before handoff. This is the person whose case it is (e.g., "my mom", "my husband"). |
| `purpose` | string | What they're calling about. Examples: "case update", "speak with case manager", "get status on my mom's case" |
| `frustration_level` | number | Caller frustration level from 0-3. Often worried about their loved one. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 9. Spanish Speaker (`handoff_to_spanish_speaker`)

**Variables Expected by Agent:**
- `caller_name` (optional)
- `purpose` (optional)

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the caller if provided. OPTIONAL - don't delay transfer to collect. Quick transfer is priority for language barrier situations. |
| `purpose` | string | Why they're calling if understood. OPTIONAL - may not be clear due to language barrier. |
| `frustration_level` | number | Caller frustration level from 0-3. Often elevated due to language barrier. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 10. Referral Source (`handoff_to_referral_source`)

**Variables Expected by Agent:**
- `caller_name`
- `client_name` (the person they referred)
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the referrer who is calling. REQUIRED - must collect before handoff. This is the person who referred someone to the firm. |
| `client_name` | string | Name of the person they referred to the firm. PREFERRED - try to collect but don't block handoff. |
| `purpose` | string | What they're calling about. Examples: "referral fee", "referral status", "check on my referral", "I referred someone" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 11. Legal System (`handoff_to_legal_system`)

**Variables Expected by Agent:**
- `caller_name`
- `caller_type`
- `organization_name`
- `client_name`
- `purpose`

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the legal professional calling. REQUIRED - must collect before handoff. |
| `caller_type` | string | Type of legal system caller. Examples: "court_reporter", "defense_attorney", "court_clerk", "process_server", "opposing_counsel" |
| `organization_name` | string | Court, law firm, or company name they're calling from. PREFERRED - helps identify caller context. |
| `client_name` | string | Name of the client/case they're calling about. May be empty if not provided yet. |
| `purpose` | string | What they need. Examples: "deposition scheduling", "document service", "subpoena", "hearing notice", "case manager contact", "speak with attorney" |
| `frustration_level` | number | Caller frustration level from 0-3. 0=calm, 1=slightly rushed, 2=noticeably frustrated, 3=angry/upset |

---

## 12. Sales Solicitation (`handoff_to_sales_solicitation`)

**Variables Expected by Agent:**
- `caller_name` (optional)
- `organization_name` (optional)
- `purpose` (optional)

**Variable Extraction Plan:**

| Variable Name | Type | Description |
|---------------|------|-------------|
| `caller_name` | string | Name of the sales caller if provided. OPTIONAL - don't need to collect for sales calls. |
| `organization_name` | string | Company they represent. OPTIONAL. |
| `purpose` | string | What they're selling. OPTIONAL - examples: "software services", "marketing services", "legal tech platform" |
| `frustration_level` | number | Usually 0-1 for sales calls. 0=calm, 1=slightly rushed |

---

## Quick Copy Reference

### For VAPI Dashboard - JSON Format Examples

**Existing Client:**
```
caller_name: string - Full name of the caller (first and last name). REQUIRED for case lookup.
caller_type: string - Set to "existing_client"
purpose: string - What the caller wants (case update, speak with manager, etc.)
frustration_level: number - 0-3 scale (0=calm, 3=angry)
```

**Insurance Adjuster:**
```
caller_name: string - Name of the insurance adjuster. REQUIRED.
caller_type: string - Set to "insurance"
organization_name: string - Insurance company name
client_name: string - Client/claimant they're calling about
purpose: string - Why they're calling
frustration_level: number - 0-3 scale
```

**Direct Staff Request:**
```
caller_name: string - Name of the person calling. REQUIRED.
target_staff_name: string - Staff member name they asked for. REQUIRED.
purpose: string - Why they want this person (optional)
frustration_level: number - 0-3 scale
```

**Family Member:**
```
caller_name: string - Name of family member calling. REQUIRED.
caller_type: string - Set to "family_member"
client_name: string - Name of actual client (their relative). REQUIRED.
purpose: string - What they're calling about
frustration_level: number - 0-3 scale
```

---

## Notes

1. **REQUIRED variables** must be collected by the Greeter before calling the handoff tool. If not collected, the destination agent may fail to function correctly.

2. **PREFERRED variables** should be collected if possible but shouldn't block the handoff.

3. **OPTIONAL variables** are nice-to-have but the destination agent will function without them.

4. **frustration_level** should be extracted for all callers to help with message prioritization:
   - 0 = Calm, patient, neutral tone
   - 1 = Slightly rushed, minor impatience
   - 2 = Noticeably frustrated, short responses, sighing
   - 3 = Angry, upset, demanding, emotional distress

5. The **Spanish Speaker** destination is a quick-transfer scenario - don't delay collecting info.

6. For **Pre-Identified Client**, most variables come from the phone lookup, not from conversation extraction.
