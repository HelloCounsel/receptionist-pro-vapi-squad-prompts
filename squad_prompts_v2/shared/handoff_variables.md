# Handoff Variables Reference

This document describes all variables passed between agents during handoffs.

---

## Greeter-Classifier → Case-Handler / Transfer-Executor / Message-Taker

| Variable | Type | Description |
|----------|------|-------------|
| `caller_name` | string | The caller's full name as they provided it |
| `caller_type` | string | Classification: "existing_client", "new_client", "insurance", "medical_provider", "vendor", "legal_system", "family_member", "spanish_speaker", "sales" |
| `purpose` | string | What the caller needs (e.g., "case update", "speak with case manager", "billing inquiry") |
| `organization_name` | string | Company/organization name for business callers (e.g., "State Farm", "Emory Hospital") |
| `client_name` | string | For external callers (insurance, medical, law office) - the CLIENT they are calling about (not the caller's name) |
| `target_staff_name` | string | If caller requested a specific staff member by name |
| `destination` | string | For Transfer-Executor: "intake", "finance", "customer_success", "spanish", "insurance" |

---

## Case-Handler → Transfer-Executor / Message-Taker

| Variable | Type | Description |
|----------|------|-------------|
| `case_manager_name` | string | Name of the assigned case manager from file lookup |
| `case_manager_id` | string | Staff ID of the case manager (for transfer tool) |
| `staff_id` | string | ID of specific staff member to transfer to |
| `case_status` | string | Current case status (e.g., "Active", "Pre-Lit Treating", "Settled") |
| `destination` | string | For escalations: "customer_success" |
| `reason` | string | Why escalation is needed (e.g., "file not found after spelling", "multiple matches") |

---

## Transfer-Executor → Message-Taker

| Variable | Type | Description |
|----------|------|-------------|
| `caller_name` | string | Passed through from earlier agents |
| `caller_type` | string | Passed through from earlier agents |
| `purpose` | string | Passed through from earlier agents |
| `case_manager_name` | string | If known from previous lookup |
| `transfer_outcome` | string | Why message is being taken: "failed" (couldn't reach), "declined" (caller said no), "closed" (office closed) |
| `transfer_attempted_to` | string | Who/what department was attempted (e.g., "Sarah Johnson", "insurance department") |
| `phone_number` | string | If already collected |
| `email` | string | If already collected |
