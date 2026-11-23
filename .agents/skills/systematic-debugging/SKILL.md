---

name: systematic-debugging
description: Diagnoses bugs, failures, and unexpected behavior through evidence-driven investigation, root-cause isolation, minimal fixes, and regression verification.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Systematic Debugging

Use this skill when investigating bugs, failing tests, unexpected behavior, or inconsistent results.

## Debugging Protocol

1. **Define the symptom**

   * Identify expected behavior and actual behavior.
   * Capture the relevant error message, input, environment, and reproduction steps.
   * Reduce the problem to the smallest reliable reproduction when practical.

2. **Gather evidence**

   * Inspect stack traces, logs, failing tests, network requests, database state, configuration, and relevant source code.
   * Prefer observable evidence over assumptions.
   * Check recent changes that could plausibly affect the behavior.

3. **Form hypotheses**

   * Develop plausible explanations based on the available evidence.
   * Avoid locking onto the first explanation.
   * Test hypotheses in an order that efficiently eliminates possibilities.

4. **Trace the failure**

   * Follow the execution path from the observed symptom toward its source.
   * Identify the earliest point where actual behavior diverges from expected behavior.

5. **Identify the root cause**

   * Distinguish the underlying cause from secondary symptoms.
   * Do not declare a root cause solely because a change makes the symptom disappear.

6. **Apply the minimal sufficient fix**

   * Change only what is necessary to correct the root cause.
   * Avoid unrelated refactoring during a focused bug fix.
   * Preserve existing behavior outside the affected area.

7. **Verify the fix**

   * Reproduce the original failure and confirm it no longer occurs.
   * Run relevant automated tests and validation.
   * Test important adjacent cases when the change could affect them.

8. **Prevent regression**

   * Add or update a test when practical and appropriate.
   * If a test is not feasible, document the verification performed.

## AI Slop Prevention

* Do not guess and immediately rewrite code.
* Do not repeatedly apply speculative fixes without new evidence.
* Do not modify unrelated files simply because they look imperfect.
* Do not hide errors or weaken validation merely to make a test pass.
* Do not declare success without verification.
