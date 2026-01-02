# Staff Directory Template

Include this section in ALL assistant prompts.

Replace the placeholder data with your actual staff directory. This can be:
1. Hardcoded in the prompt
2. Injected dynamically via webhook before call
3. Stored as a VAPI variable

---

## Staff Directory

### Case Managers
{{#each staff_directory.case_managers}}
- {{name}} (ID: {{id}})
  - Phone: {{phone}}
  - Email: {{email}}
{{/each}}

### Paralegals
{{#each staff_directory.paralegals}}
- {{name}} (ID: {{id}})
  - Phone: {{phone}}
{{/each}}

### Intake Specialists
{{#each staff_directory.intake_specialists}}
- {{name}} (ID: {{id}})
  - Phone: {{phone}}
{{/each}}

### Attorneys
{{#each staff_directory.lawyers}}
- {{name}} (ID: {{id}})
  - Phone: {{phone}}
  - Email: {{email}}
{{/each}}

### Receptionists
{{#each staff_directory.receptionists}}
- {{name}} (ID: {{id}})
{{/each}}

### Admins
{{#each staff_directory.admins}}
- {{name}} (ID: {{id}})
{{/each}}

---

## Example Hardcoded Format

If injecting dynamically isn't available, hardcode like this:

```markdown
## Staff Directory

### Case Managers
- Sarah Johnson (ID: 25)
  - Phone: 404-555-0101
  - Email: sarah.johnson@beyandassociates.com
- Michael Sole (ID: 42)
  - Phone: 404-555-0102
  - Email: michael.sole@beyandassociates.com
- Victoria Morales (ID: 18)
  - Phone: 404-555-0103
  - Email: victoria.morales@beyandassociates.com

### Intake Specialists
- Taylor Middleton (ID: 7)
  - Phone: 404-555-0201
- Paige Wilson (ID: 12)
  - Phone: 404-555-0202

### Attorneys
- David Bey (ID: 1)
  - Phone: 404-555-0001
  - Email: david.bey@beyandassociates.com
```

---

## Homophone Awareness

Voice transcription often mishears names. When searching, check variants:

| Transcribed | Also Check |
|-------------|------------|
| Soul | Sole |
| Steel | Steele |
| Smith | Smyth |
| Brown | Browne |
| Green | Greene |
| Lane | Laine |
| O'Brian | O'Brien |
| Paige | Payge |
| Caitlin | Kaitlyn, Caitlyn |

If caller asks for "Michael Soul" and you have "Michael Sole" (ID: 42), that's a match.
