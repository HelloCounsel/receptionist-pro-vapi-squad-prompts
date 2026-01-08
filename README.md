# Receptionist Pro - VAPI Squad Agent

AI receptionist system prompts for Bey & Associates personal injury law firm. This repo contains prompt configurations for VAPI voice agents - backend code is in a separate repository.

## Directory Structure

```
├── prompts/
│   ├── squad/                    # 13-agent specialized squad system
│   │   ├── assistants/           # Individual agent prompts
│   │   ├── handoff_tools/        # Routing and tool configurations
│   │   └── vapi_config/          # VAPI-specific settings
│   │
│   └── standalone/               # Standalone assistant (pre-identified caller)
│       └── pre_identified_caller/
│
├── scripts/                      # Python utilities for VAPI integration
├── data/                         # Script outputs and configs
├── docs/                         # Architecture and planning docs
│
├── vapi_id_mapping.json          # Sandbox/Production assistant IDs
└── CHANGELOG.md                  # Issue tracking and fixes
```

## Active Configurations

### Squad (13 Agents)
Entry point is the **Greeter Classifier** which routes callers to specialized agents:
- Existing Client, New Client, Insurance Adjuster, Medical Provider
- Vendor, Direct Staff Request, Family Member, Spanish Speaker
- Legal System, Referral Source, Sales Solicitation

### Standalone Pre-Identified Caller
Alternative entry point for callers matched by phone number lookup - bypasses classification.

## VAPI CLI

The repo is configured for VAPI CLI usage. See `.claude/settings.local.json` for permissions and `vapi_id_mapping.json` for environment IDs.

## Key Files

- `prompts/squad/assistants/01_greeter_classifier.md` - Main entry point
- `prompts/squad/handoff_tools/greeter_handoff_destinations.md` - Routing rules
- `CHANGELOG.md` - Bug fixes and improvements history
