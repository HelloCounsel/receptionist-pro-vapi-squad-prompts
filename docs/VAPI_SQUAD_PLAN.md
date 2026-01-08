# VAPI Squad Voice Receptionist - Project Plan

## Project Overview

**Purpose:** AI receptionist for personal injury law firms (starting with Bey & Associates in Atlanta, GA)

**Platform:** VAPI (Voice AI) using Squads feature for multi-agent orchestration

**Model:** GPT-4o-mini (chosen for latency/cost balance)

**Caller Demographics:** Mostly Atlanta, Georgia - emotional, practical, skeptical but hopeful. Tone should be like a trusted neighbor helping after a bad day.

---

## Prompt Architecture (v2)

### Design Philosophy

Write prompts like you're training a competent new hire, not programming a state machine.

**Do:**
- Describe the job and how to do it well
- Provide guidance and principles
- Let the AI figure out the words
- Use natural sections (Identity, Voice, Flow, Scenarios, Knowledge)

**Don't:**
- Write decision trees ("IF X THEN Y")
- Script exact phrases ("Say: 'What's your name?'")
- Create exhaustive lookup tables
- Treat the LLM like a dumb router

### Prompt Section Structure

Each node prompt follows this structure:

1. **Identity & Purpose** - Who you are, what you're here to do, what tools you have
2. **Voice & Persona** - How to sound (personality, warmth, brevity)
3. **What You're Working With** - Variables/context available
4. **Conversation Flow** - Natural phases of the interaction
5. **Response Guidelines** - Do's and don'ts
6. **Scenario Handling** - Common situations and how to approach them
7. **Knowledge Base** - Facts the agent needs (routing, caller types, etc.)
8. **Handoff Variables** - What to pass to the next agent

### Core Principle: Information Gathering First

The Greeter's job is NOT "classify and route." It's "gather information, then hand off."

Four pieces of information needed:
1. **caller_name** - Always ask if not given
2. **organization** - Business callers only
3. **caller_type** - INFER from context (never ask)
4. **purpose** - Usually stated, ask if unclear

Once you have what you need, routing flows naturally.

---

## Architecture: 4-Node Squad

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           VAPI SQUAD FLOW                                   │
│                                                                             │
│  ┌──────────────────┐                                                       │
│  │ 1. GREETER &     │ ← Entry point (isStartMember: true)                   │
│  │    CLASSIFIER    │                                                       │
│  └────────┬─────────┘                                                       │
│           │                                                                 │
│           ├───────────────────┬────────────────────┬───────────────────┐    │
│           ▼                   ▼                    ▼                   ▼    │
│  ┌────────────────┐  ┌────────────────┐  ┌────────────────┐  ┌────────────┐ │
│  │ 2. CASE        │  │ 3. TRANSFER    │  │ 4. MESSAGE     │  │ END CALL   │ │
│  │    HANDLER     │  │    EXECUTOR    │  │    TAKER       │  │            │ │
│  └────────┬───────┘  └───────┬────────┘  └───────┬────────┘  └────────────┘ │
│           │                  │                   │                          │
│           ▼                  ▼                   ▼                          │
│  ┌────────────────────────────────────────────────────────────────────┐     │
│  │ Can hand off to: Transfer Executor, Message Taker, or End Call     │     │
│  └────────────────────────────────────────────────────────────────────┘     │
└─────────────────────────────────────────────────────────────────────────────┘
```

**Nodes:**
1. **Greeter & Classifier** - Entry point. Gathers info, routes appropriately.
2. **Case Handler** - Searches for cases (if needed) AND answers case questions. Merged node.
3. **Transfer Executor** - Gets consent and executes transfers.
4. **Message Taker** - Collects messages when transfers aren't possible.

---

## Node Responsibilities

### Node 1: Greeter & Classifier
**Purpose:** Entry point - greets, collects identity, classifies, routes

**Tools:** None (handoff only)

**Inputs:**
- `is_open` (boolean) - Office hours
- `intake_is_open` (boolean) - Intake team available
- `firm_name` (string)
- `agent_name` (string)
- `case_details` (object, optional) - Pre-identified caller info from phone match

**Outputs (to pass on handoff):**
- `caller_name` (string, REQUIRED)
- `caller_type` (enum, REQUIRED)
- `purpose` (string, REQUIRED)
- `organization_name` (string, required for business callers)
- `client_name` (string, required for case lookups)
- `target_staff_name` (string, optional)
- `frustration_level` (0-3)
- `is_pre_identified` (boolean)

**Handoff Destinations:**
| Condition | Destination |
|-----------|-------------|
| New client | Transfer Executor (intake) |
| Spanish speaker | Transfer Executor (spanish) |
| Explicit staff request by name | Transfer Executor (staff_name) |
| Vendor + invoice | Transfer Executor (finance) |
| Medical + billing (NOT insurance) | Transfer Executor (finance) |
| Customer success triggers | Transfer Executor (customer_success) |
| Sales/solicitation | Message Taker |
| Pre-ID + wants message | Message Taker |
| Needs case info (pre-ID or not) | Case Handler |
| Family member (calling about someone else) | Transfer Executor (customer_success) |
| Case type question ("Do you handle X?") | Answer, then offer intake |

---

### Node 2: Case Handler
**Purpose:** Searches for cases (if needed) and provides case information. Merged from Lookup Specialist + Information Provider.

**Tools:** `search_case_details`

**Who Gets Routed Here:**
- Insurance adjusters needing case info
- Medical providers needing case info (not billing)
- Legal system callers (court reporters, defense attorneys)
- Existing clients (pre-identified or not) asking about their case

**Who Does NOT Get Routed Here:**
- Family members (→ Customer Success instead)
- Vendors with billing (→ Finance)
- Anyone not needing case information

**Inputs:**
- All from Greeter
- `client_name` (the name to search, if not pre-identified)
- `case_details` (if pre-identified)
- `firm_id`

**Two Paths:**

**Pre-Identified Path:**
```
Case already loaded → Answer their question → Wait for follow-up or goodbye
```

**Search Path:**
```
Search by name
    │
    ├── count = 1 → Answer their question
    │
    ├── count = 0 → Ask spelling → Re-search
    │       │
    │       └── Still 0 → Escalate to customer_success
    │
    └── count > 1 → Ask DOB → Re-search
            │
            ├── Still > 1 → Ask incident date → Re-search
            │       │
            │       └── Still > 1 → Escalate to customer_success
            │
            └── count = 1 → Answer their question
```

**Can Provide:**
- Case manager name, phone, email
- Attorney contact
- Case status
- Dates (incident, filed, settlement)

**Cannot Provide:**
- Settlement amounts
- Medical records
- Legal strategy

**Handoff Destinations:**
| Situation | Destination |
|-----------|-------------|
| Caller wants to speak with case manager | Transfer Executor |
| Caller wants to leave a message | Message Taker |
| No match found after spelling correction | Transfer Executor (customer_success) |
| Multiple matches after DOB + incident date | Transfer Executor (customer_success) |
| Bulk inquiry mentioned | Transfer Executor (customer_success) |
| Caller frustrated during disambiguation | Transfer Executor (customer_success) |
| Caller done | End call |

---

### Node 3: Transfer Executor
**Purpose:** Gets consent and executes transfers

**Tools:** `classify_and_route_call`

**Protocol:**
1. Say "[Transfer announcement]. [Consent question]?" as ONE statement
2. STOP talking
3. Wait for response
4. On affirmative → Call tool immediately (no verbal acknowledgment)
5. On negative → Offer to take message

**Transfer Destinations:**
- `new_case` → Intake team
- `existing` → Case manager
- `insurance` → Insurance department
- `vendor` → Finance department
- `spanish` → Spanish team
- `customer_success` → Customer success
- `other` → Specific staff by ID

---

### Node 4: Message Taker
**Purpose:** Collects messages with proper contact info

**Tools:** `take_message`

**Two Playbooks:**
1. **Business** (Insurance, Medical, Vendor, Legal): Require phone + email
2. **Individual** (Client, Family): Require phone, email only if phone declined

**Priority Levels:**
- URGENT: Multiple attempts, explicit urgency, severe frustration
- HIGH: Moderate frustration, follow-up mentioned
- STANDARD: Default

---

## Caller Type Classifications

```
NEW_CLIENT          - Needs legal representation
EXISTING_CLIENT     - Has active case with firm
FAMILY_MEMBER       - Calling about family member's case
INSURANCE_ADJUSTER  - Insurance company rep
MEDICAL_PROVIDER    - Hospital, clinic, rehab
VENDOR              - Service provider with invoice
LEGAL_SYSTEM        - Court reporter, defense attorney
LAW_OFFICE          - Another law firm
REFERRAL_SOURCE     - Referral partner (Hurt 911, etc.)
BUSINESS_SALES      - Selling services
SPANISH_SPEAKING    - Needs Spanish team
OTHER               - Cannot classify
```

---

## Variable Extraction Schema (VAPI)

All handoffs should pass these variables (VAPI extracts automatically):

```json
{
  "caller_name": "string",
  "caller_type": "string (enum)",
  "purpose": "string",
  "organization_name": "string | null",
  "client_name": "string | null",
  "target_staff_name": "string | null",
  "frustration_level": "number (0-3)",
  "is_pre_identified": "boolean",
  "phone_number": "string | null",
  "email": "string | null"
}
```

---

## Critical Routing Rules

### Rule 1: Insurance Callers NEVER Route to Finance
Even if insurance adjuster mentions "billing" or "payment", they route to Lookup Specialist, NOT Finance. The "billing" keywords only trigger Finance for vendors and medical providers.

### Rule 2: Explicit Staff Request = Skip Lookup
If caller says "Can I speak with Sarah Johnson?", route directly to Transfer Executor. Don't do a case lookup first.

### Rule 3: Pre-Identified Takes Priority
If `case_details` is available (phone matched), the caller is definitively an existing client. Don't re-classify.

### Rule 4: 2-Try Limit
For any piece of info (name, spelling, DOB), try max 2 times. After 2 failures:
- For name: Mark as `user_declined`, proceed with what you have
- For disambiguation: Escalate to customer_success

### Rule 5: Handoffs Are Silent
Never say "Let me transfer you to..." or similar. Handoffs happen invisibly - it's experienced as a single continuous agent.

### Rule 6: Case Type Questions = Answer First, Then Offer
If caller asks "Do you handle [X] cases?" (slip and fall, medical malpractice, etc.):
1. Answer the question: "Yes, we handle [X] cases."
2. Then ask: "Would you like me to connect you with our intake team?"
3. If yes → Transfer Executor (intake)
4. If no → End call naturally

Do NOT route directly to intake without answering their question first.

### Rule 7: Family Members = Customer Success
If caller identifies as family member calling about someone else's case (NOT pre-identified):
- Route to Transfer Executor (customer_success)
- Do NOT route to Lookup Specialist
- Customer success handles third-party inquiries for privacy/authorization reasons

---

## Prompt Structure (Recommended)

Each node prompt should follow this structure:

```
1. ROLE DEFINITION (2-3 sentences)
   - Who you are
   - What tools you have access to
   - What you CANNOT do

2. CONTEXT VARIABLES (list)
   - What data you receive
   - What you can reference without asking

3. CALL FLOW (state machine)
   - Step-by-step with clear transitions
   - Decision points marked
   - Max attempts for each step

4. UNIFIED ROUTING TABLE
   - One table: Condition → Required Info → Destination
   - No scattered routing logic

5. RESPONSE GUIDELINES
   - Tone
   - Length constraints
   - Formatting (phone, email, dates)

6. EDGE CASES (if/then format)
   - "Are you AI?" → Response
   - Frustrated caller → Response
   - Wrong person (pre-ID) → Response

7. BLOCKING RULES (DO NOT)
   - Clear prohibitions
   - What must happen BEFORE handoff
```

---

## Known Issues in Current Prompts

### Issue 1: Mixed Abstraction Levels
Current prompts mix philosophy ("Two Missions"), technique ("How to Listen"), and action ("Your Opening") in non-sequential order.

**Fix:** Restructure as sequential state machine.

### Issue 2: Fragmented Routing
Routing logic appears in 3 places:
- Main prompt ("Where to Hand Off")
- Handoffs file (conditions)
- VAPI tool descriptions

**Fix:** Single source of truth in prompt. Tool descriptions are labels only.

### Issue 3: Brevity Creates Ambiguity
Instructions like "hand off immediately" don't specify what to say or what variables to ensure.

**Fix:** Be explicit about:
- Variables that MUST be captured
- Exact phrasing for transitions
- What "immediately" means in context

### Issue 4: Redundant Content
Persona, voice formatting, and silence rules appear in every assistant file.

**Fix:** Use Jinja2 includes for shared content (already have `global/` directory).

### Issue 5: File Organization Decision
Keep `handoffs/*.j2` files SEPARATE from `assistants/*.j2` files.
- Assistants: Conversation logic, tone, flow
- Handoffs: Routing rules, conditions, variables to pass
- This separation allows updating routing without touching conversation logic.

### Issue 6: Missing State Tracking
Prompts don't tell LLM to check received variables before asking questions.

**Fix:** Add explicit "check variables before asking" instructions.

---

## Prompt Restructure Plan (COMPLETED)

All prompts have been restructured using narrative guidance style instead of decision trees.

### Completed Work:

**Node 1: Greeter & Classifier** ✓
- Rewritten with narrative guidance structure
- Focus on information gathering first, routing second
- Silent handoffs emphasized
- 4 required pieces: name, organization (business only), caller type (inferred), purpose

**Node 2: Case Handler** ✓ (NEW - Merged)
- Combined Lookup Specialist + Information Provider into single node
- Handles both case search (if needed) and answering questions
- Two paths: Pre-identified (skip search) vs Search (disambiguate if needed)
- Clear escalation rules for no-match and multi-match scenarios

**Node 3: Transfer Executor** ✓
- Consent + execute flow preserved
- Hours awareness documented
- Tool usage instructions included

**Node 4: Message Taker** ✓
- Business vs individual playbooks
- Priority levels documented
- Phone confirmation emphasized

---

## File Structure

```
src/utils/llm/templates/squad_prompts/
├── global/
│   ├── persona.j2              # Shared persona (Emma, 2 years, warm)
│   ├── voice_formatting.j2     # Phone, email, date formatting
│   ├── silence_rules.j2        # Stop talking protocol
│   ├── brevity_guidelines.j2   # Response length
│   └── knowledge_base.j2       # Firm info, FAQs
│
├── assistants/                 # Main prompt for each node (4-node architecture)
│   ├── 1_greeter_classifier.j2 # Node 1: Entry point, gathers info, routes
│   ├── 2_case_handler.j2       # Node 2: Searches cases + provides info (merged)
│   ├── 3_transfer_executor.j2  # Node 3: Gets consent, executes transfers
│   └── 4_message_taker.j2      # Node 4: Collects messages
│
├── handoffs/                   # Routing rules for each node
│   ├── 1_greeter_handoffs.j2
│   ├── 2_case_handler_handoffs.j2
│   ├── 3_transfer_executor_handoffs.j2
│   └── 4_message_taker_handoffs.j2
│
├── extractors/                 # Variable extraction schemas (for reference)
│   ├── caller_identity.j2
│   ├── caller_classification.j2
│   └── ...
│
└── nodes/                      # (Deprecated - older structure)
```

---

## VAPI Configuration Reference

### Squad JSON Structure (Key Points)

```json
{
  "name": "Receptionist Pro - V1",
  "members": [
    {
      "assistantId": "...",
      "assistantName": "greeter & classifier",
      "isStartMember": true,  // Entry point
      "assistantOverrides": {
        "tools:append": [{
          "type": "handoff",
          "destinations": [
            {
              "assistantId": "...",
              "assistantName": "Information Provider",
              "description": "Condition for handoff...",
              "contextEngineeringPlan": {
                "type": "lastNMessages",
                "maxMessages": 3  // Only last 3 messages passed
              },
              "variableExtractionPlan": {
                "schema": {
                  "properties": {
                    "purpose": { "type": "string" },
                    "caller_name": { "type": "string" },
                    // ...
                  }
                }
              }
            }
          ]
        }]
      }
    }
  ]
}
```

### Important VAPI Behaviors

1. **firstMessage vs Prompt:** `firstMessage` in assistant config is spoken first. Prompt's "greeting" section should match or complement.

2. **Context Engineering:** `lastNMessages: 3` means receiving assistant only sees last 3 messages. Plan context accordingly.

3. **Variable Extraction:** VAPI extracts variables automatically based on schema. Prompt doesn't need to explicitly instruct extraction, but SHOULD reference variables so LLM knows what's available.

4. **Handoffs Are Invisible:** No audio artifacts. Caller experiences one continuous agent.

5. **firstMessageMode:** Set to `assistant-speaks-first-with-model-generated-message` for some nodes - they generate their own first message.

---

## Testing Scenarios

### Happy Paths
1. New client → Greeter → Transfer Executor (intake)
2. Insurance adjuster with full info → Greeter → Lookup → Info Provider
3. Pre-identified client asking about case → Greeter → Info Provider
4. Pre-identified client wants transfer → Greeter → Transfer Executor
5. Case type question → Answer → Offer intake → Transfer or end

### Edge Cases
1. Name not provided after 2 asks → Proceed with `user_declined`
2. Multiple matches after DOB + incident date → Escalate to customer_success
3. Pre-identified but wrong person → Reset, ask for name
4. Caller asks "Are you AI?" → Disclose, continue
5. Transfer fails → Message Taker
6. Office closed → Message Taker
7. Family member calling about relative's case → Customer Success (not Lookup)

### Regression Tests
1. Insurance caller mentions "billing" → Should NOT go to Finance
2. Explicit staff request → Should NOT do lookup first
3. After providing info → Should NOT proactively offer more
4. "Do you handle car accidents?" → Should answer FIRST, then offer intake
5. "I'm calling about my daughter's case" → Should go to Customer Success, NOT Lookup

---

## Implementation Checklist

- [x] Restructure Greeter/Classifier prompt
- [x] Merge Lookup Specialist + Information Provider into Case Handler
- [x] Restructure Transfer Executor prompt
- [x] Restructure Message Taker prompt
- [x] Update handoff files for all 4 nodes
- [x] Rename/renumber files for 4-node architecture
- [x] Remove deprecated files (old lookup, info provider)
- [ ] Consolidate global includes (remove duplication)
- [ ] Update VAPI squad configuration to match 4-node architecture
- [ ] Test all happy paths
- [ ] Test all edge cases
- [ ] Validate variable extraction works correctly

---

## Reference Links

- [VAPI Squads Documentation](https://docs.vapi.ai/squads)
- [VAPI Handoff Tools](https://docs.vapi.ai/squads#handoff-tools)
- [VAPI Variable Extraction](https://docs.vapi.ai/squads#variable-extraction)

---

## Change Log

| Date | Change | Author |
|------|--------|--------|
| 2024-12-16 | Initial plan created | Claude |
| 2024-12-16 | Restructured to 4-node architecture (merged Lookup + Info Provider → Case Handler) | Claude |
| 2024-12-16 | Rewrote all prompts with narrative guidance style | Claude |
| 2024-12-16 | Cleaned up file structure, removed deprecated files | Claude |

---

## Notes for Future Sessions

If continuing this work in a new Claude session, reference this file for:
1. Overall architecture understanding
2. Known issues and fixes needed
3. Recommended prompt structure
4. VAPI-specific behaviors to account for
5. Testing scenarios to validate

Key decisions made:
- Insurance callers NEVER route to Finance
- 2-try limit on all info gathering
- Explicit staff requests skip lookup
- Handoffs are silent (no verbal transition)
- Variables should be checked before asking (avoid re-asking)
