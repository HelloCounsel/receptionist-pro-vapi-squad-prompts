# Demo Assistants

This directory contains **standalone VAPI assistants** used for client demos. These are separate from the production squad architecture.

---

## Key Differences: Demo vs Production

| Aspect | Demo (Standalone) | Production (Squad) |
|--------|-------------------|-------------------|
| Agents | 1 (all-in-one) | 13+ specialized |
| Handoffs | None (self-contained) | Agent-to-agent |
| Routing | Internal flow logic | Greeter + handoff tools |
| Flexibility | Simpler, demo-focused | Full production capability |
| Firm | Demo firm (Duffy and Duffy) | Client firm (Bey & Associates) |

---

## Available Demo Assistants

### Standard Demo Receptionist - only email

**Location:** `standard_demo_receptionist/`

**Purpose:** All-in-one receptionist for client demos that handles:
- Existing clients → case lookup, case manager transfer
- New clients → intake team transfer
- Medical providers → case status + case manager email
- Insurance adjusters → case status + case manager email
- Escalation requests → customer success transfer

**Key Features:**
- Single-agent architecture (no handoffs)
- Email-only case manager contact (no phone numbers to providers)
- Demo firm branding (Duffy and Duffy Law Firm)

**Files:**
- `system_prompt.md` - Full prompt in repository format
- `vapi_config.json` - VAPI configuration settings
- `architecture.md` - Prompt architecture documentation

---

### Standard Demo Receptionist - email and phone

**Location:** `standard_demo_receptionist_full_contact/`

**Purpose:** Same as "only email" variant, but provides BOTH email AND phone when external callers ask for case manager contact info.

**Key Features:**
- Single-agent architecture (no handoffs)
- Full contact info (email + phone) for case managers
- Phone numbers read as words ("eight seven zero, eight seven seven...")
- Demo firm branding (Duffy and Duffy Law Firm)

**Files:**
- `system_prompt.md` - Full prompt in repository format
- `vapi_config.json` - VAPI configuration settings
- `architecture.md` - Prompt architecture documentation

**Key Differences from "only email":**
| Aspect | only email | email and phone |
|--------|------------|-----------------|
| Medical provider contact request | Email only (always) | Phone OR email (based on what's asked) |
| Insurance adjuster contact request | Email only (always) | Phone OR email (based on what's asked) |
| Staff directory fields | email | email, phone |
| Phone availability | Not available | Available when explicitly requested |
| Phone format | N/A | Words ("eight seven zero...") |

---

## Creating New Demo Assistants

1. Create a new directory under `prompts/demo/`
2. Add the three standard files:
   - `system_prompt.md` - The prompt itself
   - `vapi_config.json` - VAPI settings (model, voice, tools)
   - `architecture.md` - Architecture documentation
3. Update this README with the new assistant
4. Update `docs/CHANGELOG.md` with the addition

---

## Usage Notes

- Demo assistants are for **demonstration purposes only**
- They may use simplified flows compared to production
- Firm-specific data (staff directory, variables) is hardcoded for the demo firm
- Do not deploy demo assistants to production without proper configuration
