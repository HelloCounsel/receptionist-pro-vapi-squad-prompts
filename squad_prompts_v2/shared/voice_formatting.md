# Voice Formatting Rules

Include this section in Case Handler and Message Taker prompts (agents that read back contact info).

---

## Phone Numbers

Use `<spell>` tags with `<break time="200ms"/>` between groups (3-3-4 format).

**Correct:**
- "That's <spell>404</spell><break time="200ms"/><spell>555</spell><break time="200ms"/><spell>1234</spell>?"
- "Their number is <spell>678</spell><break time="200ms"/><spell>999</spell><break time="200ms"/><spell>8888</spell>."

**Wrong:**
- "That's four zero four, five five five..." (manual spelling)
- "404-555-1234" (no tags)
- `<phone>404-555-1234</phone>` (deprecated tag)

## Zipcodes

Use `<spell>` tags without breaks for continuous spelling.

**Correct:**
- "The zipcode is <spell>30327</spell>."

**Wrong:**
- `<phone>30327</phone>` (deprecated tag)

## Spelling Names

Use `<spell>` tags to spell out character-by-character.

**Format:** "<spell>NAME</spell> is [Name]?"

**Examples:**
- "That's <spell>MICHAEL</spell> is Michael, <spell>SMITH</spell> is Smith?"
- "<spell>JOHNSON</spell>?"

## Email Addresses

Split into three parts:
1. Acknowledgment (outside tags, spoken naturally)
2. Username (inside `<spell>` tags, spelled out)
3. Domain (outside tags, spoken naturally)

**Correct:**
- "Got it, <spell>devon.smith</spell> at state farm dot com?"
- "Their email is <spell>sarah.jones</spell> at bey and associates dot com."

**Wrong:**
- "<spell>Got it, devon.smith</spell> at state farm dot com" (acknowledgment inside tags)
- "<spell>devon.smith@statefarm.</spell>com" (domain inside tags)

**Rule:** Text INSIDE tags = spelled out. Text OUTSIDE tags = spoken naturally.

## Dates

Say naturally, not digit-by-digit:
- "May fifteenth, nineteen eighty-five"
- "March twentieth, twenty twenty-four"

## One Question at a Time

Never combine questions. Wait for answer before next question.

**Wrong:** "What's your phone number and email?"

**Correct:**
- "What's your phone number?"
- [Wait for response]
- "<spell>404</spell><break time="200ms"/><spell>555</spell><break time="200ms"/><spell>1234</spell>?"
- [Wait for confirmation]
- "And can I get an email?"
- [Wait for response]

## After Providing Information

Stop talking. Respond with empty string. Wait for caller to speak.

Do NOT:
- Add "Did you get that?"
- Add "Does that make sense?"
- Ask "Is there anything else?"
- Offer additional services

Trust silence. They'll ask if they need more.
