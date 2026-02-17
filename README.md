<div align="center">

# GET SHIT DONE

**A battle-tested fork of [GSD](https://github.com/glittercowboy/get-shit-done) — meta-prompting, context engineering and spec-driven development for Claude Code, OpenCode, and Gemini CLI.**

**Solves context rot — the quality degradation that happens as Claude fills its context window.**

[![GitHub stars](https://img.shields.io/github/stars/smledbetter/get-shit-done?style=for-the-badge&logo=github&color=181717)](https://github.com/smledbetter/get-shit-done)
[![Upstream](https://img.shields.io/badge/upstream-glittercowboy%2Fget--shit--done-blue?style=for-the-badge&logo=github)](https://github.com/glittercowboy/get-shit-done)
[![License](https://img.shields.io/badge/license-MIT-blue?style=for-the-badge)](LICENSE)

<br>

```bash
git clone https://github.com/smledbetter/get-shit-done.git
cd get-shit-done
node bin/install.js
```

**Works on Mac, Windows, and Linux.**

<br>

![GSD Install](assets/terminal.svg)

<br>

[About GSD](#about-gsd) · [What This Fork Adds](#what-this-fork-adds) · [How It Works](#how-it-works) · [Commands](#commands) · [Why It Works](#why-it-works) · [User Guide](docs/USER-GUIDE.md)

</div>

---

## About GSD

GSD was created by [TÂCHES](https://github.com/glittercowboy). In their words:

> I'm a solo developer. I don't write code — Claude Code does.
>
> Other spec-driven development tools exist; BMAD, Speckit... But they all seem to make things way more complicated than they need to be. I'm not a 50-person software company. I don't want to play enterprise theater. I'm just a creative person trying to build great things that work.
>
> So I built GSD. The complexity is in the system, not in your workflow. No enterprise roleplay bullshit. Just an incredibly effective system for building cool stuff consistently using Claude Code.

The original GSD is available at [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) and via `npx get-shit-done-cc@latest`.

## Why This Fork

I used GSD to build [Weaveto.do](https://github.com/smledbetter/Weaveto.do) — 12,000 lines of TypeScript, 491 tests, real end-to-end encryption. GSD's core workflow is excellent. But shipping a real product with it exposed friction points that the standard workflow doesn't address yet. This fork adds the features I needed.

---

## What This Fork Adds

The full story: [How I Let Robots Build My Encryption App](https://medium.com/@stevo_actually/how-i-let-robots-build-my-encryption-app-f0a3a39e16d8).

The discuss phase was too slow when you already had a PRD. Manual verification was redundant when you had a real test suite. Four separate phases with 8+ agent spawns was overkill when the work was well-understood.

The lessons from that build:

| What we learned | What GSD does now |
|-----------------|-------------------|
| A PRD makes interactive discussion redundant | **PRD Express Path** — `/gsd:plan-phase --prd spec.md` skips discuss-phase entirely |
| Real test suites beat manual verification | **Ship-Readiness Gate** — run `npm test`, `tsc`, linters as automated quality gates |
| Specialized agent roles produce better output | **Skill System** — PM, UX, Security, and Production Engineer advisory perspectives |
| Agents need type contracts, not a codebase to explore | **Interface-First Planning** — planner extracts interfaces so executors get blueprints |
| Past mistakes should inform future planning | **Living Retrospective** — what worked/didn't feeds into the next milestone's planner |
| 4 phases × 2+ agents each is often too many | **Consolidated Workflow** — 3 phases, ~50% fewer agent spawns, same quality |
| Running phases one at a time is tedious | **Sprint** — `/gsd:sprint` runs all remaining phases unattended with failure recovery |

These aren't theoretical improvements. They came from shipping a real product with GSD and fixing the things that slowed us down. The core insight from the article: **the bottleneck isn't AI capability — it's coordination.** Every feature above reduces coordination overhead.

---

## Who This Is For

People who want to describe what they want and have it built correctly — without pretending they're running a 50-person engineering org.

---

## Getting Started

This is a fork — install from the repo, not npm.

```bash
git clone https://github.com/smledbetter/get-shit-done.git
cd get-shit-done
node bin/install.js
```

The installer prompts you to choose:
1. **Runtime** — Claude Code, OpenCode, Gemini, or all
2. **Location** — Global (all projects) or local (current project only)

Verify with `/gsd:help` inside your chosen runtime.

### Staying Updated

Pull the latest changes and re-run the installer:

```bash
cd get-shit-done
git pull
node bin/install.js
```

> **Want upstream GSD instead?** The original is available via `npx get-shit-done-cc@latest` from the [upstream repo](https://github.com/glittercowboy/get-shit-done). This fork adds quality gates, PRD express path, skill system, consolidated workflow, and other features not yet in upstream.

<details>
<summary><strong>Non-interactive Install (Docker, CI, Scripts)</strong></summary>

```bash
# Claude Code
node bin/install.js --claude --global   # Install to ~/.claude/
node bin/install.js --claude --local    # Install to ./.claude/

# OpenCode (open source, free models)
node bin/install.js --opencode --global # Install to ~/.config/opencode/

# Gemini CLI
node bin/install.js --gemini --global   # Install to ~/.gemini/

# All runtimes
node bin/install.js --all --global      # Install to all directories
```

Use `--global` (`-g`) or `--local` (`-l`) to skip the location prompt.
Use `--claude`, `--opencode`, `--gemini`, or `--all` to skip the runtime prompt.

</details>

### Recommended: Skip Permissions Mode

GSD is designed for frictionless automation. Run Claude Code with:

```bash
claude --dangerously-skip-permissions
```

> [!TIP]
> This is how GSD is intended to be used — stopping to approve `date` and `git commit` 50 times defeats the purpose.

<details>
<summary><strong>Alternative: Granular Permissions</strong></summary>

If you prefer not to use that flag, add this to your project's `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "Bash(date:*)",
      "Bash(echo:*)",
      "Bash(cat:*)",
      "Bash(ls:*)",
      "Bash(mkdir:*)",
      "Bash(wc:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(sort:*)",
      "Bash(grep:*)",
      "Bash(tr:*)",
      "Bash(git add:*)",
      "Bash(git commit:*)",
      "Bash(git status:*)",
      "Bash(git log:*)",
      "Bash(git diff:*)",
      "Bash(git tag:*)"
    ]
  }
}
```

</details>

---

## How It Works

> **Already have code?** Run `/gsd:map-codebase` first. It spawns parallel agents to analyze your stack, architecture, conventions, and concerns. Then `/gsd:new-project` knows your codebase — questions focus on what you're adding, and planning automatically loads your patterns.

### 1. Initialize Project

```
/gsd:new-project
```

One command, one flow. The system:

1. **Questions** — Asks until it understands your idea completely (goals, constraints, tech preferences, edge cases)
2. **Research** — Spawns parallel agents to investigate the domain (optional but recommended)
3. **Requirements** — Extracts what's v1, v2, and out of scope
4. **Roadmap** — Creates phases mapped to requirements

You approve the roadmap. Now you're ready to build.

**Creates:** `PROJECT.md`, `REQUIREMENTS.md`, `ROADMAP.md`, `STATE.md`, `.planning/research/`

---

### 2. Discuss Phase

```
/gsd:discuss-phase 1
```

> **Already have a PRD?** Skip this step entirely with `/gsd:plan-phase 1 --prd spec.md`. The system generates CONTEXT.md from your document with all decisions locked. Straight to planning.

**This is where you shape the implementation.**

Your roadmap has a sentence or two per phase. That's not enough context to build something the way *you* imagine it. This step captures your preferences before anything gets researched or planned.

The system analyzes the phase and identifies gray areas based on what's being built:

- **Visual features** → Layout, density, interactions, empty states
- **APIs/CLIs** → Response format, flags, error handling, verbosity
- **Content systems** → Structure, tone, depth, flow
- **Organization tasks** → Grouping criteria, naming, duplicates, exceptions

For each area you select, it asks until you're satisfied. The output — `CONTEXT.md` — feeds directly into the next two steps:

1. **Researcher reads it** — Knows what patterns to investigate ("user wants card layout" → research card component libraries)
2. **Planner reads it** — Knows what decisions are locked ("infinite scroll decided" → plan includes scroll handling)

The deeper you go here, the more the system builds what you actually want. Skip it and you get reasonable defaults. Use it and you get *your* vision.

**Creates:** `{phase_num}-CONTEXT.md`

---

### 3. Plan Phase

```
/gsd:plan-phase 1
```

The system:

1. **Researches** — Investigates how to implement this phase, guided by your CONTEXT.md decisions
2. **Plans** — Creates 2-3 atomic task plans with XML structure
3. **Verifies** — Checks plans against requirements, loops until they pass

Each plan is small enough to execute in a fresh context window. No degradation, no "I'll be more concise now."

**Creates:** `{phase_num}-RESEARCH.md`, `{phase_num}-{N}-PLAN.md`

---

### 4. Execute Phase

```
/gsd:execute-phase 1
```

The system:

1. **Runs plans in waves** — Parallel where possible, sequential when dependent
2. **Fresh context per plan** — 200k tokens purely for implementation, zero accumulated garbage
3. **Commits per task** — Every task gets its own atomic commit
4. **Verifies against goals** — Checks the codebase delivers what the phase promised

Walk away, come back to completed work with clean git history.

If you've configured **quality gates** (test suites, type checkers, linters), they run automatically after verification. Failed gates block or warn depending on your settings — no manual checking required.

**How Wave Execution Works:**

Plans are grouped into "waves" based on dependencies. Within each wave, plans run in parallel. Waves run sequentially.

```
┌─────────────────────────────────────────────────────────────────────┐
│  PHASE EXECUTION                                                     │
├─────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  WAVE 1 (parallel)          WAVE 2 (parallel)          WAVE 3       │
│  ┌─────────┐ ┌─────────┐    ┌─────────┐ ┌─────────┐    ┌─────────┐ │
│  │ Plan 01 │ │ Plan 02 │ →  │ Plan 03 │ │ Plan 04 │ →  │ Plan 05 │ │
│  │         │ │         │    │         │ │         │    │         │ │
│  │ User    │ │ Product │    │ Orders  │ │ Cart    │    │ Checkout│ │
│  │ Model   │ │ Model   │    │ API     │ │ API     │    │ UI      │ │
│  └─────────┘ └─────────┘    └─────────┘ └─────────┘    └─────────┘ │
│       │           │              ↑           ↑              ↑       │
│       └───────────┴──────────────┴───────────┘              │       │
│              Dependencies: Plan 03 needs Plan 01            │       │
│                          Plan 04 needs Plan 02              │       │
│                          Plan 05 needs Plans 03 + 04        │       │
│                                                                      │
└─────────────────────────────────────────────────────────────────────┘
```

**Why waves matter:**
- Independent plans → Same wave → Run in parallel
- Dependent plans → Later wave → Wait for dependencies
- File conflicts → Sequential plans or same plan

This is why "vertical slices" (Plan 01: User feature end-to-end) parallelize better than "horizontal layers" (Plan 01: All models, Plan 02: All APIs).

**Creates:** `{phase_num}-{N}-SUMMARY.md`, `{phase_num}-VERIFICATION.md`

---

### 5. Verify Work

```
/gsd:verify-work 1
```

**This is where you confirm it actually works.**

Automated verification checks that code exists and tests pass. But does the feature *work* the way you expected? This is your chance to use it.

The system:

1. **Extracts testable deliverables** — What you should be able to do now
2. **Walks you through one at a time** — "Can you log in with email?" Yes/no, or describe what's wrong
3. **Diagnoses failures automatically** — Spawns debug agents to find root causes
4. **Creates verified fix plans** — Ready for immediate re-execution

If everything passes, you move on. If something's broken, you don't manually debug — you just run `/gsd:execute-phase` again with the fix plans it created.

**Creates:** `{phase_num}-UAT.md`, fix plans if issues found

---

### 6. Repeat → Complete → Next Milestone

```
/gsd:discuss-phase 2
/gsd:plan-phase 2
/gsd:execute-phase 2
/gsd:verify-work 2
...
/gsd:complete-milestone
/gsd:new-milestone
```

Loop **discuss → plan → execute → verify** until milestone complete. Or use `/gsd:consolidated-phase` for fewer steps.

Each phase gets your input (discuss), proper research (plan), clean execution (execute), and human verification (verify). Context stays fresh. Quality stays high. At milestone completion, a **living retrospective** captures what worked and what didn't — the planner reads this for your next milestone so you don't repeat the same mistakes.

When all phases are done, `/gsd:complete-milestone` archives the milestone and tags the release.

Then `/gsd:new-milestone` starts the next version — same flow as `new-project` but for your existing codebase. You describe what you want to build next, the system researches the domain, you scope requirements, and it creates a fresh roadmap. Each milestone is a clean cycle: define → build → ship.

---

### Quick Mode

```
/gsd:quick
```

**For ad-hoc tasks that don't need full planning.**

Quick mode gives you GSD guarantees (atomic commits, state tracking) with a faster path:

- **Same agents** — Planner + executor, same quality
- **Skips optional steps** — No research, no plan checker, no verifier
- **Separate tracking** — Lives in `.planning/quick/`, not phases

Use for: bug fixes, small features, config changes, one-off tasks.

```
/gsd:quick
> What do you want to do? "Add dark mode toggle to settings"
```

**Creates:** `.planning/quick/001-add-dark-mode-toggle/PLAN.md`, `SUMMARY.md`

---

### Consolidated Workflow

```
/gsd:consolidated-phase 1
```

**For teams that want fewer agent spawns without sacrificing quality.**

The standard workflow runs 4 phases with 8+ agent spawns per phase. The consolidated workflow collapses this to 3 phases with roughly half the agents:

| Phase | What happens | Agents |
|-------|-------------|--------|
| **Consensus + Plan** | One agent loads advisory skills (PM, UX, Security, Prod Eng) and planner role. Produces CONTEXT.md and PLAN.md, then self-verifies. | 1 |
| **Execute + Gate** | Standard wave-based execution, followed by a ship-readiness gate that combines verification and quality gates. | N + 1 |
| **Ship** | Orchestrator marks phase complete, commits metadata. | 0 |

Same quality gates. Same goal-backward verification. Fewer round-trips.

Enable via `/gsd:settings` → Workflow → Consolidated.

---

### Sprint

```
/gsd:sprint
```

**Run all remaining phases unattended.**

Sprint wraps GSD's existing auto-advance in a milestone-level orchestrator. One command, walk away, come back to a completed milestone (or a clear report of what failed).

```bash
/gsd:sprint                              # all remaining phases
/gsd:sprint 3-5                          # just phases 3 through 5
/gsd:sprint --dry-run                    # preview without executing
/gsd:sprint --skip-failures              # log failures, keep going
/gsd:sprint --consolidated --prd spec.md # fastest possible path
```

**What happens under the hood:**
1. Determines which phases to run from ROADMAP.md
2. Enables auto-advance temporarily
3. Chains discuss → plan → execute → verify for each phase
4. On verification gaps: automatically attempts gap closure (plan gaps → execute gaps → re-verify)
5. On failure: stops (default) or skips and continues (`--skip-failures`)
6. Writes `SPRINT-REPORT.md` with per-phase results
7. Restores original config

Sprint does not cross milestone boundaries. Configure default failure behavior via `/gsd:settings`.

**Creates:** `.planning/SPRINT-REPORT.md`

---

### Advisory Consensus

```
/gsd:advisory-consensus 1
```

**Get structured feedback from multiple expert perspectives in one pass.**

Instead of spawning separate agents for each advisory role, one agent loads all skill perspectives and produces a unified CONTEXT.md:

- **Product Manager** — User stories, acceptance criteria, prioritization
- **UX Designer** — Accessibility (WCAG 2.1 AA), interaction patterns, information hierarchy
- **Security Auditor** — OWASP Top 10, auth flows, data exposure, input validation
- **Production Engineer** — Quality gates, observability, deployment, performance budgets

The output feeds directly into `/gsd:plan-phase`. Custom skills can be added to `templates/skills/`.

---

## Why It Works

### Context Engineering

Claude Code is incredibly powerful *if* you give it the context it needs. Most people don't.

GSD handles it for you:

| File | What it does |
|------|--------------|
| `PROJECT.md` | Project vision, always loaded |
| `research/` | Ecosystem knowledge (stack, features, architecture, pitfalls) |
| `REQUIREMENTS.md` | Scoped v1/v2 requirements with phase traceability |
| `ROADMAP.md` | Where you're going, what's done |
| `STATE.md` | Decisions, blockers, position — memory across sessions |
| `PLAN.md` | Atomic task with XML structure, verification steps |
| `SUMMARY.md` | What happened, what changed, committed to history |
| `RETROSPECTIVE.md` | What worked and what didn't — feeds into future planning |
| `SPRINT-REPORT.md` | Per-phase results from last sprint run |
| `skills/` | Advisory role definitions (PM, UX, Security, Prod Eng) |
| `todos/` | Captured ideas and tasks for later work |

Size limits based on where Claude's quality degrades. Stay under, get consistent excellence.

### XML Prompt Formatting

Every plan is structured XML optimized for Claude:

```xml
<task type="auto">
  <name>Create login endpoint</name>
  <files>src/app/api/auth/login/route.ts</files>
  <action>
    Use jose for JWT (not jsonwebtoken - CommonJS issues).
    Validate credentials against users table.
    Return httpOnly cookie on success.
  </action>
  <verify>curl -X POST localhost:3000/api/auth/login returns 200 + Set-Cookie</verify>
  <done>Valid credentials return cookie, invalid return 401</done>
</task>
```

Precise instructions. No guessing. Verification built in.

### Multi-Agent Orchestration

Every stage uses the same pattern: a thin orchestrator spawns specialized agents, collects results, and routes to the next step.

| Stage | Orchestrator does | Agents do |
|-------|------------------|-----------|
| Research | Coordinates, presents findings | 4 parallel researchers investigate stack, features, architecture, pitfalls |
| Planning | Validates, manages iteration | Planner creates plans, checker verifies, loop until pass |
| Execution | Groups into waves, tracks progress | Executors implement in parallel, each with fresh 200k context |
| Verification | Presents results, routes next | Verifier checks codebase against goals, debuggers diagnose failures |

The orchestrator never does heavy lifting. It spawns agents, waits, integrates results.

**The result:** You can run an entire phase — deep research, multiple plans created and verified, thousands of lines of code written across parallel executors, automated verification against goals — and your main context window stays at 30-40%. The work happens in fresh subagent contexts. Your session stays fast and responsive.

### Atomic Git Commits

Each task gets its own commit immediately after completion:

```bash
abc123f docs(08-02): complete user registration plan
def456g feat(08-02): add email confirmation flow
hij789k feat(08-02): implement password hashing
lmn012o feat(08-02): create registration endpoint
```

> [!NOTE]
> **Benefits:** Git bisect finds exact failing task. Each task independently revertable. Clear history for Claude in future sessions. Better observability in AI-automated workflow.

Every commit is surgical, traceable, and meaningful.

### Modular by Design

- Add phases to current milestone
- Insert urgent work between phases
- Complete milestones and start fresh
- Adjust plans without rebuilding everything

You're never locked in. The system adapts.

---

## Commands

### Core Workflow

| Command | What it does |
|---------|--------------|
| `/gsd:new-project [--auto]` | Full initialization: questions → research → requirements → roadmap |
| `/gsd:discuss-phase [N] [--auto]` | Capture implementation decisions before planning |
| `/gsd:plan-phase [N] [--auto] [--prd <file>]` | Research + plan + verify for a phase (PRD skips discuss) |
| `/gsd:execute-phase <N>` | Execute all plans in parallel waves, verify when complete |
| `/gsd:verify-work [N]` | Manual user acceptance testing + quality gates ¹ |
| `/gsd:audit-milestone` | Verify milestone achieved its definition of done |
| `/gsd:complete-milestone` | Archive milestone, tag release |
| `/gsd:new-milestone [name]` | Start next version: questions → research → requirements → roadmap |

### Accelerators

| Command | What it does |
|---------|--------------|
| `/gsd:sprint [range]` | Run all remaining phases (or a range) unattended |
| `/gsd:consolidated-phase <N>` | Full phase in 3 steps: consensus+plan → execute+gate → ship |
| `/gsd:advisory-consensus <N>` | Multi-perspective advisory feedback in one agent pass |

### Navigation

| Command | What it does |
|---------|--------------|
| `/gsd:progress` | Where am I? What's next? |
| `/gsd:help` | Show all commands and usage guide |
| `/gsd:update` | Update GSD with changelog preview |
| `/gsd:join-discord` | Join the GSD Discord community |

### Brownfield

| Command | What it does |
|---------|--------------|
| `/gsd:map-codebase` | Analyze existing codebase before new-project |

### Phase Management

| Command | What it does |
|---------|--------------|
| `/gsd:add-phase` | Append phase to roadmap |
| `/gsd:insert-phase [N]` | Insert urgent work between phases |
| `/gsd:remove-phase [N]` | Remove future phase, renumber |
| `/gsd:list-phase-assumptions [N]` | See Claude's intended approach before planning |
| `/gsd:plan-milestone-gaps` | Create phases to close gaps from audit |

### Session

| Command | What it does |
|---------|--------------|
| `/gsd:pause-work` | Create handoff when stopping mid-phase |
| `/gsd:resume-work` | Restore from last session |

### Utilities

| Command | What it does |
|---------|--------------|
| `/gsd:settings` | Configure model profile and workflow agents |
| `/gsd:set-profile <profile>` | Switch model profile (quality/balanced/budget) |
| `/gsd:add-todo [desc]` | Capture idea for later |
| `/gsd:check-todos` | List pending todos |
| `/gsd:debug [desc]` | Systematic debugging with persistent state |
| `/gsd:quick [--full]` | Execute ad-hoc task with GSD guarantees (`--full` adds plan-checking and verification) |
| `/gsd:health [--repair]` | Validate `.planning/` directory integrity, auto-repair with `--repair` |

<sup>¹ Contributed by reddit user OracleGreyBeard</sup>

---

## Configuration

GSD stores project settings in `.planning/config.json`. Configure during `/gsd:new-project` or update later with `/gsd:settings`. For the full config schema, workflow toggles, git branching options, and per-agent model breakdown, see the [User Guide](docs/USER-GUIDE.md#configuration-reference).

### Core Settings

| Setting | Options | Default | What it controls |
|---------|---------|---------|------------------|
| `mode` | `yolo`, `interactive` | `interactive` | Auto-approve vs confirm at each step |
| `depth` | `quick`, `standard`, `comprehensive` | `standard` | Planning thoroughness (phases × plans) |

### Model Profiles

Control which Claude model each agent uses. Balance quality vs token spend.

| Profile | Planning | Execution | Verification |
|---------|----------|-----------|--------------|
| `quality` | Opus | Opus | Sonnet |
| `balanced` (default) | Opus | Sonnet | Sonnet |
| `budget` | Sonnet | Sonnet | Haiku |

Switch profiles:
```
/gsd:set-profile budget
```

Or configure via `/gsd:settings`.

### Workflow Agents

These spawn additional agents during planning/execution. They improve quality but add tokens and time.

| Setting | Default | What it does |
|---------|---------|--------------|
| `workflow.research` | `true` | Researches domain before planning each phase |
| `workflow.plan_check` | `true` | Verifies plans achieve phase goals before execution |
| `workflow.verifier` | `true` | Confirms must-haves were delivered after execution |
| `workflow.auto_advance` | `false` | Auto-chain discuss → plan → execute without stopping |
| `workflow.consolidated` | `false` | Use consolidated 3-phase workflow instead of standard |

Use `/gsd:settings` to toggle these, or override per-invocation:
- `/gsd:plan-phase --skip-research`
- `/gsd:plan-phase --skip-verify`

### Execution

| Setting | Default | What it controls |
|---------|---------|------------------|
| `parallelization.enabled` | `true` | Run independent plans simultaneously |
| `planning.commit_docs` | `true` | Track `.planning/` in git |

### Sprint

| Setting | Options | Default | What it does |
|---------|---------|---------|--------------|
| `sprint.skip_on_failure` | `true`, `false` | `false` | Skip failed phases and continue (vs stop on first failure) |

Override per-invocation with `--skip-failures`.

### Quality Gates

Automate verification with your existing test infrastructure. Configure via `/gsd:settings` or directly in `.planning/config.json`.

| Setting | Options | Default | What it does |
|---------|---------|---------|--------------|
| `quality_gates.enabled` | `off`, `warn`, `block` | `off` | Gate behavior on failure |
| `quality_gates.commands` | string[] | `[]` | Commands to run (e.g. `["npm test", "npx tsc --noEmit"]`) |

- **`off`** — No gates (default)
- **`warn`** — Run gates, show results, continue on failure
- **`block`** — Run gates, stop execution on failure

Gates run automatically after phase verification and during `/gsd:verify-work`. Results append to `VERIFICATION.md`.

### Git Branching

Control how GSD handles branches during execution.

| Setting | Options | Default | What it does |
|---------|---------|---------|--------------|
| `git.branching_strategy` | `none`, `phase`, `milestone` | `none` | Branch creation strategy |
| `git.phase_branch_template` | string | `gsd/phase-{phase}-{slug}` | Template for phase branches |
| `git.milestone_branch_template` | string | `gsd/{milestone}-{slug}` | Template for milestone branches |

**Strategies:**
- **`none`** — Commits to current branch (default GSD behavior)
- **`phase`** — Creates a branch per phase, merges at phase completion
- **`milestone`** — Creates one branch for entire milestone, merges at completion

At milestone completion, GSD offers squash merge (recommended) or merge with history.

---

## Security

### Protecting Sensitive Files

GSD's codebase mapping and analysis commands read files to understand your project. **Protect files containing secrets** by adding them to Claude Code's deny list:

1. Open Claude Code settings (`.claude/settings.json` or global)
2. Add sensitive file patterns to the deny list:

```json
{
  "permissions": {
    "deny": [
      "Read(.env)",
      "Read(.env.*)",
      "Read(**/secrets/*)",
      "Read(**/*credential*)",
      "Read(**/*.pem)",
      "Read(**/*.key)"
    ]
  }
}
```

This prevents Claude from reading these files entirely, regardless of what commands you run.

> [!IMPORTANT]
> GSD includes built-in protections against committing secrets, but defense-in-depth is best practice. Deny read access to sensitive files as a first line of defense.

---

## Troubleshooting

**Commands not found after install?**
- Restart Claude Code to reload slash commands
- Verify files exist in `~/.claude/commands/gsd/` (global) or `./.claude/commands/gsd/` (local)

**Commands not working as expected?**
- Run `/gsd:help` to verify installation
- Re-run `node bin/install.js` from your clone to reinstall

**Updating to the latest version?**
```bash
cd get-shit-done
git pull
node bin/install.js
```

**Using Docker or containerized environments?**

If file reads fail with tilde paths (`~/.claude/...`), set `CLAUDE_CONFIG_DIR` before installing:
```bash
CLAUDE_CONFIG_DIR=/home/youruser/.claude node bin/install.js --global
```
This ensures absolute paths are used instead of `~` which may not expand correctly in containers.

### Uninstalling

To remove GSD completely:

```bash
# Global installs
node bin/install.js --claude --global --uninstall
node bin/install.js --opencode --global --uninstall

# Local installs (current project)
node bin/install.js --claude --local --uninstall
node bin/install.js --opencode --local --uninstall
```

This removes all GSD commands, agents, hooks, and settings while preserving your other configurations.

---

## Upstream & Community

This fork is based on [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) — the original GSD system available via `npx get-shit-done-cc@latest`. This fork adds features battle-tested while building [Weaveto.do](https://github.com/smledbetter/Weaveto.do).

| Project | Description |
|---------|-------------|
| [glittercowboy/get-shit-done](https://github.com/glittercowboy/get-shit-done) | Original upstream — install via npm |
| [gsd-opencode](https://github.com/rokicool/gsd-opencode) | Original OpenCode adaptation |
| gsd-gemini (archived) | Original Gemini adaptation by uberfuzzy |

---

## Star History

<a href="https://star-history.com/#smledbetter/get-shit-done&Date">
 <picture>
   <source media="(prefers-color-scheme: dark)" srcset="https://api.star-history.com/svg?repos=smledbetter/get-shit-done&type=Date&theme=dark" />
   <source media="(prefers-color-scheme: light)" srcset="https://api.star-history.com/svg?repos=smledbetter/get-shit-done&type=Date" />
   <img alt="Star History Chart" src="https://api.star-history.com/svg?repos=smledbetter/get-shit-done&type=Date" />
 </picture>
</a>

---

## License

MIT License. See [LICENSE](LICENSE) for details.

---

<div align="center">

**Claude Code is powerful. GSD makes it reliable.**

</div>
