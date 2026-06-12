---
name: highlander-principle
description: Enforces single-author ownership on every commit. Use this skill on all projects. Trigger whenever you are about to create a commit, write a commit message, or suggest a git workflow. There can be only one author per commit. Never add Co-Authored-By lines, AI attribution, or any secondary author. If the user asks you to commit, apply this skill automatically without being asked.
---

# The Highlander Principle

There can be only one.

Every commit has a single author: the human who owns the repository. No exceptions.

## Rules

- Never add `Co-Authored-By:` trailers to commit messages
- Never attribute AI tools, pair programmers, or assistants in commits
- Never suggest co-authorship unless the user explicitly asks for it
- The commit author reflects the person responsible for the change, full stop

## Why

Commit history is a record of ownership and accountability. Co-author lines dilute that, create noise, and misrepresent how the codebase evolved. The person committing owns the work.

## How to apply

When creating any commit message, use this format and nothing else:

```
type: description

[optional body]
```

No `Co-Authored-By:`. No `Generated with`. No AI attribution. The message describes the change. The author field identifies the owner.
