# Contributing

Contributions are welcome: new skills, improvements to existing ones, additional language examples, and bug fixes.

## What belongs here

Skills in this repo are:

- **Language-agnostic**: the core principles apply to any stack. Language-specific tooling (e.g. `flutter analyze`, `dotnet test`) belongs in your own project's skill files.
- **Example-grounded**: abstract principles are supported by concrete code examples in `references/`.
- **Honest**: examples compile and make sense. Bad/good pairs demonstrate a real difference, not a strawman.

## Adding a new skill

1. Fork the repo and create a branch: `git checkout -b feat/skill-name`
2. Create a directory under `skills/`: `skills/your-skill-name/`
3. Write `SKILL.md`: see the structure below
4. Add language reference files under `skills/your-skill-name/references/` for any languages you can verify
5. Add a row to the skills table in `README.md`
6. Add a reference section at the bottom of `README.md`
7. Open a pull request

## Improving an existing skill

- Corrections to examples, clearer wording, additional language references, and bug fixes are all welcome
- Keep changes focused: one concern per PR
- If you're adding a language reference that doesn't exist yet, follow the pattern of an existing one for that skill

## SKILL.md structure

```
---
name: skill-name
description: One or two sentences. State what the skill does AND when to trigger it.
             Include example user phrases. Be specific enough to trigger in the right
             contexts without over-triggering.
---

# Skill Title

Brief framing sentence.

## Section

Explanation of the principle and how to apply it.

## Language Examples

When the project language is known, read the relevant reference:

- `references/typescript.md`
- `references/python.md`
- ...
```

The `description` field is the primary trigger mechanism: Claude reads it to decide whether to load the skill. Make it specific and include real phrases a user might type.

## Language reference files

Each reference file lives at `skills/<skill-name>/references/<language>.md` and covers the same concepts as the SKILL.md but with idiomatic, language-specific examples.

- Examples must be correct and compile (or be obviously illustrative pseudo-code, clearly labelled)
- Bad/good pairs must demonstrate a real difference: not a strawman
- Use the language's standard idioms: e.g. `Protocol` for Python interfaces, `trait` for Scala, `record` for Java domain types
- Keep examples short enough to read at a glance

## Commit style

Follow [Conventional Commits](https://www.conventionalcommits.org/):

```
feat: add observability skill
fix: correct Scala Order constructor in quality-testing examples
docs: add C# language references for dependency-injection
```

## Code of conduct

Be direct and constructive. Feedback on examples should be specific: point to the line and explain what is wrong and why. Personal criticism is not welcome.
