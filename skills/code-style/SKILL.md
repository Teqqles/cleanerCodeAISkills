---
name: code-style
description: Enforces consistent code style conventions: naming, formatting, comments, language tone, and security hygiene. Use this skill whenever writing or reviewing code in any language. Trigger when the user mentions naming conventions, formatting, comments, commit messages, documentation style, or says "keep it consistent", "clean this up", or "follow our conventions". Apply these rules by default on all code output.
---

# Code Style Conventions

Language-agnostic style rules. Apply to any codebase regardless of language or framework.

## Follow the Project's Formatter

Format code using the project's configured formatter before committing.

Common formatters: Prettier (JS/TS), Black (Python), gofmt (Go), dotnet format (C#), dart format (Dart), rustfmt (Rust).

- Run the formatter before every commit
- Verify the output matches the surrounding code style
- Do not commit code that fails the formatter

## Descriptive Naming

Names communicate intent without a comment to explain them.

- Variables, functions, and types read like sentences at the call site
- Wrapper or adapter types: name them after what they wrap plus their purpose
  - Pattern: `<OriginalName><Purpose>`: e.g., `WindowsRegistryService`, `InMemoryEventStore`
  - Avoid generic names like `Helper`, `Manager`, `Util` unless truly general
- Avoid abbreviations that are not universally understood in the domain
- Longer, unambiguous names beat short, clever ones

## Comments Are a Code Smell

Comments that explain *what* the code does indicate the code is unclear. Rewrite the code instead.

- Write self-documenting code first
- Add a comment only when the **why** cannot be expressed through code
- Never describe what the code does: rename or restructure instead
- Remove comments referencing the current task or fix ("added for X", "used by Y")
- Public API documentation (`///`, JSDoc, docstrings) is acceptable for public-facing symbols

## No Emojis

- Never use emojis in code comments, commit messages, or documentation
- Use markdown structure (headers, lists, code blocks) for organisation
- Exception: only if the user explicitly requests it

## Objective Language

Subjective qualifiers belong in marketing copy, not technical writing.

- Describe what was implemented, not how good it is
- Avoid: "comprehensive", "robust", "world-class", "production-ready", "enterprise-grade", "professional"
- Use concrete facts: "adds retry logic with exponential backoff", not "robust retry support"
- Commit messages, PR descriptions, and documentation state capabilities: they do not praise them

## Language Examples

When the project language is known, read the relevant reference for formatter commands, naming conventions, and documentation style:

- `references/typescript.md`
- `references/python.md`
- `references/java.md`
- `references/scala.md`
- `references/javascript.md`

## Security at Boundaries Only

- Validate at system boundaries: user input, external APIs, file system reads, environment variables
- Trust internal code and framework guarantees: no defensive checks inside already-validated paths
- No injection vulnerabilities (command injection, SQL injection, XSS)
- Never store secrets in code or version control
