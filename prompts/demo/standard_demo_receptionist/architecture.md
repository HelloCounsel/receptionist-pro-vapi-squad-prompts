# Standard Demo Receptionist - Architecture

This document describes the prompt architecture for the Standard Demo Receptionist assistant.

---

## 1. Overview

| Attribute | Value |
|-----------|-------|
| Agent Name | Kate |
| Firm | Duffy and Duffy Law Firm |
| Firm ID | 8 |
| Practice Area | Personal Injury (MVA) |
| Architecture | Standalone (single-agent) |
| Tools | search_case_details, transfer_call |

**Purpose:** All-in-one demo receptionist that handles all caller types without handoffs to other agents. Designed for client demos with a simplified, self-contained flow.

---

## 2. Prompt Structure

The prompt follows a structured XML-like format:

```
<agent_identity>         → Agent name, firm config, system variables
<greeting>               → First message template
<staff_directory>        → Hardcoded staff list for the demo
<conversation_rules>     → Behavioral constraints
<caller_identification>  → 5 caller types with routing rules
<call_flows>             → 9 distinct flows with step-by-step logic
<routing_logic>          → Transfer destinations and failure handling
<tool_usage>             → Tool definitions and usage patterns
<callback_time_calculation> → Time calculation logic for callbacks
<style_guidelines>       → Tone and response patterns
```

---

## 3. Caller Classification Matrix

| Caller Type | Detection Signals | Destination Flow |
|-------------|-------------------|------------------|
| Existing Client | "calling about my case", "need to talk to case manager", provides name for case lookup | existing_client_flow |
| Medical Provider | Name + Organization + "patient" | medical_provider_flow |
| Insurance Adjuster | Name + Company + "claimant" | insurance_adjuster_flow |
| New Client | "I was in an accident", "I need a lawyer", "I was injured" | new_client_flow |
| Escalation Request | Frustration, "human", "manager", refuses to leave message | escalation_flow |

### Classification Rules

1. **Existing Client** - Has a case with the firm, needs case manager
2. **Medical Provider** - Calling from a healthcare organization about a patient
3. **Insurance Adjuster** - Calling from an insurance company about a claimant
4. **New Client** - Potential client who needs legal help
5. **Escalation** - Caller explicitly requests human assistance or is frustrated

---

## 4. Call Flows

### 4.1 Primary Flows

| Flow | Goal | Steps |
|------|------|-------|
| existing_client_flow | Connect to case manager | Get name → Search case → Announce transfer → Attempt transfer |
| medical_provider_flow | Provide case status + email | Get patient name → Search case → Provide info (status + case manager) → Wait for response |
| insurance_adjuster_flow | Provide case status + email | Get claimant name → Search case → Provide info (status + case manager) → Wait for response |
| new_client_flow | Transfer to intake | Get name → Announce transfer → Attempt transfer |
| escalation_flow | Transfer to customer success | Acknowledge → Announce transfer → Attempt transfer |

### 4.2 Fallback Flows (Transfer Failures)

| Flow | Trigger | Action |
|------|---------|--------|
| callback_scheduling_flow | Existing client transfer fails | Collect callback number → Collect reason → Offer callback time |
| message_flow_external | Provider/adjuster transfer fails | Collect callback number → Confirm message |
| message_flow_new_client | New client transfer fails | Collect callback number → Collect preferred time → Collect notes |
| message_flow_customer_success | Customer success transfer fails | Collect callback number → Collect notes |

### 4.3 Utility Flows

| Flow | Purpose |
|------|---------|
| transfer_for_assistance | When provider/adjuster needs more than status info |

---

## 5. Call Flow Diagrams

### Existing Client Flow

```
┌─────────────────┐
│ Caller Arrives  │
└────────┬────────┘
         ▼
┌─────────────────┐
│  Get Name       │
│  (if needed)    │
└────────┬────────┘
         ▼
┌─────────────────┐
│ search_case_    │
│ details         │
└────────┬────────┘
         ▼
    ┌────┴────┐
    │ Found?  │
    └────┬────┘
    Yes  │  No
    ▼    └──────► Ask for alternate name
┌─────────────────┐
│ Announce        │
│ Transfer        │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Wait for        │
│ Acknowledgment  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ transfer_call   │
└────────┬────────┘
         ▼
    ┌────┴────┐
    │Success? │
    └────┬────┘
    Yes  │  No
    ▼    └──────► callback_scheduling_flow
┌─────────────────┐
│ "I have [CM]    │
│ on the line"    │
└─────────────────┘
```

### Medical Provider / Insurance Adjuster Flow

```
┌─────────────────┐
│ Caller Arrives  │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Get caller info │
│ (name, org)     │
└────────┬────────┘
         ▼
┌─────────────────┐
│ Get patient/    │
│ claimant name   │
└────────┬────────┘
         ▼
┌─────────────────┐
│ search_case_    │
│ details         │
└────────┬────────┘
         ▼
┌─────────────────────────────┐
│ Provide status + CM name    │
│ *** STOP - wait silently ***│
└────────┬────────────────────┘
         ▼
    ┌────────────────┐
    │ Caller asks    │
    │ for more?      │
    └────────┬───────┘
    │        │        │
   Done  Email req   More help
    │        │        │
    ▼        ▼        ▼
  Goodbye  Give    transfer_for_
           email   assistance
```

---

## 6. Tool Integration

### search_case_details

| Attribute | Value |
|-----------|-------|
| Type | API Request |
| Tool ID | 2f4bd459-4587-4563-8241-78c9b884cc1b |
| Input | caller_name (string) |
| Output | client_full_name, case_status, case_manager, staff_id |

**Usage Pattern:**
```
1. Collect name from caller
2. Call search_case_details with name
3. Handle response:
   - Found: Extract case_manager, staff_id
   - Not found: Ask for alternate name
```

### transfer_call

| Attribute | Value |
|-----------|-------|
| Type | Function |
| Tool ID | 3b6b2ba2-d30b-4cc5-a982-8c9c7749018f |

**Input Patterns:**

| Destination | Parameters |
|-------------|------------|
| Case manager | staff_id, firm_id: 8 |
| Intake queue | firm_id: 8, caller_type: "new_case" |
| Customer success | firm_id: 8, caller_type: "customer_success" |

**Critical Rules:**
1. ALWAYS announce transfer first
2. ALWAYS wait for acknowledgment before calling
3. Never call silently
4. On failure: go directly to fallback flow (no "transfer failed" message)

---

## 7. Key Design Patterns

### Pattern 1: One Question Per Turn
Never combine multiple questions in a single response. Ask one question, wait for response, then proceed.

**Wrong:** "May I have your name and what's this regarding?"
**Correct:** "May I have your name?" → wait → "Thanks. What can I help you with?"

### Pattern 2: Announce Before Transfer
Always get caller acknowledgment before executing transfer_call.

**Pattern:**
```
Agent: "Let me get you over to [person]. Is that alright?"
Caller: "Yes" / "Okay" / "Sure"
Agent: [calls transfer_call]
```

### Pattern 3: Silent Wait After Info
After providing requested information, STOP and wait. Do not prompt with "anything else?".

**Wrong:** "The case is in discovery and Sarah is the case manager. Is there anything else I can help with?"
**Correct:** "The case is in discovery and Sarah is the case manager." → [silence, wait for caller]

### Pattern 4: Callback Time Calculation (Existing Clients Only)
```
current_time → round up to hour → add 2 hours
Example: 1:23 PM → 2:00 PM → 4:00 PM
```

Only offer callback times to existing clients. For external callers (providers, adjusters), just take the message.

### Pattern 5: Phone Number Readback
Read digit by digit with natural grouping, then WAIT for confirmation.

**Pattern:**
```
Agent: "Five five five, three two one, seven seven eight eight."
Caller: "Yes" / "That's right"
Agent: "Got it."
```

### Pattern 6: Email-Only Contact for External Callers
When medical providers or insurance adjusters ask for case manager contact:
- Provide email only
- Do NOT offer phone number

This is a demo-specific configuration (hence "only email" in the name).

---

## 8. Differences from Squad Architecture

| Aspect | Demo (Standalone) | Production (Squad) |
|--------|-------------------|-------------------|
| **Agent Count** | 1 | 13+ |
| **Caller Classification** | Internal flow logic | Greeter classifier agent |
| **Handoffs** | None (self-contained) | Agent-to-agent handoffs |
| **Context Transfer** | N/A | variableExtractionPlan |
| **Case Manager Contact** | Email only | Phone + email |
| **Staff Directory** | Hardcoded in prompt | RAG-based lookup tool |
| **Firm Config** | Duffy and Duffy | Bey & Associates |

---

## 9. Variables

### System Variables
| Variable | Source | Usage |
|----------|--------|-------|
| {{firm_name}} | VAPI | Greeting, references |
| {{current_time}} | VAPI | Callback time calculation |

### Staff Directory (Hardcoded)
| Name | Staff ID | Role | Email |
|------|----------|------|-------|
| Brittany | 69 | Case Manager | brittany@pendergastlaw.com |
| Sarah Johnson | 70 | Case Manager | sarah@pendergastlaw.com |

---

## 10. Error Handling

### Transfer Failure Response Pattern
1. Do NOT say "transfer failed" or similar
2. Do NOT ask "would you like to leave a message?"
3. Go directly to the fallback flow's first response

**Example:**
```
[transfer_call fails]
Agent: "[Case_manager]'s not available right now. Let me take a message and make sure she gets back to you. What's the best number for her to reach you?"
```

### Case Not Found
1. Ask for alternate name or spelling
2. If still not found, offer to take a message for the team

---

## 11. Style Guidelines Summary

| Guideline | Example |
|-----------|---------|
| Warm but efficient | "Got it. Let me look that up." |
| Natural transitions | "One moment", "Let me look that up" |
| Vary acknowledgments | "Got it", "Perfect", "Thanks", "Sure thing" |
| Brief empathy for new clients | "I'm sorry to hear that" |
| No salesy language | Never "You've called the right place" |
| No over-apologizing | One acknowledgment of unavailable transfer is enough |
