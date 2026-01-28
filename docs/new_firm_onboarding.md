# New Firm Onboarding Questionnaire

Questions grounded in actual agent configuration levers.

---

## 1. Firm Identity (Variables)

These directly populate `{{firm_name}}`, `{{agent_name}}`, and prompt templates.

| Question | Maps To |
|----------|---------|
| What is the firm name? | `{{firm_name}}` - used in greetings and references |
| What should the AI receptionist be named? | `{{agent_name}}` - e.g., "Emma", "Sarah" |
| What is the firm's main phone number? | `profile.contact.phone` |
| What is the firm's address? | `profile.locations` |
| What is the firm's email? | `profile.contact.email` |
| What is the firm's website? | `profile.contact.website` |
| What timezone is the firm in? | `firms.timezone` - e.g., "America/New_York" |

---

## 2. Business Hours (Variables)

These control whether transfers are attempted or messages are taken.

| Question | Maps To |
|----------|---------|
| What are your office hours? | `{{is_open}}` / `firms.working_hours` |
| What are your intake team hours? | `{{intake_is_open}}` / `firms.intake_working_hours` |

---

## 3. Practice Areas & Fees

Greeter and New Client agents reference these when answering caller questions.

| Question | Maps To |
|----------|---------|
| What case types do you handle? | `profile.services` - agent answers "Do you handle X?" |
| What's your fee structure? | `profile.fees` - agent answers "How much do you charge?" |

---

## 4. Staff Directory & Routing

The `transfer_call` tool routes to `caller_type` departments or specific `staff_id`.

| Question | Maps To |
|----------|---------|
| Who should new client/intake calls go to? | `caller_type="new_case"` destination |
| Who should existing client calls go to when case manager unavailable? | `caller_type="customer_success"` destination |
| Who should insurance adjuster calls go to? | `caller_type="insurance"` destination |
| Who is the fallback/front desk for unclear calls? | `caller_type="customer_success"` fallback |
| Staff list: names, roles, phone numbers, emails? | `staff_directory` table - powers `staff_directory_lookup` |

**Staff roles we track:** case_manager, paralegal, intake_specialist, receptionist, admin, lawyer

---

## 5. Existing Client Handling

| Question | Why It Matters |
|----------|----------------|
| Should we verify caller identity before sharing case info? | We have strict vs. lenient variants - strict requires DOB |
| What case statuses do you use? | Agent says "Your case status is [X]" - must match your CMS |
| What can the agent share? | Default: case manager contact, case status, incident date |
| What should the agent NOT discuss? | Default blocked: settlement amounts, medical records, legal strategy |

---

## 6. Insurance Adjuster Handling

| Question | Why It Matters |
|----------|----------------|
| What case info can we share with adjusters? | Default: case manager contact, case status, dates |
| What should we NOT share? | Default blocked: settlement amounts, medical records |
| Should we collect claim number before transferring? | Not currently collected - can add |

---

## 7. Medical Provider Handling

| Question | Why It Matters |
|----------|----------------|
| What is your fax number for case inquiries? | Agent provides this to all medical provider callers |
| Is fax-only policy correct for your firm? | Default: no case lookup, redirect all to fax |

---

## 8. New Client Handling

| Question | Why It Matters |
|----------|----------------|
| What should we collect if intake is unavailable? | Default: callback number + brief description |

---

## 9. Post-Call Workflow

After every call ends, the system processes the conversation and sends notifications.

### Email Notifications

| Question | Maps To |
|----------|---------|
| Who should receive new case/intake call summaries? | `category_receivers` where category = "new_case" |
| Who should receive existing client call summaries? | `category_receivers` where category = "existing_client" |
| Who should receive escalation/frustrated caller alerts? | `category_receivers` where category = "escalation" |
| Who should receive insurance adjuster call summaries? | `category_receivers` where category = "insurance" |
| Should all calls go to a general inbox too? | Additional recipient in all categories |

### Case Management Integration

| Question | Maps To |
|----------|---------|
| What case management system do you use? | Determines integration path |
| **If SmartAdvocate:** API credentials? | `SMARTADVOCATE_API_USERNAME`, `SMARTADVOCATE_API_PASSWORD` |
| **If SmartAdvocate:** Should we create case notes from calls? | Enables `SmartAdvocateNoteService` |
| **If Filevine:** What email should case summaries go to? | `filevine_email` configuration |
| Do you want call recordings linked in case notes? | Recording URL included in notes |

### Call Data

| Question | Why It Matters |
|----------|----------------|
| How long should we retain call recordings? | Compliance/storage policy |
| Should transcripts be stored? | Default: yes, stored in `calls` table |
| Do you need call duration tracking? | Default: yes, calculated from timestamps |

---

## 10. Technical Integration

### Voice Platform (VAPI)

| Question | Maps To |
|----------|---------|
| Which phone number(s) will route to the agent? | `voice_platforms.platform_phone_id` |
| Should we set up a backup/overflow number? | Additional phone mapping |

### Case Lookup API

| Question | Maps To |
|----------|---------|
| Can we integrate with your CMS for case lookup? | Enables `search_case_details` tool |
| Do you want pre-identified caller routing? | Phone lookup → skip greeter classification |

### Transfer API

| Question | Maps To |
|----------|---------|
| What phone system do you use? | Determines `transfer_call` integration |
| Can we get direct dial numbers for staff? | Enables direct transfers vs. department routing |

---

## Quick Reference Checklist

```
Firm Variables:
- [ ] Firm name
- [ ] Receptionist name
- [ ] Timezone
- [ ] Phone, address, email, website
- [ ] Office hours
- [ ] Intake hours

Staff Directory:
- [ ] Names, roles, phone, email for all staff

Routing Destinations:
- [ ] New case / intake team
- [ ] Customer success / existing client fallback
- [ ] Insurance department
- [ ] Fallback / front desk

Caller-Type Specific:
- [ ] Case statuses used in your CMS
- [ ] Fax number for medical providers
- [ ] Fee structure
- [ ] Case types handled
- [ ] Verification policy (strict vs. lenient)

Post-Call Notifications:
- [ ] Email recipients per call category
- [ ] CMS integration (SmartAdvocate/Filevine credentials)
- [ ] Case note creation preferences

Technical:
- [ ] VAPI phone number(s)
- [ ] Case management system + API access
- [ ] Phone system for transfers
```

---

## Phased Rollout

**Phase 1 - Minimum Viable Pilot:**
- Firm identity + hours + timezone
- Staff directory (names/phones/emails)
- Routing destinations
- Fax number
- Post-call email recipients

**Phase 2 - Enhanced:**
- Case lookup integration (pre-identify callers)
- CMS integration (SmartAdvocate/Filevine notes)
- Custom case status terminology

**Phase 3 - Optimized:**
- Pre-identified caller routing
- Direct staff transfers
- Custom verification policies
