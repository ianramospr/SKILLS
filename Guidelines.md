# `SKILL.md` Development Guidelines

Authoring and review standards for an Agent Skill.

---

## 1. One coherent responsibility

A skill should represent one reusable capability or workflow—like a well-scoped function.

**Good scopes:**

- Review a pull request according to company conventions
- Configure iOS Universal Links
- Generate and validate database migrations
- Convert product requirements into Linear issues

**Avoid skills that are:**

- So broad they cover an entire engineering discipline
- So narrow they solve only one exact prompt
- Mostly personality instructions or generic advice
- Duplicating behavior the agent already performs reliably

The most effective skills come from real tasks, corrections, runbooks, code reviews, and project conventions—not generic "best practices."

---

## 2. Directory structure

```text
skill-name/
├── SKILL.md
├── scripts/      # Optional executable tools
├── references/   # Optional documentation
└── assets/       # Optional templates, schemas, examples
```

Only `SKILL.md` is required. Add scripts, references, and assets only when they improve reliability or reduce repeated work.

---

## 3. YAML frontmatter

Every `SKILL.md` must begin with:

```yaml
---
name: example-skill
description: Use this skill when...
---
```

### `name`

- 1–64 characters
- Lowercase letters, numbers, and hyphens only
- Must match the parent directory name
- Cannot start or end with a hyphen, or contain consecutive hyphens

```yaml
# Good
name: ios-universal-links

# Invalid
name: IOS-Universal-Links
name: -universal-links
name: universal--links
```

### `description`

- Non-empty, under 1,024 characters
- Explains what the skill does **and** when it activates
- Includes relevant trigger terms
- Establishes boundaries to prevent activation for unrelated tasks

> The agent sees only the skill's name, description, and path before deciding whether to load it. Frontmatter quality directly determines activation accuracy.

---

## 4. Write the description as a trigger

```yaml
# Good
description: Use this skill when implementing, debugging, or reviewing iOS Universal Links,
  including associated domains, AASA files, routing, entitlements, and end-to-end link
  verification. Do not use it for ordinary in-app navigation.

# Weak
description: Helps with iOS links.
```

A strong description:

- Starts with the primary use case
- Describes the user's intent, not the internal implementation
- Covers indirect ways the task may be expressed
- Names important exclusions
- Uses a few sentences, not a long catalog

Place the most important trigger language first — descriptions may be truncated when many skills are installed.

---

## 5. Body structure

Recommended sections:

```markdown
## Purpose

## When to use

## When not to use

## Required context

## Workflow

## Decision rules

## Validation

## Definition of done

## Gotchas

## Available resources
```

Not every skill needs every section. The body should let the agent determine:

1. What it is trying to accomplish
2. What information it needs
3. What sequence to follow
4. Where judgment is allowed
5. How to verify the result
6. When the task is complete

---

## 6. Write procedures, not declarations

**Weak:**

```markdown
Follow security best practices.
Handle errors appropriately.
```

**Strong:**

```markdown
1. Identify every externally accessible endpoint changed by the task.
2. Verify authentication before reading or modifying user-owned data.
3. Confirm database queries use parameterized values.
4. Check that errors do not expose stack traces, tokens, or internal identifiers.
5. Run the repository security tests before finalizing.
```

Instructions should tell the agent what to inspect, what action to take, and how to determine success.

---

## 7. Give defaults, not menus

**Weak:**

```markdown
You may use npm, pnpm, Yarn, or Bun.
```

**Better:**

```markdown
Use the package manager indicated by the existing lockfile:

- `pnpm-lock.yaml` → pnpm
- `yarn.lock` → Yarn
- `bun.lock` → Bun
- Otherwise → npm
```

A skill should reduce ambiguity, not transfer every decision back to the agent.

---

## 8. Match strictness to risk

Be flexible where multiple solutions are valid:

```markdown
Choose a component boundary that keeps state ownership clear and avoids unnecessary prop drilling.
```

Be exact where order or consistency matters:

```markdown
Run the migration in this order:

1. Create a backup.
2. Run in dry-run mode.
3. Validate the generated SQL.
4. Apply the migration.
5. Run integrity checks.

Do not apply if dry-run validation fails.
```

Fragile, destructive, or compliance-sensitive operations require more prescriptive instructions.

---

## 9. Include validation and a definition of done

```markdown
## Validation

1. Run the relevant unit tests.
2. Run the type checker.
3. Run the linter or formatter.
4. Exercise the changed flow manually when feasible.
5. Fix all failures introduced by the change.
6. Repeat until validation passes.

## Definition of done

- The requested behavior is implemented.
- Relevant tests pass.
- No unrelated files were changed.
- Documentation is updated when behavior or setup changed.
- Remaining limitations are stated explicitly.
```

Validation loops make skills substantially more reliable than one-pass instructions.

---

## 10. Record non-obvious gotchas

A `Gotchas` section should contain facts a capable agent would reasonably get wrong without project-specific knowledge.

```markdown
## Gotchas

- The production app uses `AssociatedDomains.entitlements`; do not edit the generated file under `DerivedData`.
- The AASA file must be served without redirects.
- The staging bundle identifier differs from production.
- The `/health` endpoint does not verify database connectivity; use `/ready`.
```

When the agent makes a recurring mistake, add the correction here.

---

## 11. Keep `SKILL.md` focused

Target limits:

- Under 500 lines
- Under ~5,000 tokens

Move detailed material into reference files:

```text
references/
├── architecture.md
├── api-contracts.md
├── troubleshooting.md
└── examples.md
```

Tell the agent exactly when to read each file:

```markdown
Read `references/troubleshooting.md` only when link verification fails.
Read `references/api-contracts.md` before changing the routing API.
```

Avoid instructions like "read everything in `references/`." Progressive disclosure keeps irrelevant material out of the context window.

---

## 12. Bundle scripts for deterministic logic

Use `scripts/` when the agent would otherwise repeatedly recreate complex or error-prone logic.

**Good candidates:** validators, parsers, code generators, environment inspections, format conversions, deployment/migration checks.

Reference scripts with paths relative to the skill root:

````markdown
Run:

```bash
python3 scripts/validate_aasa.py --url "$AASA_URL"
```
````

````

Scripts should:
- Be non-interactive
- Accept input via flags, environment variables, or stdin
- Support `--help` and produce helpful errors
- Use structured output (stdout for data, stderr for diagnostics)
- Be idempotent where possible
- Support `--dry-run` for destructive operations
- Use meaningful exit codes
- Pin dependency versions when reproducibility matters

---

## 13. Include templates for consistent output

When output must follow a specific structure, show it rather than describing it abstractly.

```markdown
## Final report format

# Summary
[What was implemented]

## Files changed
- `path/to/file`: [reason]

## Validation performed
- [Command or check]: [result]

## Remaining limitations
- [Limitation, or "None"]
````

Long templates belong in `assets/`; short ones can stay in `SKILL.md`.

---

## 14. State prerequisites explicitly

Use the optional `compatibility` field when the skill has environmental requirements:

```yaml
---
name: docker-release
description: Use this skill when building and publishing this project's Docker release images.
compatibility: Requires Docker, git, jq, network access, and container registry credentials.
---
```

Do not silently assume a tool, OS, credential, or network connection is available.

---

## 15. Design for safe failure

```markdown
## Missing prerequisites

- If no lockfile exists, use npm.
- If required credentials are unavailable, stop before making remote changes and report what is missing.
- If the repository has uncommitted changes, preserve them and avoid destructive resets.
- If validation cannot run, explain why and identify the unverified behavior.
```

Do not tell the agent to guess when ambiguity could produce destructive or incorrect results.

---

## 16. Test both activation and execution

Test with prompts that:

- Clearly should activate the skill
- Describe the task without using its exact terminology
- Include typos or casual phrasing
- Embed the task inside a larger workflow
- Share keywords but should **not** activate the skill
- Exercise missing prerequisites and failure cases

A useful starting set: **8–10 should-trigger** and **8–10 should-not-trigger** prompts, run multiple times (activation can vary).

Also inspect the execution trace:

- Did the skill activate?
- Did the agent follow the intended sequence?
- Did it run the expected tools and validate the result?
- Did it create unnecessary files or perform unnecessary work?

---

## 17. Iterate from observed failures

After each real use:

1. Identify what the agent misunderstood or skipped.
2. Determine whether the problem came from the description, missing instructions, ambiguous defaults, missing project knowledge, or a fragile manual operation that should become a script.
3. Make the smallest reusable correction.
4. Add a regression test for the failure.
5. Remove instructions that no longer add value.

A skill should evolve from evidence, not accumulate rules indefinitely.

---

# Starter template

````markdown
---
name: skill-name
description: Use this skill when [primary intent and trigger language]. It handles [core scope]. Do not use it for [important exclusions].
---

# Purpose

[The reusable capability this skill provides.]

## When to use

- [Trigger condition]
- [Indirectly expressed trigger]

## When not to use

- [Adjacent but excluded workflow]
- [Task the agent handles without this skill]

## Required context

Before starting, identify:

- [Required files, inputs, or repository state]
- [Required tools or credentials]
- [Project conventions to inspect]

If critical information is unavailable, follow the safest non-destructive path and state what is missing.

## Workflow

1. Inspect the existing implementation and relevant project conventions.
2. Determine the smallest valid change.
3. Perform the change using established conventions.
4. Run validation.
5. Fix any failures introduced by the change.
6. Summarize the result.

## Decision rules

- Use [default approach] unless [specific exception].
- Preserve [important invariant].
- Do not [prohibited action].
- For destructive changes, use a dry run or preview first.

## Validation

```bash
[validation command]
```

If validation fails:

1. Read the complete error.
2. Fix the underlying problem.
3. Run validation again.
4. Do not finalize until validation passes or the blocker is clearly documented.

## Definition of done

- [Expected artifact or behavior exists]
- [Tests or validators pass]
- [No unrelated changes introduced]
- [Documentation updated when required]

## Gotchas

- [Non-obvious project fact]
- [Recurring failure mode]

## Available resources

- `references/example.md` — Read when [specific condition].
- `assets/template.md` — Use when producing [specific output].
- `scripts/validate.py` — Run to validate [specific artifact].
````

---

**Core principle:** A good `SKILL.md` makes a specialized workflow more reliable without unnecessarily restricting the agent or filling its context with information it already knows.
