---
name: ai-code-review
description: Review AI-generated code for correctness, error handling, tests, security, quality and performance. Use when reviewing code an AI assistant produced, before merging a generated change, or when the user asks to check generated code for defects.
effort: xhigh
---

# AI Code Review

Review the code against every check below. If a diff and the original prompt are available, use them; the correctness checks need them, the rest read the file as it stands.

Read every file the diff touches in full before judging any change. A diff shows what moved, not what the code now does.

## How to work

Three passes. Finish each before starting the next, and say which pass you are in as you go.

1. **Does it do what was asked?** — the prompt against the code, point by point.
2. **Does it break?** — every input path, every failure mode.
3. **Should it look like this?** — structure, duplication, performance, security.

Never write "may fail", "could be improved", "consider adding" or "is not robust". Every finding names a concrete input and states what happens with it. If you cannot construct that input, you have not found anything — drop it.

## Checks

### Pass 1 — Correctness and logic

- **Missed requirement**: go through the prompt point by point and mark each one done, partial or missing.
- **Unrequested functionality**: anything not asked for, including extra features, dependencies, file writes and network calls.
- **Wrong business logic**: code that runs but returns the wrong result. For every formula and condition, say what it actually does, then trace one input and give expected vs actual.
- **Hallucinated functionality**: functions, methods, parameters, imports or endpoints that do not exist. List every external symbol and mark it verified or not.
- **Incorrect assumption**: decisions made about things the prompt left open, such as argument order, arity, units, formats, empty values or error behaviour.
- **Broken existing code**: every removed guard, changed signature, changed return type, renamed symbol or altered error handling.

### Pass 2 — Error handling and input validation

- **Empty values**: everywhere the code reads input, confirm it handles an empty string, and check any output that reads from an empty collection.
- **Invalid input**: every conversion from text to a number, confirm it is guarded against text that isn't numeric.
- **Unexpected data**: every function whose return can be an error string, `None`, or another sentinel, confirm every caller checks for it before treating it as the expected type.
- **Boundary cases**: every arithmetic operation, confirm it is bounded against zero, negative, and very large inputs.
- **Exceptions**: confirm anything that can raise is caught, and that state isn't lost when it does.
- **Missing error handling**: every function that can fail, confirm it reports failure through its return value or an exception, not through a `print()` plus a sentinel.

### Pass 3 — Tests, quality, security

**Tests and edge cases**

- **Missing scenarios**: for every function, check whether its behaviour is defined for every input it can receive. Flag any input with no stated expected result.
- **Edge cases**: for every guard, find one input outside what it checks and show it still failing. For every rounding call or float comparison, give one input where the result differs from what a person would get by hand.
- **Test coverage**: say whether a test suite exists and what it covers. Report a gap as a finding. Write tests only when asked.

**Quality and performance**

- **Duplicated logic**: any function, guard or test block that only repeats something already available, and what it could call instead.
- **Poor structure**: any global state mutated from outside the scope that owns it, and any function mixing input, computation, formatting and storage that could be split.
- **Overengineering**: any field, branch or class boundary serving fewer cases than its presence implies, and what could be removed without changing behaviour.
- **Inefficient code**: any value that grows without bound relative to what's read back, and any work repeated every loop pass that doesn't need to be.

**Security**

- **Unsafe user input**: any input reaching a shell, a query, a path, a deserializer or an eval without validation.
- **Exposed credentials**: any key, token, password or connection string in source.
- **Problematic dependencies**: any import that is unpinned, unused, unnecessary, or duplicates the standard library.

## Output

Print the report and nothing else. No preamble, no "I reviewed the code and found", no closing recap of what you just said.

A verdict line, then the findings.

```
BLOCKED — 2 blockers, 3 defects, 4 notes

[1] blocker · src/cart.ts:42
    Voucher applied before tax. A 100 cart with a 10 voucher and
    20% tax charges 108, not 110.
    → Move the voucher subtraction below the tax on line 44.

[2] defect · src/form.tsx:18
    Empty title is accepted. Submitting a blank form adds an
    unnamed item to the list.
    → Guard the submit handler against an empty trimmed title.
```

**Verdict** — `BLOCKED` if any blocker, `NEEDS WORK` if any defect, else `CLEAN`.

**Severity** — `blocker`: runs but gives the wrong answer, loses data, or exposes something. `defect`: crashes or fails on a realistic input. `note`: structure, duplication, performance, missing tests.

**Findings** — numbered, worst first. Three lines each: location, what happens with a concrete input, the fix. Max five notes; blockers and defects uncapped.

Do not rewrite the code under review.

Do not invent issues. A review that finds nothing is a successful review — reporting a non-issue costs more than missing a minor one, because the reader stops trusting the report.

## Ask before fixing

After the report, call AskUserQuestion. Do not fix anything before you have an answer.

```json
{
  "questions": [
    {
      "question": "What should I fix?",
      "header": "Fix scope",
      "options": [
        {
          "label": "Blockers only",
          "description": "[n] findings — wrong results, data loss, exposure"
        },
        {
          "label": "Blockers and defects",
          "description": "[n] findings — adds crashes and realistic failures"
        },
        {
          "label": "Everything",
          "description": "[n] findings — adds structure, duplication, tests"
        },
        {
          "label": "Nothing",
          "description": "Leave the code as it is. The report is enough."
        }
      ],
      "multiSelect": false
    },
    {
      "question": "Should I write the missing tests?",
      "header": "Tests",
      "options": [
        {
          "label": "Yes",
          "description": "Cover the scenarios and edge cases found above"
        },
        {
          "label": "No",
          "description": "Report the gap, leave the suite as it is"
        }
      ],
      "multiSelect": false
    }
  ]
}
```

Replace `[n]` with the counts from what you actually found. Drop any option whose count is zero, keeping at least two options and always keeping "Nothing". Include the second question only when the review found a test coverage gap. If there are no findings at all, skip this step — there is nothing to ask.

Keep the options in this order. Do not mark one as recommended.

Once answered, fix only what was selected, and write tests only if asked to. Report each fix in one line: finding number, file, what changed. Do not re-review your own fixes in the same pass.
