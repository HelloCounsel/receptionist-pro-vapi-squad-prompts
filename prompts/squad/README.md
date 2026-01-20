# Squad Variants

This directory contains different variants of the receptionist squad, each designed to support different client firm policies regarding caller verification and information sharing.

## Variants

### `/lenient/`
The **lenient** variant provides open information sharing with minimal verification requirements. This is suitable for firms that prioritize accessibility and quick service.

**Characteristics:**
- Minimal caller verification before sharing information
- Open information sharing policy
- Streamlined call flow with fewer verification steps

### `/strict/`
The **strict** variant requires thorough caller verification before sharing any information. This is suitable for firms with strict confidentiality requirements.

**Characteristics:**
- Thorough caller verification required before sharing any information
- Strict information sharing controls
- Additional verification tools and escalation paths

> **Note:** The strict variant currently contains a copy of the lenient variant as a base. Future modifications will add verification requirements, verification tools, and verification failure escalation paths.

## Directory Structure

Each variant contains:
```
variant/
├── README.md                      # Variant-specific documentation
├── vapi_variable_extraction_plans.md  # Variable extraction configuration
├── assistants/                    # Agent prompts (13 agents)
├── handoff_tools/                 # Tool configurations
├── vapi_config/                   # Squad settings
└── standalone/                    # Standalone agent prompts
    └── pre_identified_caller/     # Pre-identified caller flow
```

## Choosing a Variant

| Requirement | Recommended Variant |
|-------------|---------------------|
| Quick service, accessibility focus | `lenient` |
| Strict confidentiality, compliance focus | `strict` |
| Default/general use | `lenient` |
