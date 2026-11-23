---

name: conventional-commits
description: Creates concise Git commit messages following the Conventional Commits specification and accurately describing the change represented by the commit.
-----------------------------------------------------------------------------------------------------------------------------------------------------------------

# Conventional Commits

Use Conventional Commits when creating or suggesting Git commit messages.

## Format

```text
<type>[optional scope]: <description>
```

Breaking changes may be indicated using `!`:

```text
feat!: change authentication flow
feat(api)!: remove deprecated endpoint
```

## Common Types

* `feat` — introduces a new feature
* `fix` — fixes incorrect behavior
* `refactor` — changes internal structure without intentionally changing behavior
* `perf` — improves performance
* `test` — adds or modifies tests
* `docs` — documentation changes
* `style` — formatting or non-functional style changes
* `build` — build system or dependency changes
* `ci` — continuous integration changes
* `chore` — maintenance that does not fit another type

Use the project's established commit conventions when they are explicitly defined.

## Rules

1. Use a concise, meaningful description.
2. Start the description with a lowercase imperative verb where practical.
3. Do not add a period to the subject.
4. Choose the type based on the primary purpose of the change.
5. Use a scope only when it improves clarity.
6. Do not claim a feature, fix, or behavior change that the commit does not actually contain.
7. Keep one logical change per commit when practical.
8. If a commit contains multiple related changes, choose the type representing its primary purpose.
9. For breaking changes, clearly indicate the breaking nature using the Conventional Commits format and explain the impact in the commit body when appropriate.

## AI Behavior

* Inspect the actual changes before proposing a commit message.
* Never invent changes that were not made.
* Prefer a concise subject.
* If the project has existing commit conventions, follow them unless they conflict with an explicit user requirement.
