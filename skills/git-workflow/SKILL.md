---
name: git-workflow
description: Applies consistent Git workflow conventions: branching, commit format, PR guidelines, and safety rules. Use this skill whenever the user asks you to commit code, create a branch, write a commit message, open a PR, or push changes. Also trigger when the user says "check this in", "save my work", "create a pull request", or asks about branching strategy. Follow these conventions on every project using these skills unless the user explicitly overrides a specific rule.
---

# Git Workflow Conventions

Language-agnostic Git conventions. Apply to all projects regardless of stack.

## Always Use Feature Branches

Never push directly to `main`. Code review prevents unreviewed code from reaching production: bypassing it defeats the purpose.

- Create a feature branch for every change: `git checkout -b <type>/<short-description>`
- Branch naming:
  - `feat/`: new features
  - `fix/`: bug fixes
  - `docs/`: documentation
  - `test/`: test additions or updates
  - `refactor/`: code restructuring without functional change
  - `chore/`: maintenance (deps, build config, tooling)
- Push to origin: `git push -u origin <branch-name>`
- Create PR from feature branch to `main`
- Return to `main` after pushing

## No Co-Author Attribution

Commits reflect the human author only.

- Commit format: `git commit -m "type: description"`
- Never include `Co-Authored-By:` lines
- Never include AI model attribution in commits

## Conventional Commits Format

All commits follow [Conventional Commits](https://www.conventionalcommits.org/):

```
<type>: <description>

[optional body]
```

**Types:** `feat`, `fix`, `docs`, `test`, `refactor`, `chore`

**Examples:**
```
feat: add keyboard shortcut for primary action
fix: prevent crash when config file is missing
docs: add setup instructions to README
test: cover error path in data loader
refactor: extract order validation into dedicated class
```

## Commit Message Quality

The diff shows *what* changed. The message explains *why*.

- Subject line under 70 characters
- Imperative mood: "add feature" not "added feature"
- Describe the reason, not the diff: "fix crash on missing config" not "updated ConfigLoader.cs"
- No subjective qualifiers: facts only
- Reference issues when applicable: `fix: crash on missing config (#45)`

## PR Guidelines

- Keep PRs small and focused on one concern
- Describe **why** the change is needed, not just what it does
- Include tests for any logic changes
- CI must pass before merge
- Squash merge for clean history

## Git Safety Protocol

These commands are destructive or hard to reverse. Never run them without explicit user confirmation:

- `git push --force`
- `git reset --hard`
- `git clean -fd`
- `git checkout .`

Never bypass hooks with `--no-verify` or `--no-gpg-sign`. When a hook fails, fix the root cause.

Always create **new commits** rather than amending, unless the user explicitly requests it. If a hook fails, the commit did not happen: fix the issue and commit fresh.
