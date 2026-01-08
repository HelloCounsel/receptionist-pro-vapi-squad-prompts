# Squad: Caller-Type-Based Multi-Agent System

## Overview

This is a 10-agent VAPI squad system for handling incoming calls at Bey & Associates personal injury law firm.

**Architecture:** Caller-type-based specialization where each specialized agent handles the complete flow (lookup → info → transfer → message).

**Note:** Pre-identified callers (phone number matched to existing client) are handled by the standalone assistant in `prompts/standalone/`, not this squad. The backend routes calls to either this squad or the standalone assistant based on phone lookup.

## Agent List

| # | Agent Name | Purpose |
|---|------------|---------|
| 1 | Greeter Classifier | Entry point - collects name, purpose, routes |
| 2 | Existing Client | Non-pre-identified existing clients |
| 3 | Insurance Adjuster | Insurance company representatives |
| 4 | Medical Provider | Hospitals, clinics (NOT billing) |
| 5 | New Client | People who need a lawyer |
| 6 | Vendor | Invoice/billing callers |
| 7 | Direct Staff Request | Specific staff requests |
| 8 | Family Member | Third-party callers |
| 9 | Spanish Speaker | Spanish transfer wrapper |
| 10 | Legal System | Court reporters, defense attorneys |

## Setup Instructions

### Step 1: Create Assistants in VAPI Dashboard

For each assistant in `assistants/`:

1. Go to VAPI Dashboard → Assistants → Create New
2. Set **Name** exactly as specified (case-sensitive for handoff routing)
3. Copy the system prompt from the `.md` file
4. Configure model: `gpt-4o` or `chatgpt-4o-latest`
5. Configure voice: Cartesia `sonic-3` with voiceId `f786b574-daa5-4673-aa0c-cbe3e8534c02`
6. Configure transcriber: Deepgram `flux-general-en`

### Step 2: Configure Assistant Settings

For **Greeter Classifier** (entry point):
```
firstMessage: "Bey and Associates, this is {{agent_name}}. How can I help you?"
firstMessageMode: "assistant-speaks-first"
```

For **ALL OTHER assistants** (silent handoff recipients):
```
firstMessage: ""
firstMessageMode: "assistant-speaks-first-with-model-generated-message"
```

### Step 3: Add Tools

**For Greeter Classifier:**
- Add handoff tool from `handoff_tools/greeter_handoff_tool.json`

**For agents that do case lookup** (Existing, Insurance, Medical, Legal):
- Add `search_case_details` API tool
- Add `classify_and_route_call` tool
- Add `take_message` tool

**For transfer-only agents** (New Client, Vendor, Staff Request, Family, Spanish):
- Add `classify_and_route_call` tool
- Add `take_message` tool

### Step 4: Create Squad

1. Go to VAPI Dashboard → Squads → Create New
2. Add the 10 active assistants as members
3. Set **Greeter Classifier** as the start member (`isStartMember: true`)
4. Apply member overrides from `vapi_config/assistant_settings.json`

### Step 5: Configure Variables

Pass these variables to the squad:
- `firm_name`: "Bey and Associates"
- `firm_id`: 1
- `agent_name`: "Emma"
- `is_open`: boolean (office hours status)
- `intake_is_open`: boolean (intake team hours status)
- `case_details`: object (for pre-identified callers, null otherwise)

### Step 6: Add staff_directory_lookup Tool

The staff directory is now accessed via RAG-based lookup tool:
- Add `staff_directory_lookup` query tool to all agents (see `agent_tools.json` for tool_assignments)
- This tool uses knowledge base lookup to find staff members by name

## File Structure

```
prompts/squad/
├── README.md                    # This file
├── assistants/                  # System prompts (copy to VAPI)
│   ├── 01_greeter_classifier.md
│   ├── 03_existing_client.md
│   └── ... (10 active + 3 WIP)
├── handoff_tools/               # Tool configurations
│   ├── greeter_handoff_tool.json
│   ├── greeter_handoff_destinations.md
│   ├── agent_tools.json
│   └── staff_directory_lookup_description.txt
└── vapi_config/                 # VAPI-specific configs
    ├── squad_structure.json
    └── assistant_settings.json
```

### Assistant Files

**Active (deployed):**
- 01_greeter_classifier.md
- 03_existing_client.md
- 04_insurance_adjuster.md
- 05_medical_provider.md
- 06_new_client.md
- 07_vendor.md
- 08_direct_staff_request.md
- 09_family_member.md
- 10_spanish_speaker.md
- 12_legal_system.md

**WIP (not deployed):**
- 11_referral_source.md
- 13_sales_solicitation.md

## Critical Configuration Notes

### Silent Handoffs
All handoff tools must have `"messages": []` (empty array) for silent transitions.

### Assistant Names Must Match
Handoff tool destinations reference assistants by `assistantName`. Names must match exactly.

### Context Transfer
- Greeter → Pre-ID: `"type": "all"` (full context)
- Greeter → Others: `"type": "lastNMessages", "value": 5`
- Greeter → Sales: `"type": "lastNMessages", "value": 3`

### Variable Extraction
Each handoff includes `variableExtractionPlan` to pass context:
- `caller_name`, `caller_type`, `purpose`
- `organization_name` (business callers)
- `client_name` (external callers with case needs)
- `frustration_level` (0-3 scale)

## Testing Checklist

- [ ] New client → intake transfer
- [ ] Insurance adjuster → case lookup → info provision
- [ ] Existing client → case manager transfer
- [ ] Pre-identified client → direct service
- [ ] Staff request → direct transfer
- [ ] Medical + Billing → Vendor routing
- [ ] Misclassification → customer success escalation
- [ ] After-hours → message taking
- [ ] Frustrated caller → priority escalation
