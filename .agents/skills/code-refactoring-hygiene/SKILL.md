---

name: code-refactoring-hygiene
description: Improves existing code through evidence-based refactoring while preserving intended behavior, minimizing unnecessary changes, and preventing complexity or duplication from accumulating.
------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------

# Code Refactoring & Hygiene

Use this skill when modifying or improving existing code. Prioritize correctness, clarity, maintainability, and minimal change over stylistic perfection.

## Core Principles

1. **Understand before refactoring**

   * Inspect the surrounding code, callers, dependencies, tests, and relevant project conventions.
   * Identify the actual problem before changing structure.
   * Do not refactor code merely because it differs from a preferred style.

2. **Preserve behavior**

   * Maintain existing behavior unless a behavior change is explicitly requested.
   * Preserve public interfaces and contracts where possible.
   * Treat externally consumed behavior as a compatibility boundary.

3. **Refactor based on complexity, not arbitrary thresholds**

   * Refactor when code has excessive responsibility, duplication, coupling, nesting, unclear naming, difficult testing, or other concrete maintainability problems.
   * Do not split functions, classes, or modules solely because they exceed an arbitrary line count.

4. **Prefer the smallest useful abstraction**

   * Extract reusable logic when duplication or complexity justifies it.
   * Avoid creating abstractions for hypothetical future requirements.
   * Do not introduce layers, patterns, wrappers, helpers, or services without a concrete benefit.

5. **Reduce duplication carefully**

   * Consolidate genuinely identical or strongly related behavior.
   * Do not force unrelated concepts into a shared abstraction merely because they look similar.

6. **Improve data access**

   * Identify N+1 queries, unnecessary queries, inefficient loops, missing indexes, excessive payloads, and avoidable repeated work.
   * Optimize based on evidence and expected impact.
   * Do not sacrifice clarity for speculative micro-optimizations.

7. **Control side effects**

   * Keep functions focused and predictable.
   * Prefer explicit data flow over hidden global state or surprising mutations.
   * Preserve necessary side effects when they are part of the existing contract.

8. **Remove dead code**

   * Remove unused code only when its unused status is reasonably established.
   * Do not delete seemingly unused public APIs, configuration, routes, or extension points without checking their consumers.

9. **Verify after refactoring**

   * Run relevant tests, static analysis, linting, type checks, or other project validation available.
   * Compare behavior before and after when practical.
   * Do not combine unrelated cleanup with a focused refactor unless explicitly requested.

## AI Slop Prevention

* Do not rewrite working code without a concrete reason.
* Do not create abstractions merely to appear architecturally sophisticated.
* Do not rename large portions of a codebase without need.
* Do not introduce dependencies for problems that can be solved simply with existing project capabilities.
* Do not mix unrelated refactoring into feature work or bug fixes.
* Prefer a small, understandable change over a large "clean rewrite."
