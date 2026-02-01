1. always update readme.md and changelog.md after every change is made
2. always assess the root cause of an issue or a behaviour before starting any work
3. always exercise first principles thinking
4. always think second order of your changes
5. when fixing bugs solve the root cause and not symptoms
6. think like an expert prompt engineer when generating or modifying prompts
7. whenever a change to the file is made, undertsand the format/architecture of the file and folow it
8. when modifying prompt solution might not just be adding more instruction, sometimes it might be removing it - think critically before making changes

## VAPI Agent Debugging Process

When debugging agent behavior issues, follow this process:

### 1. Reconstruct the Timeline First
Before proposing fixes, extract exact timestamps from call logs and map what actually happened vs. what should have happened. Identify the precise moment where behavior diverged from expectation.

### 2. Compare to Working Agents
Find an agent that handles the same scenario successfully. Read its full prompt and identify structural differences - the fix is usually adopting the working pattern, not adding more instructions to the broken one.

### 3. Understand Model Behavior vs. Prompt Intent
If the prompt says "do X immediately" but the model doesn't, ask why. Models interpret sequentially. Instructions like "IMMEDIATELY" are hints, not guarantees. The fix often requires changing the interaction pattern (e.g., adding a question to create a second turn).

### 4. Adopt Proven Patterns Over New Instructions
Copy structural patterns from working agents rather than inventing new instruction text. A proven pattern (like two-turn consent flow) provides mechanical guarantees that instruction emphasis cannot.

### 5. Check Structural Consistency
Verify the agent has all standard sections that working agents have. Missing sections (error handling, voice formatting, role clarification) often cause subtle failures.