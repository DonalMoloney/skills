---
name: prompt-craft
description: Design, debug, and improve prompts for large language models — use when writing or fixing a system prompt, few-shot template, chain-of-thought sequence, or prompt pipeline.
---

# Prompt Craft

## The Law
**A prompt is a specification — write it with the same precision you would write a function signature: ambiguity in the prompt produces ambiguity in the output, every time.**

## When to Use
- Writing a system prompt for an application, agent, or tool
- A model is returning inconsistent or wrong outputs and the cause is unclear
- Designing a few-shot template for a classification, extraction, or generation task
- Building a multi-step chain and the handoff between steps is producing errors
- **Never skip when:** the same prompt will run at scale — inconsistency that appears rarely in manual testing compounds into noise at high volume

## Process

### Phase 1: Define the Output Contract
1. Before writing a single word of prompt text, specify exactly what the output must be:
   - Format: plain text, JSON, markdown, code, a number, a list — pick one
   - Length: one sentence, one paragraph, one code block, a list of N items
   - Tone and register: formal, technical, conversational, directive
   - What must never appear in the output (confidential data, hallucinated URLs, first-person hedging, etc.)
2. Write the output contract as a comment or note before the prompt — it is the acceptance criterion for the prompt.
3. A prompt without an output contract cannot be evaluated — do not proceed without one.

### Phase 2: Write the System Prompt
1. Open with a role declaration that gives the model a stable persona: "You are a..."
   - Make the role specific enough to constrain the domain of knowledge: "You are a Python backend engineer reviewing API contracts" beats "You are a helpful assistant".
2. State the primary task in one sentence in the imperative: "Classify the following support ticket into exactly one of these categories: ..."
3. List constraints and rules in a numbered or bulleted block — one rule per line:
   - Constraints on content ("Never include pricing information")
   - Constraints on format ("Return only valid JSON — no prose before or after")
   - Constraints on reasoning ("Base your answer only on the provided context")
4. State the output format explicitly at the end of the system prompt — repeat the format spec from Phase 1 in the model's language.

Python example — structured system prompt construction:
```python
def build_system_prompt(role: str, task: str, rules: list[str], output_format: str) -> str:
    rules_block = "\n".join(f"{i+1}. {rule}" for i, rule in enumerate(rules))
    return f"""You are {role}.

{task}

Rules:
{rules_block}

Output format:
{output_format}"""

system = build_system_prompt(
    role="a senior data engineer reviewing SQL queries for correctness and performance",
    task="Review the SQL query provided by the user and identify any correctness issues, missing indexes, or N+1 patterns.",
    rules=[
        "Point out only genuine issues — do not flag style preferences",
        "If the query is correct and efficient, say so explicitly",
        "Never rewrite the query unless the user asks",
    ],
    output_format='Return a JSON object: {"issues": [{"type": str, "line": int, "explanation": str}], "verdict": "ok"|"issues-found"}',
)
```

### Phase 3: Add Few-Shot Examples
1. Include few-shot examples when the task has an output pattern that is easier to demonstrate than to describe.
2. Rule of thumb: 2–6 examples for classification tasks; 1–3 for generation tasks; 0 for tasks where examples would constrain the model too narrowly.
3. Format each example as a complete input/output pair — do not abbreviate:
   ```
   Input: "The payment failed three times and now my account is locked."
   Output: {"category": "billing", "urgency": "high", "sentiment": "frustrated"}
   ```
4. Ensure examples cover edge cases, not just the easy central case — the model learns from the boundary examples more than from the typical ones.
5. Place few-shot examples after the system prompt rules and before the actual user input.

### Phase 4: Apply Chain-of-Thought Where Needed
1. Use chain-of-thought (CoT) when the task requires multi-step reasoning and intermediate steps must be traceable.
2. Two forms:
   - **Zero-shot CoT**: append "Think step by step before giving your final answer." to the user message — effective for arithmetic, logic, and planning tasks.
   - **Few-shot CoT**: include examples where the reasoning trace is shown explicitly before the final answer — more reliable than zero-shot CoT for complex tasks.
3. Separate the reasoning trace from the final answer in the output format:
   ```
   Return your response in two parts:
   <reasoning>
   ... your step-by-step thinking ...
   </reasoning>
   <answer>
   ... final answer only ...
   </answer>
   ```
4. Never ask for CoT and demand a one-word answer — the constraint pair is contradictory and produces unreliable output.

### Phase 5: Debug a Failing Prompt
1. Isolate the failure type before changing anything:
   - **Wrong format**: the output structure is off — add or tighten the format spec in the system prompt.
   - **Wrong content**: the model ignored a constraint — make the constraint more explicit; move it higher in the system prompt.
   - **Inconsistent output**: runs differ on the same input — add a format example; lower temperature if the API allows it.
   - **Hallucination**: the model invents facts — add "Base your answer only on the provided context; if the answer is not in the context, say so."
   - **Refusal**: the model declines a legitimate task — rephrase to clarify the legitimate use case; avoid language that pattern-matches to disallowed categories.
2. Change one thing per debugging iteration — changing multiple variables makes the cause untraceable.
3. Test with at least 5 diverse inputs after each change before concluding the fix holds.

Python example — prompt A/B test harness:
```python
import anthropic

client = anthropic.Anthropic()

def test_prompt(system: str, user_inputs: list[str]) -> list[str]:
    results = []
    for user in user_inputs:
        msg = client.messages.create(
            model="claude-sonnet-4-6",
            max_tokens=512,
            system=system,
            messages=[{"role": "user", "content": user}],
        )
        results.append(msg.content[0].text)
    return results

inputs = [
    "The invoice is overdue by 30 days.",
    "My subscription renewed but I didn't authorise it.",
    "Can I get a refund for last month?",
    "The payment portal is broken.",
    "I was charged twice.",
]

results_a = test_prompt(system_v1, inputs)
results_b = test_prompt(system_v2, inputs)

for i, (a, b) in enumerate(zip(results_a, results_b)):
    match = a == b
    print(f"Input {i+1}: {'same' if match else 'DIFFER'}")
    if not match:
        print(f"  v1: {a}")
        print(f"  v2: {b}")
```

### Phase 6: Model-Agnostic Portability Check
1. Remove any model-specific idioms (OpenAI function-calling syntax, Anthropic tool-use blocks) from the core prompt text — keep model-specific wiring in the API call layer, not in the prompt.
2. Test the prompt against a second model if the application may need to swap providers.
3. Document the model the prompt was tuned against and the date — prompt effectiveness drifts with model version updates.

## Red Flags — Stop Immediately
- The prompt has no explicit output format — every model will invent one
- You are changing the system prompt and the user message in the same debugging iteration
- A few-shot example shows only the easy central case — add an edge case before going to production
- The system prompt is over 2000 tokens for a simple classification task — trim ruthlessly
- You are treating a temperature change as the fix for a content accuracy problem — temperature affects randomness, not knowledge

## Common Rationalizations
| Excuse | Why It's Wrong |
|--------|----------------|
| "The model is smart enough to figure out the format" | Every format ambiguity produces at least two different valid interpretations; the model picks one arbitrarily |
| "I only need to test it a few times" | Five successful runs prove nothing for a prompt running at scale — test across 20+ diverse inputs |
| "Chain-of-thought makes every prompt better" | CoT adds latency and token cost; skip it for tasks that require only direct recall or lookup |
| "I'll clean up the system prompt later" | A messy system prompt becomes the production system prompt; there is no later |
| "The model refused, so the task is impossible" | Refusals are almost always prompt framing issues — rephrase before concluding the task cannot be done |
| "Lowering temperature fixed the wrong-answer problem" | Temperature controls variance, not accuracy; the fix is better grounding in the prompt |

## Quick Reference
| Phase | Core Action | Done When |
|-------|-------------|-----------|
| 1. Output Contract | Specify format, length, tone, exclusions | Written and agreed before prompt drafting begins |
| 2. System Prompt | Role + task + rules + format spec | All four elements present and unambiguous |
| 3. Few-Shot Examples | Add 2–6 input/output pairs covering edge cases | Examples include non-obvious boundary cases |
| 4. Chain-of-Thought | Add CoT only when reasoning must be traceable | Reasoning trace separated from final answer in output |
| 5. Debug | Isolate failure type; change one variable per iteration | 5+ diverse inputs pass after each fix |
| 6. Portability | Strip model-specific idioms; document tuning model | Prompt works without provider-specific wiring |
