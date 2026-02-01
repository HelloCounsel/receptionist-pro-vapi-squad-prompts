# SOP Analysis Capability Checklist

Reference document for analyzing client SOPs against VAPI squad agent capabilities.

---

## 1. Caller Type Matrix

| # | Agent | Caller Type | Key Capabilities | Tools |
|---|-------|-------------|------------------|-------|
| 1 | Greeter Classifier | Entry point | Routes all calls, collects name/purpose, classifies caller type | handoff_tool, staff_directory_lookup |
| 2 | Pre-Identified Client | Phone-matched clients | Pre-loaded case context, no verification needed for lookup | staff_directory_lookup, transfer_call |
| 3 | Existing Client | Unidentified clients | DOB verification, case lookup, case manager transfer | search_case_details, staff_directory_lookup, transfer_call |
| 4 | Insurance Adjuster | Insurance reps | Case lookup, status sharing, case manager contact | search_case_details, transfer_call |
| 5 | Medical Provider | Hospitals/clinics | Fax redirect only (no case info - third-party policy) | transfer_call |
| 6 | New Client | Accident victims | Intake transfer, case type Q&A, message taking | transfer_call |
| 7 | Vendor | Billing inquiries | Finance department transfer | transfer_call |
| 8 | Direct Staff Request | Named staff requests | Directory lookup, direct transfer by staff_id | staff_directory_lookup, transfer_call |
| 9 | Family Member | Third-party inquiries | Privacy protection, customer success routing | transfer_call |
| 10 | Spanish Speaker | Spanish language | Quick transfer to Spanish line | transfer_call |
| 11 | Referral Source | Referral inquiries | Customer success routing | transfer_call |
| 12 | Legal System | Courts/attorneys | Case lookup, legal team routing | search_case_details, transfer_call |
| 13 | Sales Solicitation | Sales calls | Polite decline, no transfers | (none) |
| 14 | Fallback Line | General/unclear | Safety net, customer success routing | transfer_call |

---

## 2. Information Sharing Policies

### What CAN Be Shared (Per Caller Type)

| Caller Type | Sharable Information |
|-------------|---------------------|
| Pre-Identified Client | Case manager name/phone/email, incident date, filing date |
| Existing Client (verified) | Case manager name/phone/email, incident date, filing date |
| Insurance Adjuster | Case manager name/phone/email, case status (translated), incident date, filing date, settlement date (not amounts) |
| Medical Provider | Fax number only |
| Legal System | Case manager name/phone/email, case status, dates |

### What CANNOT Be Shared

| Information Type | Blocked For |
|------------------|-------------|
| Internal case status codes | All clients (e.g., "pre lit demand draft", "discovery") |
| Settlement amounts | Everyone |
| Medical record contents | Everyone |
| Legal strategy / work product | Everyone |
| Case outcome predictions | Everyone |
| Client information to third parties | Medical providers, family members (without authorization) |

---

## 3. Transfer Destinations (caller_type Values)

| caller_type | Description | Used By |
|-------------|-------------|---------|
| `new_case` | Intake team for new potential clients | New Client agent |
| `existing_client` | Case manager transfer (requires staff_id) | Existing Client, Pre-ID Client |
| `customer_success` | Fallback for unclear routing, escalations | All agents |
| `insurance` | Insurance department | Insurance Adjuster agent |
| `vendor` | Finance department for billing | Vendor agent |
| `spanish` | Spanish-speaking line | Spanish Speaker agent |
| `escalation` | Priority queue for frustrated callers | Agents with escalation triggers |
| `legal` | Legal team for court/attorney matters | Legal System agent |

---

## 4. Tool Specifications

### search_case_details

**Type:** API Request (POST)

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| client_name | string | Yes | Full name of the client |
| client_dob | string | No | Date of birth (MM/DD/YYYY) |
| incident_date | string | No | Date of incident |
| firm_id | integer | Yes | Firm identifier |

**Returns:**
- `count`: Number of matching cases
- `cases[]`: Array of case objects with:
  - `client_full_name`, `client_phones`, `client_date_of_birth`
  - `case_number`, `case_manager`, `staff_id`
  - `case_incident_date`, `case_phase`, `case_type`, `case_status`
  - `case_manager_phone`, `case_manager_email`

**Limitations:**
- Requires exact or close name match
- Multiple results require disambiguation (DOB, incident date)
- Cannot search by phone number (backend phone lookup is separate)

---

### staff_directory_lookup

**Type:** RAG Query (Knowledge Base)

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| query | string | Yes | Staff name to search |

**Returns:**
- Staff name, role, phone, email, staff_id

**Limitations:**
- Fuzzy name matching
- Does not return availability status
- Cannot search by role (only by name)

---

### transfer_call

**Type:** Function

**Parameters:**
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| caller_type | string | Yes | Destination type (see table above) |
| firm_id | integer | Yes | Firm identifier |
| staff_id | string | No | Specific staff member ID (for direct transfers) |
| staff_name | string | No | Staff name (for logging) |
| caller_name | string | No | Caller's name |
| client_name | string | No | Client name (for insurance/medical) |
| reason | string | No | Reason for transfer |

**Limitations:**
- Requires phone system integration
- Transfer may fail (staff unavailable, after hours)
- Cannot queue callbacks
- Cannot schedule future calls

---

### handoff_tool (Greeter Only)

**Type:** Handoff

**Parameters:**
| Parameter | Type | Description |
|-----------|------|-------------|
| assistantName | string | Destination agent name (case-sensitive) |
| variableExtractionPlan | object | Context variables to pass |

**Context Variables Passed:**
- `caller_name`, `caller_type`, `purpose`
- `organization_name` (business callers)
- `client_name` (external callers with case needs)
- `target_staff_name` (direct staff requests)

---

## 5. Workflow Capabilities

### Hours-Based Logic

| Condition | Available Actions |
|-----------|-------------------|
| `is_open = true` | Transfers to staff, departments |
| `is_open = false` | Message taking only |
| `intake_is_open = true` | New case transfers available |
| `intake_is_open = false` | New case message taking |

### Verification Levels

| Level | Description | Used For |
|-------|-------------|----------|
| Strict | Requires DOB verification before sharing any case info | Existing Client agent |
| Lenient | Name matching sufficient | Pre-identified, business callers |
| None | No verification | New clients, general inquiries |

### Escalation Triggers

| Trigger | Detection Signals | Action |
|---------|-------------------|--------|
| Frustrated Caller | "I've been calling", "no one calls back", repeated attempts | Transfer to customer_success |
| Prior Contact | "I left a message", "haven't heard back" | Proactive customer_success offer |
| Communication Breakdown | Settlement/payment delays, extended wait times | Urgent escalation |

### Message Taking

| Scenario | Collected Info |
|----------|----------------|
| After hours | Callback number, message content |
| Transfer failure | Callback number, message content |
| Business caller | Phone AND email, message content |

---

## 6. Data Requirements Checklist

### Firm Configuration (Required)

| Data Point | Variable/Field | Purpose |
|------------|----------------|---------|
| Firm name | `{{firm_name}}` | Greetings, references |
| Receptionist name | `{{agent_name}}` | Agent identity |
| Timezone | `firms.timezone` | Hours calculations |
| Working hours | `firms.working_hours` | 7-day schedule, controls `is_open` |
| Intake hours | `firms.intake_working_hours` | 7-day schedule, controls `intake_is_open` |
| Main phone | `profile.contact.phone` | Caller reference |
| Address | `profile.locations` | Caller reference |
| Email | `profile.contact.email` | Caller reference |
| Website | `profile.contact.website` | Caller reference |

### Staff Directory (Required)

| Data Point | Field | Purpose |
|------------|-------|---------|
| Staff names | `staff_directory.name` | Lookup, transfers |
| Staff roles | `staff_directory.role` | Context (case_manager, paralegal, etc.) |
| Staff phones | `staff_directory.phone` | Direct dial, contact info |
| Staff emails | `staff_directory.email` | Contact info |
| Staff IDs | `staff_directory.staff_id` | Transfer routing |

**Supported Roles:** case_manager, paralegal, intake_specialist, receptionist, admin, lawyer

### Transfer Destinations (Required)

| Destination | Configuration | Purpose |
|-------------|---------------|---------|
| New case line | Phone number for `caller_type="new_case"` | Intake transfers |
| Customer success | Phone number for `caller_type="customer_success"` | Fallback, escalations |
| Insurance department | Phone number for `caller_type="insurance"` | Insurance adjuster routing |
| Finance department | Phone number for `caller_type="vendor"` | Vendor/billing routing |
| Spanish line | Phone number for `caller_type="spanish"` | Language support |
| Fax number | Direct number | Medical provider redirect |

### CRM/Case Data (Required for Case Lookup)

| Data Point | Field | Purpose |
|------------|-------|---------|
| Client full name | `case_details.client_full_name` | Matching, greeting |
| Client phone(s) | `case_details.client_phones` | Pre-identification |
| Client DOB | `case_details.client_date_of_birth` | Verification |
| Case number | `case_details.case_number` | Reference |
| Case manager | `case_details.case_manager` | Transfer routing |
| Staff ID | `case_details.staff_id` | Direct transfer |
| Incident date | `case_details.case_incident_date` | Disambiguation |
| Case phase | `case_details.case_phase` | Status context |
| Case type | `case_details.case_type` | Context |
| Case status | `case_details.case_status` | Internal tracking |

### Email Routing (Required for Post-Call)

| Category | Purpose |
|----------|---------|
| `new_case` | New client call summaries |
| `existing_client` | Existing client call summaries |
| `escalation` | Frustrated caller alerts |
| `insurance` | Insurance adjuster call summaries |

### CMS Integration (Optional)

| System | Required Config |
|--------|-----------------|
| SmartAdvocate | API username, API password |
| Filevine | Filevine email address |

---

## 7. Feature Limitations

### NOT Currently Supported

| Feature | Status | Workaround |
|---------|--------|------------|
| Appointment scheduling | Not available | Take message, manual callback |
| Payment processing | Not available | Transfer to finance |
| Document sending | Not available | Provide fax/email for requests |
| Outbound calls | Not available | Message taking with callback promise |
| SMS/text messaging | Not available | Voice only |
| Real-time availability | Not available | Transfer attempt, fall back to message |
| Multi-language (beyond Spanish) | Not available | English or Spanish only |
| Claim number collection | Not standard | Can be added to prompts |

### Known Constraints

| Constraint | Impact |
|------------|--------|
| No case info to medical providers | Fax redirect only (privacy policy) |
| Internal status codes hidden from clients | Must offer case manager callback for status |
| Single call handling | Cannot conference or add parties |
| Max call duration | 600 seconds (10 minutes) |
| Silence timeout | 30 seconds before prompt |

---

## 8. Routing Decision Tree

```
Incoming Call
    │
    ├── Phone matches existing client?
    │   └── YES → Pre-Identified Client (standalone)
    │   └── NO → Greeter Classifier
    │
Greeter Classification
    │
    ├── Speaks Spanish? → Spanish Speaker
    ├── Asks for specific staff by NAME? → Direct Staff Request
    ├── Says "I'm a client" / "my case"? → Existing Client
    ├── Says "I had an accident" / "need a lawyer"? → New Client
    ├── From insurance company? → Insurance Adjuster
    ├── From hospital/clinic? → Medical Provider
    ├── Billing/invoice question? → Vendor
    ├── "My mom's case" / family relationship? → Family Member
    ├── Court reporter / defense attorney? → Legal System
    ├── Referred someone / referral fee? → Referral Source
    ├── Selling services? → Sales Solicitation
    └── Unclear after 2 questions? → Fallback Line
```

---

## 9. Quick Reference: What Each Agent Handles

| Agent | Primary Question | Required Context |
|-------|------------------|------------------|
| Pre-ID Client | Case manager transfer, case info | Pre-loaded case_details |
| Existing Client | "What's my case status?" | Name, DOB, case lookup |
| Insurance Adjuster | "Status on claim for [client]?" | Client name, case lookup |
| Medical Provider | "Need case manager contact" | Redirect to fax |
| New Client | "I was in an accident" | Transfer to intake |
| Vendor | "Invoice payment status?" | Transfer to finance |
| Direct Staff | "Is [Name] available?" | Staff name |
| Family Member | "Calling about my [relation]'s case" | Caller name, client name |
| Spanish Speaker | (Spanish spoken) | Quick transfer |
| Referral Source | "I referred [Name] to you" | Referrer name |
| Legal System | Depositions, subpoenas, hearings | Case lookup |
| Sales | "I'd like to introduce our services" | Polite decline |
| Fallback | "Front desk please" / unclear | Customer success |
