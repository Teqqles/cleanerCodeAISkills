# cleanAI

Claude Code skills for writing clean, testable, maintainable code. Language-agnostic: drop them into any project on any stack.

---

## Skills

| Skill | What it does |
|---|---|
| [`clean-code`](#clean-code) | Clarity over cleverness, descriptive names, early returns, no duplication |
| [`code-style`](#code-style) | Formatting, naming conventions, objective language, security at boundaries |
| [`git-workflow`](#git-workflow) | Feature branches, Conventional Commits, no co-authors, safety rules |
| [`dependency-injection`](#dependency-injection) | Inject collaborators, depend on abstractions, composition at the boundary |
| [`test-driven-development`](#test-driven-development) | Red-green-refactor, test-first, deterministic isolated tests |
| [`quality-testing`](#quality-testing) | RIGHTBICEP heuristic for coverage across behaviour, boundaries, and failure |
| [`maintainable-architecture`](#maintainable-architecture) | Separate domain from infrastructure, composition over inheritance |
| [`refactoring`](#refactoring) | Small reversible steps, remove dead code, rename, flatten |
| [`api-design`](#api-design) | Predictable, minimal, explicit, versioned, mockable interfaces |
| [`rigor`](#rigor) | Explicit errors, failure-first design, observability, production-readiness |
| [`highlander-principle`](#highlander-principle) | One author per commit, no co-authors, no AI attribution |

---

## Installation

### Option 1: Copy into an existing project

Copy skill directories from this repo's `skills/` folder into your project's `.claude/skills/`:

```
your-project/
└── .claude/
    └── skills/
        ├── clean-code/
        │   └── SKILL.md
        ├── git-workflow/
        │   └── SKILL.md
        └── ...
```

Claude Code loads skills from `.claude/skills/` automatically.

### Option 2: Install globally

Copy skills into your user-level Claude directory to use them in every project.

**macOS / Linux:**
```bash
mkdir -p ~/.claude/skills
cp -r skills/clean-code ~/.claude/skills/
cp -r skills/git-workflow ~/.claude/skills/
# repeat for each skill
```

**Windows:**
```powershell
mkdir -Force "$env:USERPROFILE\.claude\skills"
Copy-Item -Recurse skills\clean-code "$env:USERPROFILE\.claude\skills\"
Copy-Item -Recurse skills\git-workflow "$env:USERPROFILE\.claude\skills\"
# repeat for each skill
```

### Option 3: Clone and copy all skills at once

```bash
git clone https://github.com/Teqqles/cleanerCodeAISkills
cp -r cleanerCodeAISkills/skills/. ~/.claude/skills/
```

Or as a submodule:

```bash
git submodule add https://github.com/Teqqles/cleanerCodeAISkills tools/cleanAI
ln -s tools/cleanAI/skills .claude/skills
```

---

## Usage

### Explicit invocation

Type `/` followed by the skill name:

```
/clean-code
/git-workflow
/quality-testing
```

### Automatic triggering

Each skill description tells Claude when to apply it:

- Writing a new class triggers `dependency-injection` and `clean-code`
- Running `git commit` triggers `git-workflow`
- Writing tests triggers `quality-testing` and `test-driven-development`
- Reviewing an interface triggers `api-design`

Skills trigger without being asked when the context matches.

### Using skills together

A typical feature cycle:

1. `test-driven-development`: write a failing test first
2. `clean-code`: implement with clarity and small functions
3. `dependency-injection`: inject all collaborators
4. `quality-testing`: verify RIGHTBICEP coverage
5. `refactoring`: clean up once green
6. `git-workflow`: commit on a feature branch with a Conventional Commit message

---

## Skill Reference

### clean-code

- Obvious solutions over clever ones
- Names that reveal intent at the call site
- Small, single-purpose functions with no hidden side effects
- Early returns to flatten nesting
- Duplication removed through meaningful abstraction
- Domain-centric module structure
- Comments only when the why cannot be expressed through code

**Triggers:** Writing any function, class, or module. Any mention of readability, simplicity, or naming.

---

### code-style

- Run the project formatter before committing (Prettier, Black, gofmt, etc.)
- Wrap types named after what they wrap: `<OriginalName><Purpose>`
- No comments that explain what the code does
- No emojis in code, commits, or documentation
- Concrete language: "adds retry with exponential backoff", not "robust retry support"
- Validate at system boundaries only

**Triggers:** Any code output. Naming, formatting, or documentation questions.

---

### git-workflow

- Feature branches for every change. Never push to `main` directly.
- Single-author commits, no co-author attribution
- Conventional Commits: `feat:`, `fix:`, `docs:`, `test:`, `refactor:`, `chore:`
- Commit messages explain why, not what
- PRs: small, focused, CI passes, squash merge
- No force-push, no `--no-verify`, new commits not amends

**Triggers:** Any git operation: committing, branching, PRs, pushing.

---

### dependency-injection

- Classes receive collaborators; they do not construct them
- Depend on interfaces, not concrete types
- Constructor injection by default
- No static state, global singletons, or service locators in core logic
- Infrastructure depends on the domain, never the reverse
- Every component replaceable with a fake for testing
- Object graph wired at the application boundary only

**Triggers:** Writing any class or service with collaborators. Mentions of testability, coupling, or "hard to test".

---

### test-driven-development

- Write a failing test before any production code
- Implement only the minimum to pass the test
- Refactor with all tests green
- Tests are deterministic, isolated, and fast
- Mock only stable interfaces; prefer hand-written fakes
- Test names describe behaviour: `throws_when_order_is_empty`, not `testOrder`
- The suite must be trusted enough to refactor aggressively

**Triggers:** Implementing any new function, class, or feature. Mentions of "write tests", "test first", or "TDD".

---

### quality-testing

| Dimension | What to test |
|---|---|
| **R**: Right | Correct output for valid inputs |
| **I**: Inverse | Roundtrip operations undo each other |
| **G**: Good | Typical, expected use cases |
| **H**: Harmful | Invalid, malicious, and adversarial inputs |
| **T**: Time | Time-dependent behaviour via a fake clock |
| **B**: Boundaries | Limits, edges, and state transitions |
| **I**: Interfaces | External system interactions via fakes |
| **C**: Cross-checks | Results validated by an alternative mechanism |
| **E**: Error conditions | Graceful handling of all failure modes |
| **P**: Performance | Efficiency under load with concrete thresholds |

Zero test failures, zero warnings. Fix pre-existing failures; don't defer them.

**Triggers:** Writing or reviewing tests. Coverage questions. After any code change.

---

### maintainable-architecture

- Domain logic in its own layer, no framework imports
- No infrastructure types in core logic
- Business rules testable without starting the framework
- Small, cohesive, loosely coupled modules
- Add behaviour by extending, not by modifying existing code
- Composition over inheritance
- Hard to test means wrong boundaries; fix the structure

**Triggers:** Designing a system or feature. Questions about structure, where code belongs, or how to separate concerns.

---

### refactoring

- Leave code cleaner than you found it, every time you touch it
- One small step per commit, tests green throughout
- Delete dead code immediately, no commenting out
- Extract complex conditions into named functions
- Replace magic values with named constants
- Rename anything whose name no longer matches what it does
- Refactoring commits contain no behaviour changes

**Triggers:** Requests to clean up, simplify, rename, or improve existing code. Spotting magic values, deep nesting, or misleading names.

---

### api-design

- Consistent behaviour: similar operations work the same way
- Expose only what callers need
- No hidden global state or ambient context
- API vocabulary matches the caller's domain, not the implementation
- Breaking changes get a version bump or a deprecation window
- Working examples are the primary documentation
- Every boundary interface mockable with a small fake

**Triggers:** Designing a public API, library interface, HTTP endpoint, or module boundary. Any interface review.

---

### rigor

- Unvalidated assumptions become bugs; test or encode them in the type system
- Every failure path is explicit and predictable
- External calls (network, disk, database) have retry, fallback, or circuit-breaker logic
- Choose correct algorithms from the start; optimise only when you measure a problem
- Production code ships with structured logging, health checks, and environment-driven config
- Errors diagnosable from logs alone, without reading the source
- Every commit makes the code clearer, more correct, or more capable

**Triggers:** Writing production code, error handling, external service calls. Mentions of reliability, observability, or "production-ready".

---

### highlander-principle

There can be only one. Every commit has a single author: the human who owns the repository.

- No `Co-Authored-By:` lines
- No AI tool attribution in commits
- No secondary authors unless the user explicitly requests it
- The commit author is the person accountable for the change

**Triggers:** Any commit, commit message, or git workflow. Applies to every project automatically.

---

## Contributing

Each skill lives in its own directory under `skills/`:

```
skills/
└── skill-name/
    └── SKILL.md
```

`SKILL.md` needs YAML frontmatter with `name` and `description`. The description controls when Claude triggers the skill. Be specific about contexts and include example phrases a user might type.

When adding a skill:
1. Create a directory under `skills/`
2. Write `SKILL.md` with frontmatter and instructions
3. Add a row to the table above
4. Add a reference entry below

See the [Anthropic skills creator documentation](https://github.com/anthropics/skills/blob/main/skills/skill-creator/SKILL.md) for the full authoring guide.

---

## Licence

MIT
