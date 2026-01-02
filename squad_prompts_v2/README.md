# AI Receptionist Squad - VAPI Dashboard Setup Guide

## Overview

This folder contains dashboard-ready prompts for a 4-agent squad handling incoming calls for a personal injury law firm.

## VAPI Compliance

All prompts follow VAPI's recommended structure:

1. **System Context block** - Multi-agent awareness and seamless handoff behavior
2. **Bracket-based sections** - `[Identity]`, `[Style]`, `[Response Guidelines]`, `[Task]`, `[Error Handling]`
3. **Silent handoff rules** - "Never say transferring", "trigger tool silently"
4. **Context block for non-entry agents** - "proceed without greetings"
5. **Step-by-step tasks** with `- Wait for the customer's response.` after each step
6. **Explicit handoff triggers** - `trigger the handoff tool with [Agent-Name] Assistant`

## Squad Architecture

```
Caller → [1. Greeter/Classifier]
              ↓
         Classifies caller type & purpose
              ↓
    ┌─────────┼─────────┐
    ↓         ↓         ↓
[2. Case   [3. Transfer  [4. Message
 Handler]   Executor]     Taker]
    ↓         ↓             ↓
    └────→────┴──────→──────┘
         (handoffs as needed)
```

## Files in This Folder

### `/assistants/` - System Prompts (Copy into VAPI Dashboard)
| File | Agent | Entry Point? | Tools |
|------|-------|--------------|-------|
| `1_greeter_classifier.md` | Greeter & Classifier | YES (firstMessage) | Handoff only |
| `2_case_handler.md` | Case Handler | NO | `search_case_details`, Handoff |
| `3_transfer_executor.md` | Transfer Executor | NO | `classify_and_route_call`, Handoff |
| `4_message_taker.md` | Message Taker | NO | `take_message` |

### `/shared/` - Reusable Components (Embed in each prompt as needed)
| File | Purpose |
|------|---------|
| `persona.md` | Voice/personality (include in ALL agents) |
| `knowledge_base.md` | Firm facts, locations, fees (include in Greeter, Case Handler) |
| `staff_directory_template.md` | Template for injecting staff directory (include in ALL agents) |
| `voice_formatting.md` | Phone/email formatting rules (include in Case Handler, Message Taker) |

### `/extractors/` - Variable Extraction Schemas (For VAPI variableExtractionPlan)
| File | When to Use |
|------|-------------|
| `caller_identity.json` | Greeter → any handoff |
| `caller_classification.json` | Greeter → any handoff |
| `case_context.json` | Case Handler → Transfer/Message |
| `contact_info.json` | Message Taker → end of call |
| `frustration_level.json` | Any agent → next agent |

---

## VAPI Dashboard Setup Steps

### Step 1: Create Squad
1. Go to VAPI Dashboard → Squads → Create New Squad
2. Name: "AI Receptionist Pro"

### Step 2: Create Assistants
Create 4 assistants with these settings:

#### Assistant 1: Greeter & Classifier
- **Name**: `Greeter-Classifier` (exact name matters for handoffs)
- **First Message**: `"{{firm_name}}, this is {{agent_name}}. How can I help you?"`
- **First Message Mode**: `assistant-speaks-first`
- **System Prompt**: Copy from `assistants/1_greeter_classifier.md`
- **Tools**: Add handoff tool with destinations: Case-Handler, Transfer-Executor, Message-Taker

#### Assistant 2: Case Handler
- **Name**: `Case-Handler`
- **First Message**: `` (empty string - CRITICAL for silent handoff)
- **First Message Mode**: `assistant-speaks-first-with-model-generated-message`
- **System Prompt**: Copy from `assistants/2_case_handler.md`
- **Tools**: `search_case_details`, handoff tool

#### Assistant 3: Transfer Executor
- **Name**: `Transfer-Executor`
- **First Message**: `` (empty string)
- **First Message Mode**: `assistant-speaks-first-with-model-generated-message`
- **System Prompt**: Copy from `assistants/3_transfer_executor.md`
- **Tools**: `classify_and_route_call`, handoff tool

#### Assistant 4: Message Taker
- **Name**: `Message-Taker`
- **First Message**: `` (empty string)
- **First Message Mode**: `assistant-speaks-first-with-model-generated-message`
- **System Prompt**: Copy from `assistants/4_message_taker.md`
- **Tools**: `take_message`

### Step 3: Configure Handoff Tools

For each handoff tool, set:
```json
{
  "type": "handoff",
  "messages": [],  // MUST be empty for silent handoff
  "destinations": [
    {
      "type": "assistant",
      "assistantName": "Case-Handler",
      "description": "Caller needs case information lookup"
    },
    {
      "type": "assistant",
      "assistantName": "Transfer-Executor",
      "description": "Caller needs to be transferred to a person or department"
    },
    {
      "type": "assistant",
      "assistantName": "Message-Taker",
      "description": "Caller wants to leave a message"
    }
  ]
}
```

### Step 4: Configure Variable Extraction

In each handoff tool, add `variableExtractionPlan` using the schemas in `/extractors/`.

Example for Greeter → Case Handler:
```json
{
  "variableExtractionPlan": {
    "output": [
      {"name": "caller_name", "type": "string"},
      {"name": "caller_type", "type": "string"},
      {"name": "purpose", "type": "string"},
      {"name": "organization_name", "type": "string"},
      {"name": "client_name", "type": "string"},
      {"name": "frustration_level", "type": "number"}
    ]
  }
}
```

### Step 5: Add Squad Members

Add assistants to squad in order:
1. Greeter-Classifier (first = entry point)
2. Case-Handler
3. Transfer-Executor
4. Message-Taker

---

## Variable Placeholders

These prompts use `{{variable}}` placeholders. Replace or configure in VAPI:

| Variable | Source | Example |
|----------|--------|---------|
| `{{firm_name}}` | Static config | "Bey & Associates" |
| `{{agent_name}}` | Static config | "Fatima" |
| `{{firm_id}}` | Static config | 123 |
| `{{is_open}}` | Dynamic (webhook) | true/false |
| `{{intake_is_open}}` | Dynamic (webhook) | true/false |
| `{{case_details}}` | Pre-call lookup | Object or null |
| `{{caller_name}}` | Extracted variable | "Devon" |
| `{{caller_type}}` | Extracted variable | "INSURANCE_ADJUSTER" |

---

## Testing Checklist

Before going live, test these scenarios:

- [ ] New client inquiry (goes to intake)
- [ ] Insurance adjuster asking for case manager contact
- [ ] Existing client asking about case status
- [ ] Caller asking for specific staff member by name
- [ ] Medical provider asking about a patient
- [ ] Vendor calling about an invoice
- [ ] Transfer fails → message taken
- [ ] After-hours call → message taken
- [ ] Caller asks "Are you AI?"
- [ ] Frustrated caller
- [ ] Multiple cases / bulk inquiry → customer success

---

## Troubleshooting

### Agent says "Let me transfer you" during handoff
- Check that `messages: []` is set in handoff tool config
- Ensure system prompt includes "Never say transferring" rule

### Agent re-introduces itself after handoff
- Check destination assistant has empty `firstMessage`
- Check system prompt has "proceed without greetings" context block

### Variables not passing between agents
- Verify `variableExtractionPlan` is configured on handoff tool
- Check variable names match exactly (case-sensitive)

### Agent asks for info caller already provided
- Check handoff summary is being passed
- Verify extraction is capturing the data
