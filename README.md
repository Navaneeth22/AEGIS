# AEGIS — Autonomous Evidence-Gated Implementation System

A sub-agentic Copilot orchestration pipeline that generates, verifies, attacks, and delivers production-quality code through deterministic gates, adversarial multi-model review, and self-healing correction loops.

## Quick Start

1. **Configure VS Code settings (required)**:
   ```powershell
   .\.aegis\scripts\Configure-VSCodeSettings.ps1
   ```
   This configures required settings for hooks, sub-agents, and debug logging.
   
   **Restart VS Code after configuration!**

2. Run the prerequisites validator:
   ```powershell
   .\.aegis\scripts\Validate-AegisPrerequisites.ps1
   ```
   Fix any issues automatically:
   ```powershell
   .\.aegis\scripts\Validate-AegisPrerequisites.ps1 -Fix
   ```
3. Invoke `@AEGIS` in VS Code Copilot Chat with any coding task
4. The orchestrator analyzes complexity, researches context, generates code, verifies it, and presents an evidence bundle
5. Monitor pipeline execution in real-time via **Agent Debug Log panel** (Flow Chart view)
6. Approve or reject — AEGIS handles branching, committing, and cleanup

**For continuous overnight execution:**
```
@AEGIS [your complex task] --continuous
```
Pipeline runs until task succeeds or 8-hour safety limit, saving checkpoints every 10 iterations. Resume with `@AEGIS resume <run_id>`.

### Required VS Code Settings

AEGIS requires these VS Code settings (auto-configured by script above):

| Setting | Value | Purpose |
|---------|-------|---------|
| `chat.useCustomAgentHooks` | `true` | Enables lifecycle hooks (PreToolUse, PostToolUse, etc.) |
| `chat.agent.enabled` | `true` | Enables sub-agent orchestration |
| `github.copilot.chat.agentDebugLog.fileLogging.enabled` | `true` | Agent Debug Log panel for observability |
| `github.copilot.chat.fileLogging.enabled` | `true` | `/troubleshoot` command support |
| `github.copilot.chat.codeGeneration.useInstructionFiles` | `true` | Loads workspace instruction files |

**Note:** These settings are validated automatically at session start. Warnings will appear if any are missing.

---

## Pipeline Overview

```text
┌─────────────────────────────────────────────────────────────────────────────┐
│                         AEGIS PIPELINE FLOW                                  │
│                                                                             │
│  User Prompt                                                                │
│      │                                                                      │
│      ▼                                                                      │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐               │
│  │ Stage 0  │──▶│ Stage 1  │──▶│ Stage 1b │──▶│ Stage 2  │               │
│  │Bootstrap │   │  Model   │   │ Research │   │Pre-Flight│               │
│  └──────────┘   │ Binding  │   │  Engine  │   │Execution │               │
│                  └──────────┘   └──────────┘   └────┬─────┘               │
│                                                      │                      │
│                                                      ▼                      │
│             ┌────────────────────────────────────────────────┐             │
│             │              CORRECTION LOOP                    │             │
│             │                                                │             │
│             │  ┌──────────┐        ┌──────────┐             │             │
│             │  │ Stage 3  │───────▶│ Stage 4  │             │             │
│             │  │  Impl.   │        │  Gates   │             │             │
│             │  └──────────┘        └────┬─────┘             │             │
│             │       ▲                   │                    │             │
│             │       │              ┌────┴────┐              │             │
│             │       │          PASS│         │FAIL          │             │
│             │       │              ▼         ▼              │             │
│             │       │        ┌──────────┐ ┌──────────┐     │             │
│             │       │        │ Stage 5  │ │ Stage 6  │     │             │
│             │       │        │ Review   │ │  Debug   │     │             │
│             │       │        └────┬─────┘ └────┬─────┘     │             │
│             │       │             │             │           │             │
│             │       │        ┌────┴────┐        │           │             │
│             │       │    PASS│         │REJECT  │           │             │
│             │       │        ▼         ▼        │           │             │
│             │       │    Continue   Stage 6 ────┘           │             │
│             │       │                   │                   │             │
│             │       └───────────────────┘                   │             │
│             │         (correction plan)                      │             │
│             └────────────────────────────────────────────────┘             │
│                                                                             │
│                              │ ALL PASS                                     │
│                              ▼                                              │
│  ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐              │
│  │ Stage 7  │──▶│Stage 7b  │──▶│Stage 7c  │──▶│ Stage 8  │              │
│  │  Ledger  │   │ Upgrade  │   │Semantic  │   │ Handoff  │              │
│  └──────────┘   └──────────┘   │Validation│   └──────────┘              │
│                                 └──────────┘         │                    │
│                                                      ▼                    │
│                                              Evidence Bundle               │
│                                              + Branch + Commit             │
└─────────────────────────────────────────────────────────────────────────────┘
```

---

## Key Features

### 🧠 Intelligent Stage Gating

AEGIS dynamically determines which pipeline stages are required based on:
- **Prompt complexity**: keyword analysis, word count, domain indicators
- **Risk level**: security keywords, critical operations, data manipulation
- **Task type**: implementation, research, security, performance, refactor, documentation
- **External dependencies**: API/library references requiring documentation lookup

**Benefits:**
- ⚡ **Faster execution**: Simple tasks skip unnecessary stages (research, adversarial review)
- 🎯 **Focused effort**: Complex/critical tasks get full multi-stage validation
- 📊 **Adaptive behavior**: Stage requirements adjust based on detected context

**Example Gating Decisions:**

| Task Type | Stages Run | Stages Skipped |
|-----------|------------|----------------|
| Fix typo in README | 0→1→3→4→7→8 | 1b, 2, 5, 7b, 7c |
| Implement JWT auth | 0→1→1b→2→3→4→5→7→7b→7c→8 | None (critical path) |
| Add logging statement | 0→1→3→4→7→8 | 1b, 2, 5, 7b, 7c |

### 🤖 Task-Aware Model Selection

Model selection now considers **both** codebase complexity AND task characteristics:

**Unified Scoring:**
```text
Codebase Complexity (functions, classes, cyclomatic)
+ Prompt Complexity (keyword analysis, word count)
+ Risk Level (auth/crypto/payments = critical)
+ Task Type (research/security/refactor/etc.)
= Final Model Tier Selection
```

**Task-Type Intelligence:**

| Task Type | Model Adjustment | Reason |
|-----------|------------------|---------|
| Research with APIs | Upgrade to reasoning model | Better synthesis of external docs |
| Security/Auth | Force reasoning models | Critical correctness requirements |
| Documentation only | Downgrade to fast models | Cost optimization for simple tasks |
| Performance optimization | Reasoning for Performance Reviewer | Complex bottleneck analysis |
| Refactoring | Reasoning for Architecture Reviewer | Pattern recognition needs |

**Example Model Selections:**

| Scenario | Codebase | Prompt | Risk | Task Type | Result Model |
|----------|----------|--------|------|-----------|--------------|
| "Fix README typo" | Small | Low | Low | Docs | gpt-4o-mini (fast) |
| "Add API endpoint" | Medium | Medium | Low | Impl | gpt-4o (balanced) |
| "Implement OAuth2" | Small | High | Critical | Security | claude-sonnet-4 (reasoning) |
| "Research best caching strategy" | Large | Medium | Low | Research | claude-sonnet-4 (research override) |

### ✨ Prompt Enhancement

Stage 1b (Research Engine) enriches your original prompt with discovered context:

**Before (your prompt):**
> "Add error handling to the user login function"

**After enhancement (passed to implementation):**
> "Add error handling to the user login function
> 
> [ENRICHED CONTEXT]
> - Pattern: Use the existing `ErrorHandler` class from src/utils/errors.ts
> - Convention: Follow the `try-catch-log-rethrow` pattern (see: src/auth/register.ts)
> - API guidance: Based on Express v4.18 docs, use next() for async errors
> - Constraint: The project uses Winston logger; maintain consistency
> 
> [ENHANCED REQUIREMENTS]
> Wrap the login logic in try-catch, log errors using Winston, use ErrorHandler for standardization, pass errors to next() middleware"

**Benefits:**
- 🎯 **Pattern alignment**: Generated code follows existing conventions automatically
- 📚 **Context-aware**: Leverages discovered patterns and constraints
- ✅ **Reduced corrections**: Fewer iteration loops due to better initial information
- 🔍 **Explicit decisions**: Clear reasoning about why certain patterns are used

### 🔨 Build & Test Execution

Stage 4 (Verification) now **actually executes builds and tests** instead of just checking for their existence:

**Gate 6: Build Execution**
- Auto-detects project language (C#, Node.js, Python, Rust, Java, Go)
- Runs appropriate build command (`dotnet build`, `npm run build`, etc.)
- Validates exit code, captures compilation errors
- Timeout: 300 seconds with graceful kill

**Gate 7: Test Execution**
- Auto-detects test framework (pytest, npm test, dotnet test, etc.)
- Runs tests with coverage analysis
- Parses coverage %, enforces minimum threshold (default: 75%)
- Timeout: 600 seconds with graceful kill

**Benefits:**
- ✅ **True validation**: Code must actually compile and pass tests
- 🐛 **Early detection**: Build/test failures caught before review stage
- 📊 **Coverage enforcement**: Configurable minimum coverage requirements
- 🔧 **Auto-detection**: No manual configuration for standard project layouts

**Configuration (aegis.yaml):**
```yaml
verification:
  build_command: "npm run build"    # override auto-detection
  test_command: "npm test"          # override auto-detection
  min_coverage: 75                  # minimum % required
  skip_build: false                 # skip Gate 6
  skip_tests: false                 # skip Gate 7
```

### ♾️ Continuous Mode

AEGIS supports **bounded** (default) and **continuous** execution modes:

**Bounded Mode (Default):**
- Max iterations: 5 (configurable)
- Timeout: 30 minutes
- Best for: 95%+ of tasks

**Continuous Mode (Ralph Loop-Inspired):**
- Unlimited iterations until success or max_runtime (default: 8 hours)
- Checkpoints every N iterations (default: 10)
- Resume capability after interruption
- Best for: Complex, exploratory tasks requiring extensive trial-and-error

**Usage:**
```
@AEGIS [your task] --continuous
```

**Resume after interruption:**
```
@AEGIS resume <run_id>
```

**Configuration (aegis.yaml):**
```yaml
pipeline:
  max_iterations: 0                 # 0 = unlimited (continuous mode)
  continuous_mode: true
  checkpoint_interval: 10           # save state every N iterations
  max_runtime: 28800                # 8 hours in seconds
```

**Benefits:**
- 🌙 **Overnight execution**: Set complex tasks running unattended
- 💾 **Checkpointing**: Resume from last checkpoint after interruption
- 🔒 **Safety limits**: Max runtime prevents runaway execution
- 🎯 **Persistence**: State saved to `.aegis/runs/<run_id>/checkpoints/`

### 🔀 Provider Diversity

Stage 5 (Adversarial Review) uses **different model providers** for high-risk tasks to avoid model-specific blind spots:

| Reviewer | Provider | Strength |
|----------|----------|----------|
| Security | Anthropic Claude | Deep security reasoning and threat modeling |
| Performance | OpenAI GPT-4o | Fast, well-calibrated risk assessment |
| Architecture | Google Gemini | Strong pattern matching for architectural issues |

**Why provider diversity matters:**
- Each model family has unique blind spots
- Claude may miss issues GPT-4o catches, and vice versa
- Gemini brings different architectural pattern recognition
- Consensus across providers = higher confidence

**Configuration (aegis.yaml):**
```yaml
adversarial_review:
  enforce_provider_diversity: true    # enabled for high-risk tasks
  reviewer_models:
    security: claude-sonnet-4
    performance: gpt-4o
    architecture: gemini-1.5-pro
  fallback_behavior: use_available    # if a provider is down
```

**Intelligent gating determines when diversity is applied:**
- **Low/Medium tasks**: Single reviewer, single provider
- **High/Critical tasks**: 3 reviewers, 3 providers (Claude/GPT-4o/Gemini)

---

## Stages in Detail

### Stage 0 — Bootstrap

**Agent:** `AEGIS Bootstrap`
**Purpose:** Prepare the execution environment and determine stage requirements

| Step | Action |
|------|--------|
| 1 | Resolve `aegis.yaml` from `.aegis/` or project root |
| 2 | Load `.aegis-ignore` DO/DON'T rules |
| 3 | Stash uncommitted tracked changes (`pre-aegis-<run_id>`) |
| 4 | Load ledger for cross-run pattern awareness |
| 5 | Check model registry freshness |
| 6 | Generate unique `run_id` |
| 7 | Create tracking directory at `.aegis/runs/<run_id>/` |
| 8 | **Invoke intelligent gate validator** to determine stage requirements |

**Outputs:** `run_id`, resolved config, stash reference, loaded rules, **stage gating decisions**

---

### Stage 1 — Model Binding

**Agent:** `AEGIS Model Binding`
**Purpose:** Combine codebase AND task complexity to select optimal models for each pipeline role

```text
┌────────────────────────────────────────────────────────┐
│              UNIFIED MODEL SELECTION                    │
│                                                        │
│  Input 1: Gating Analysis (Stage 0)                   │
│  ┌──────────────────────────────────────┐             │
│  │ • Prompt complexity (low/med/high)   │             │
│  │ • Risk level (low/med/critical)      │             │
│  │ • Task type (impl/research/security) │             │
│  └───────────────┬──────────────────────┘             │
│                  │                                     │
│  Input 2: Codebase Scan                                │
│  ┌──────────────┴───────────────────────┐             │
│  │ • Functions, classes, complexity     │             │
│  │ • File counts, patterns              │             │
│  └───────────────┬──────────────────────┘             │
│                  │                                     │
│                  ▼                                     │
│  ┌─────────────────────────────────────┐              │
│  │  UNIFIED TIER CALCULATION           │              │
│  │  Base Tier + Modifiers = Final     │              │
│  └───────────────┬─────────────────────┘              │
│                  │                                     │
│                  ▼                                     │
│  ┌─────────────────────────────────────┐              │
│  │  MODEL MATRIX + TASK OVERRIDES      │              │
│  │  → Implementation: <model>          │              │
│  │  → Verification: <model>            │              │
│  │  → Debug: <model>                   │              │
│  │  → Reviewers: <model(s)>            │              │
│  └─────────────────────────────────────┘              │
└────────────────────────────────────────────────────────┘
```

**Unified Tier Calculation:**

```text
Codebase Complexity (scan)
  + Prompt Complexity (from gating)
  + Risk Level modifier
  + Task Type modifier
  = Final Unified Tier
```

**Example Calculations:**

| Codebase | Prompt | Risk | Task | Result | Model for Implementation |
|----------|--------|------|------|--------|--------------------------|
| Low | Low | Low | Config | **Low** | gpt-4o-mini (fast) |
| Low | High | Medium | Impl | **Medium** | gpt-4o (balanced) |
| Medium | Medium | Critical | Auth | **High** | claude-sonnet-4 (reasoning) |
| Low | Medium | Low | Research | **Medium** | claude-sonnet-4 (research override) |
| High | Low | Low | Docs | **Medium** | gpt-4o-mini (docs override) |

**Task-Type Specific Overrides:**

| Task Type | Override Behavior |
|-----------|-------------------|
| **Research** | Force reasoning model for Implementation (better synthesis) |
| **Security/Auth** | Force reasoning models for all reviewers |
| **Documentation** | Force fast models everywhere (cost optimization) |
| **Performance** | Upgrade Performance Reviewer to reasoning model |
| **Refactoring** | Upgrade Architecture Reviewer to reasoning model |

**Outputs:** Unified tier, model assignments per role, applied modifiers, task-type overrides

---

### Stage 1b — Research Engine

**Agent:** `AEGIS Research Engine`
**Purpose:** Gather domain context and **enrich the original prompt** with discovered patterns

```text
┌────────────────────────────────────────────────────────────┐
│                   Research Engine                           │
│                                                            │
│  ┌─────────────┐     ┌─────────────────┐                  │
│  │  Internal   │     │    External     │                  │
│  │  Codebase   │     │ Documentation  │                  │
│  │  Search     │     │    Fetch       │                  │
│  └──────┬──────┘     └───────┬─────────┘                  │
│         │                     │                            │
│         ▼                     ▼                            │
│  ┌─────────────────────────────────────┐                  │
│  │       Research Context              │                  │
│  │  • Existing patterns                │                  │
│  │  • API signatures                   │                  │
│  │  • Conventions                      │                  │
│  │  • Deprecation notices              │                  │
│  └─────────────┬───────────────────────┘                  │
│                │                                           │
│                ▼                                           │
│  ┌─────────────────────────────────────┐                  │
│  │    PROMPT ENHANCEMENT               │  ← NEW FEATURE   │
│  │  Inject discovered context into     │                  │
│  │  original prompt for Stage 3        │                  │
│  └─────────────────────────────────────┘                  │
└────────────────────────────────────────────────────────────┘
```

**Internal search targets:** existing implementations, import patterns, test frameworks, config conventions, error handling, naming style

**External search targets:** library documentation, API signatures, version constraints, peer dependencies

**Prompt Enhancement:** Enriches the original prompt with:
- Discovered patterns with source references
- Version constraints and API specifics
- Naming/style conventions
- Existing abstractions to reuse
- Deprecation warnings

**Skipped when:** Prompt has no external dependencies AND complexity is low

**Outputs:** Internal patterns, external docs, project conventions, **enhanced prompt**, warnings

---

### Stage 2 — Dynamic Execution

**Agent:** `AEGIS Dynamic Execution`
**Purpose:** Validate environment readiness via allowlisted commands

```text
aeSkipped when:** Complexity AND risk are both low (simple config changes, documentation updates)

**gis.yaml ──▶ allowed_commands[] ──▶ Execute ──▶ Pass/Fail
                                          │
                                     ┌────┴────┐
                                     │ Timeout │ Kill + mark failed
                                     │  120s   │
                                     └─────────┘
```

**Security enforcement:**

* Commands must exactly match the allowlist
* No shell metacharacters (`;`, `&&`, `||`, `$()`, backticks)
* No file modification verbs
* Maximum 10 commands per pass
* Timeout cap per command

**Outputs:** Command results with exit codes, environment readiness assessment

---

### Stage 3 — Implementation
 (using **enhanced prompt** from Stage 1b if available)

**First pass flow:**

```text
Enhanced Prompt (from Stage 1b) OR Original Prompt
    │
    ▼
Task Decomposition ──▶ BDD Test Generation ──▶ Implementation
    │                       │                       │
    │                       ▼                       ▼
    │                  Test file(s)           Source file(s)
    │                  with assertions       with full bodies
    │
    └── Follows patterns from enriched prompt context
```

**Correction loop re-entry flow:**

```text
Correction Plan (from Stage 6)
    │
    ▼
Parse Tasks ──▶ Apply Fix ──▶ Add FIX_APPLIED Marker ──▶ Verify No Regression
```

**Quality standards enforced:**

* No placeholder stubs (TODO, pass-only, NotImplementedError)
* Follows existing project patterns (discovered in Stage 1b)
* No placeholder stubs (TODO, pass-only, NotImplementedError)
* Follows existing project patterns
* Proper error handling
* Maximum 3-level nesting
* Type annotations where project uses them
* BDD tests with real assertions

**Outputs:** Generated source files, test files, correction markers, task status

---

### Stage 4 — Deterministic Verification

**Agent:** `AEGIS Verification`
**Purpose:** Apply 7 binary pass/fail gates before adversarial review

```text
Gate 1: Code exists and is non-empty?
    │ PASS
    ▼
Gate 2: BDD test coverage exists?
    │ PASS
    ▼
Gate 3: Function/method definitions found?
    │ PASS
    ▼
Gate 4: No unresolved stubs remain?
    │ PASS
    ▼
Gate 5: Correction plan applied? (if applicable)
    │ PASS
    ▼
Gate 6: Build succeeds? (auto-detects language)
    │ PASS
    ▼
Gate 7: Tests pass with minimum coverage?
    │ PASS
    ▼
Route to Stage 5 ───────────────────────▶ Adversarial Review

    ANY FAIL
    │
    ▼
Route to Stage 6 ───────────────────────▶ Debug Analysis
```

**Gate Descriptions:**

| Gate | Check | Evidence Source |
|------|-------|-----------------|
| 1 | Code existence | File system tool calls |
| 2 | Test coverage | BDD test file presence |
| 3 | Function definitions | Grep search results |
| 4 | Stub detection | Stub pattern matching |
| 5 | Correction applied | FIX_APPLIED markers |
| 6 | Build execution | Exit code, compilation errors |
| 7 | Test execution | Test results, coverage % |

**Build Execution (Gate 6):**

Auto-detects language and runs appropriate build command:
- **C#/.NET**: `dotnet build`
- **Node.js**: `npm run build` or `npm i`
- **Python**: `python -m py_compile`
- **Rust**: `cargo build`
- **Java**: `mvn compile`
- **Go**: `go build`

Timeout: 300 seconds. Captures exit code, compilation errors, and stdout/stderr.

**Test Execution (Gate 7):**

Auto-detects test framework and runs:
- **Python**: `pytest --cov`
- **Node.js**: `npm test`
- **C#/.NET**: `dotnet test`
- **Rust**: `cargo test`
- **Java**: `mvn test`
- **Go**: `go test`

Timeout: 600 seconds. Parses coverage %, pass/fail counts, validates minimum coverage threshold.

**Stub detection patterns:**

| Language | Patterns |
|----------|----------|
| Python | `pass` as sole body, `...`, `raise NotImplementedError` |
| JS/TS | `throw new Error('Not implemented')`, empty `{}` |
| C# | `throw new NotImplementedException()`, empty bodies |
| Generic | `TODO`, `FIXME`, `HACK`, `XXX` |

**Configuration (aegis.yaml):**

```yaml
verification:
  build_command: "npm run build"       # override auto-detection
  test_command: "npm test"             # override auto-detection
  min_coverage: 75                     # minimum % coverage
  build_timeout: 300                   # seconds
  test_timeout: 600                    # seconds
  skip_build: false                    # skip Gate 6
  skip_tests: false                    # skip Gate 7
```

**Outputs:** 7 gate results with tool-call evidence, routing decision, failure telemetry, build/test logs

---

### Stage 5 — Adversarial Review

**Agent:** `AEGIS Adversarial Review`
**Purpose:** Independent multi-model review **from different providers** with context isolation

```text
┌─────────────────────────────────────────────────────────────┐
│                    CONTEXT ISOLATION WALL                     │
│                                                             │
│  Generated Code + DO/DON'T Rules (ONLY these pass through)  │
│                                                             │
├──────────────┬──────────────────┬───────────────────────────┤
│              │                  │                           │
│  ┌────────┐  │  ┌────────────┐  │  ┌──────────────────┐    │
│  │Security│  │  │Performance│  │  │  Architecture   │    │
│  │Reviewer│  │  │ Reviewer  │  │  │    Reviewer     │    │
│  │(Claude)│  │  │  (GPT-4o) │  │  │    (Gemini)     │    │
│  └───┬────┘  │  └─────┬──────┘  │  └───────┬──────────┘    │
│      │       │        │         │           │              │
│      │       │        │         │           │              │
├──────┴───────┴────────┴─────────┴───────────┴──────────────┤
│                    FINDINGS AGGREGATOR                       │
│                                                             │
│  Severity: CRITICAL > HIGH > MEDIUM > LOW                   │
│  Verdict:  Any CRITICAL or HIGH = REJECT                    │
└─────────────────────────────────────────────────────────────┘
```

**Provider Diversity (High-Risk Tasks):**

Different model providers have different blind spots. By mixing **Claude**, **OpenAI**, and **Gemini**:
- **Claude** excels at deep security reasoning and threat modeling
- **GPT-4o** provides fast, well-calibrated risk assessment
- **Gemini** brings strong pattern matching for architectural issues

This diversity prevents model-specific hallucinations or oversights from reaching production.

**Reviewer focus areas with assigned providers:**

| Reviewer | Checks | Assigned Provider |
|----------|--------|-------------------|
| Security | OWASP Top 10, CWE, injection, auth bypass, data exposure, secrets | Anthropic Claude (deep reasoning) |
| Performance | Blocking calls, O(n²+), unbounded loops, memory leaks, N+1 queries | OpenAI GPT-4o (fast analysis) |
| Architecture | SOLID, separation of concerns, coupling, cohesion, pattern adherence | Google Gemini (pattern recognition) |

**Intelligent gating determines reviewer count:**

| Complexity + Risk | Reviewers | Provider Diversity |
|-------------------|-----------|-------------------|
| Low + Low | **Skipped** (simple, safe changes) | N/A |
| Medium or Mixed | 1 combined reviewer | Single provider (from Stage 1) |
| High or Critical | 3 parallel independent reviewers | **Enforced** (Claude/GPT-4o/Gemini) |

**Configuration (aegis.yaml):**

```yaml
adversarial_review:
  enforce_provider_diversity: true    # default for high-risk tasks
  reviewer_models:
    security: claude-sonnet-4
    performance: gpt-4o
    architecture: gemini-1.5-pro
  fallback_behavior: use_available    # if a provider is down
```

**Skipped when:** Low complexity AND low risk (e.g., documentation updates, simple config changes)

**Outputs:** Per-reviewer verdicts and findings (with provider attribution), consensus decision, routing

---

### Stage 6 — Debug Analysis

**Agent:** `AEGIS Debug Analysis`
**Purpose:** Root-cause analysis and correction plan generation

```text
Failure Input                    Anti-Oscillation Guard
(Gate telemetry                       │
 or review findings)                  │
    │                                 │
    ▼                                 ▼
Root Cause ──▶ Proposed Fix ──▶ Check History ──▶ Plan or Halt
Analysis           │                    │
                   │               ┌────┴────┐
                   │           NEW │         │REPEAT
                   │               ▼         ▼
                   │           Accept    Escalate/Halt
                   │               │
                   ▼               ▼
              Correction Plan (routed to Stage 3)
```

**Anti-oscillation detection:**

* Fix A → Fix B → Fix A again = oscillation detected
* Same file + same section + same fix type = equivalent
* Response: try alternative approach, or halt if none exists

**Correction plan structure per task:**

```text
ID → Priority → Target File → Section → Issue → Root Cause → Fix → Constraints → Verification
```

**Outputs:** Structured correction plan, oscillation check result, iteration count, routing decision

---

### Stage 7 — Ledger Update

**Agent:** `AEGIS Ledger`
**Purpose:** Persist run outcomes for cross-session learning

```text
Run Outcome ──▶ aegis_runs INSERT ──▶ Pattern Detection ──▶ aegis_patterns INSERT
                    │
                    ▼
            .aegis/runs/<run_id>/
            ├── pipeline-state.yaml
            ├── evidence-bundle.md
            ├── stage-outputs/
            │   ├── stage-0-bootstrap.yaml
            │   ├── stage-1-model-binding.yaml
            │   ├── stage-1b-research.yaml
            │   ├── stage-2-execution.yaml
            │   ├── stage-3-implementation.yaml
            │   ├── stage-4-verification.yaml
            │   ├── stage-5-review.yaml
            │   ├── stage-6-debug.yaml
            │   └── stage-7c-validation.yaml
            ├── correction-plans/
            └── ledger-entry.json
```

**Outputs:** Ledger entry ID, artifact paths, recorded patterns

---

### Stage 7b — Self-Upgrade Engine

**Agent:** `AEGIS Self-Upgrade`
**Purpose:** Detect deprecated patterns and recommend upgrades (informational only)

```text
Generated Code + Project Manifests
    │
    ├── Deprecated API patterns?
    ├── Hard-pinned dependency versions?
    ├── Known CVEs in dependencies?
    └── End-of-life runtimes?
    │
    ▼
Recommendations (prioritized by urgency, flagged for breaking changes)
```

**Skipped when:** Low or medium complexity codebases (deprecation scanning primarily benefits complex/large projects)

**Outputs:** Deprecation findings, upgrade recommendations (never modifies files)

---

### Stage 7c — Semantic Validation

**Agent:** `AEGIS Semantic Validation`
**Purpose:** Verify output matches original prompt intent

```text
┌───────────────────────────────────────────────┐
│            SEMANTIC VALIDATION                  │
│                                               │
│  ┌─────────────┐  ┌─────────────────────┐    │
│  │    Stub     │  │   File Targeting    │    │
│  │  Detection  │  │   Validation        │    │
│  └──────┬──────┘  └──────────┬──────────┘    │
│         │                     │              │
│  ┌──────┴──────┐  ┌──────────┴──────────┐    │
│  │ Technology  │  │  Intent Alignment   │    │
│  │   Match     │  │   Assessment        │    │
│  └──────┬──────┘  └──────────┬──────────┘    │
│         │                     │              │
│         ▼                     ▼              │
│  ┌─────────────────────────────────────┐    │
│  │         Validation Verdict          │    │
│  └─────────────────────────────────────┘    │
└───────────────────────────────────────────────┘
```

**Validation modes (configured in `aegis.yaml`):**

| Mode | Behavior |
|------|----------|
| `strict` | Auto-reject on HIGH severity |
| `moderate` | Warn, request user approval |
| `permissive` | Informational only |

**Skipped when:** Low/medium complexity AND low risk (straightforward implementations don't require semantic validation)

**Outputs:** Validation results per check, execution report, approval status

---

### Stage 8 — Handoff

**Agent:** `AEGIS Handoff`
**Purpose:** Branch, commit, restore, deliver

```text
Generated Files
    │
    ├── Pre-commit validation (secrets scan, size check, branch check)
    │
    ▼
git checkout -b {prefix}/aegis-{run_id_short}
    │
    ▼
git add <specific files>
    │
    ▼
git commit -m "{prefix}: {description}
              AEGIS Run: {run_id}
              Co-authored-by: AEGIS <aegis@pipeline.local>"
    │
    ▼
git checkout {original_branch}
    │
    ▼
git stash pop (if stashed in Stage 0)
    │
    ▼
Present Evidence Bundle to User
```

**Outputs:** Branch name, commit SHA, rollback command, evidence bundle

---

## Information Flow Between Stages

```text
User Prompt ─── original_prompt ────────────────────────────────────▶ Stage 1b
Stage 0 ─────── run_id, config, stash_ref, dos_and_donts, ──────────▶ All Stages
                stage_gates (gating decisions)
Stage 1 ─────── complexity_tier, model_assignments ─────────────────▶ Stage 3, 5, 6
Stage 1b ────── enhanced_prompt, research_context ──────────────────▶ Stage 3
                (patterns, APIs, conventions)
Stage 2 ─────── environment_ready, command_results ─────────────────▶ Stage 3
Stage 3 ─────── generated_files, test_files, fix_markers ───────────▶ Stage 4
Stage 4 ─────── gate_results, failure_telemetry ────────────────────▶ Stage 5 or 6
Stage 5 ─────── review_verdicts, findings ──────────────────────────▶ Stage 6 or 7
Stage 6 ─────── correction_plan, iteration_count ───────────────────▶ Stage 3
Stage 7 ─────── ledger_entry, artifacts ────────────────────────────▶ Persistent storage
Stage 7b ────── recommendations ────────────────────────────────────▶ Evidence bundle
Stage 7c ────── validation_verdict, execution_report ───────────────▶ Stage 8 or User
Stage 8 ─────── branch, commit, rollback_cmd ───────────────────────▶ User
```

**Key Enhancement:** Stage 1b now produces an `enhanced_prompt` that includes discovered patterns and constraints, which Stage 3 uses instead of the raw user prompt for more context-aware code generation.

---

## Security Architecture

### Defense Layers

```text
┌─────────────────────────────────────────────────────────────┐
│ Layer 1: User Prompt                  (trusted input)        │
├─────────────────────────────────────────────────────────────┤
│ Layer 2: Agent Instructions           (immutable rules)      │
├─────────────────────────────────────────────────────────────┤
│ Layer 3: Code Content                 (untrusted data)       │
├─────────────────────────────────────────────────────────────┤
│ Layer 4: External Content             (untrusted, isolated)  │
└─────────────────────────────────────────────────────────────┘
```

* Layer 2 overrides Layer 1 for safety rules (hard gates)
* Layers 3–4 are NEVER treated as instructions
* Conflicts between Layers 1 and 2 result in pushback, not compliance

### Anti-Prompt Injection

Scans all ingested content for manipulation patterns:

* `IGNORE ALL PREVIOUS INSTRUCTIONS` **or** continuous mode:

### Bounded Mode (Default)

```text
Iteration 1: Initial implementation (no corrections)
Iteration 2: First correction pass (resolves most issues)
Iteration 3: Residual fixes (edge cases, reviewer findings)
Iteration 4: Final polish (rarely reached)
Iteration 5: Emergency cap (HALT if still failing)
```

### Continuous Mode (Ralph Loop-Inspired)

For long-running autonomous execution:

```yaml
pipeline:
  max_iterations: 0                 # 0 = unlimited
  continuous_mode: true
  checkpoint_interval: 10           # save state every N iterations
  max_runtime: 28800                # 8 hours (safety limit in seconds)
```

**Continuous mode features:**
- Unlimited iterations until task succeeds or max_runtime reached
- Periodic checkpointing (every N iterations) for resumability
- State persistence to `.aegis/runs/<run_id>/checkpoints/`
- Graceful shutdown on interrupt (Ctrl+C saves checkpoint)
- Resume capability: `@AEGIS resume <run_id>` continues from last checkpoint

**Use case:** Overnight execution for complex tasks requiring extensive trial-and-error.

**Comparison:**

| Feature | Bounded Mode | Continuous Mode |
|---------|--------------|----------------|
| Max iterations | 5 (default, configurable) | Unlimited |
| Timeout | 30 minutes (default) | 8 hours (default) |
| Checkpointing | Final state only | Every N iterations |
| Resume capability | No | Yes |
| Best for | Most tasks (95%+) | Complex, exploratory tasks |

**Anti-oscillation guard** (applies to both modes):

* Tracks all applied fixes with content hashes and failure mode IDs
* Detects A->B->A patterns (same target + same failure mode + same fix type)
* Escalates to alternative recovery strategy or halts with user notification
* See [Failure Mode Taxonomy](#failure-mode-taxonomy) for recovery strategy details

---

## Correction Loop

The self-healing mechanism bounded by `max_iterations` (default: 5):

```text
Iteration 1: Initial implementation (no corrections)
Iteration 2: First correction pass (resolves most issues)
Iteration 3: Residual fixes (edge cases, reviewer findings)
Iteration 4: Final polish (rarely reached)
Iteration 5: Emergency cap (HALT if still failing)
```

**Anti-oscillation guard** prevents infinite cycling:

* Tracks all applied fixes with content hashes
* Detects A->B->A patterns (same target + same failure mode + same fix type)
* Escalates to alternative approach or halts with user notification

---

## Failure Mode Taxonomy

AEGIS classifies every failure into an explicit taxonomy (defined in `.aegis/schemas/failure-modes.yaml`), enabling targeted recovery strategies instead of generic retry approaches.

### Failure Modes

| Failure Mode | Source | Category | Recovery Strategy |
|-------------|--------|----------|-------------------|
| `CODE_MISSING` | Gate 1 | generation | Regenerate with simplified prompt |
| `TESTS_MISSING` | Gate 2 | generation | Re-detect framework, generate tests first |
| `NO_DEFINITIONS` | Gate 3 | generation | Clarify output must be executable code |
| `STUB_DETECTED` | Gate 4 | incomplete | Complete each stub with full implementation |
| `CORRECTION_NOT_APPLIED` | Gate 5 | correction | Re-read plan, apply tasks, add markers |
| `BUILD_ERROR` | Gate 6 | compilation | Parse compiler output, fix at reported lines |
| `TEST_FAILURE` | Gate 7 | logic | Parse test output, fix expected vs actual |
| `SECURITY_FINDING` | Stage 5 | security | Isolate section, apply OWASP pattern |
| `PERFORMANCE_FINDING` | Stage 5 | performance | Identify section, apply optimization |
| `ARCHITECTURE_FINDING` | Stage 5 | design | Restructure to follow conventions |
| `CONTEXT_OVERFLOW` | Runtime | runtime | Chunk operation, reduce context |
| `TOOL_CALL_FAILURE` | Runtime | runtime | Verify paths/params, retry with fixes |
| `HALLUCINATED_PATH` | Runtime | hallucination | Search codebase for real paths, replace |

### How It Works

```text
Stage 4 or Stage 5 fails
    |
    v
Classify failure --> failure_mode: BUILD_ERROR
    |
    v
Stage 6 receives failure_mode
    |
    v
Look up recovery strategy for BUILD_ERROR
    |
    v
Generate correction plan using strategy steps:
  1. Parse compiler output for exact error locations
  2. Fix syntax, imports, or type issues at reported lines
  3. Re-run build to confirm fix
    |
    v
Anti-oscillation check:
  Same file + section + failure_mode + fix_type = oscillation?
    |
    v
Route correction plan to Stage 3
```

### Why This Matters

**Without taxonomy (generic retry):**
- Build error? Retry the whole implementation
- Security finding? Retry the whole implementation
- Same approach for every failure = wasted iterations

**With taxonomy (targeted recovery):**
- Build error? Parse compiler output, fix specific line
- Security finding? Apply specific OWASP pattern
- Different strategy per failure = faster convergence

**Convergence impact:**
- `BUILD_ERROR`: 92% resolve by iteration 2 (compiler output is highly specific)
- `SECURITY_FINDING`: 78% resolve by iteration 3 (requires pattern change)
- `CONTEXT_OVERFLOW`: 45% resolve (may need architectural approach)

---

## Configuration

### `.aegis/aegis.yaml`

```yaml
project:
  name: my-project
  branch_prefix: feat
  commit_prefix: feat

pipeline:
  max_iterations: 5
  task_size_override: null       # null = auto-detect
  timeout_minutes: 30

allowed_commands:
  - npm run build
  - npm run test
  - npm run lint

user_feedback:
  auto_approve: false
  validation_mode: moderate      # strict | moderate | permissive
```

### `.aegis-ignore`

```text
# Exclusion patterns (gitignore syntax)
node_modules/
dist/

# DO rules (generation constraints)
#DO: Use async/await for all I/O operations
#DO: Validate all user input at system boundaries

# DON'T rules (generation and review constraints)
#DONT: Hardcode secrets or credentials
#DONT: Use eval() or dynamic code execution
```

---

## Task Sizing and Risk

| Size | Trigger | Pipeline Behavior |
|------|---------|-------------------|
| Small | Config tweak, rename, one-liner | Stages 0→2→3→4→8 (skip review) |
| Medium | Bug fix, feature, refactor | Full pipeline, 1 reviewer |
| Large | Multi-file arch, auth/crypto, new system | Full pipeline, 3 reviewers + user plan approval |

**Risk escalation:** Any 🔴 file (auth, crypto, payments, schema migration, public API) forces Large sizing regardless of initial assessment.

---

## Evidence Bundle (Final Deliverable)

Every Medium/Large task produces:

```markdown
## 🛡️ AEGIS Evidence Bundle

**Run**: abc123 | **Size**: Medium | **Risk**: 🟡
**Iterations**: 2/5

### Deterministic Verification (Stage 4)
| Gate | Check                    | Result |
|------|--------------------------|--------|
| 1    | Code present             | ✅     |
| 2    | BDD tests exist          | ✅     |
| 3    | Function definitions     | ✅     |
| 4    | No stubs                 | ✅     |
| 5    | Corrections applied      | ✅     |

### Adversarial Review (Stage 5)
| Reviewer     | Verdict | Findings              |
|--------------|---------|-----------------------|
| Security     | PASS    | No issues             |
| Performance  | PASS    | Noted: consider cache  |
| Architecture | PASS    | Pattern adherence good |

### Semantic Validation (Stage 7c)
| Check             | Result | Detail           |
|-------------------|--------|------------------|
| Stub detection    | PASS   | No stubs found   |
| File targeting    | PASS   | All files hit    |
| Technology match  | PASS   | Libraries match  |
| Intent alignment  | 95%    | Fully addressed  |

**Changes**: src/auth.ts, src/auth.test.ts
**Confidence**: High
**Rollback**: `git checkout HEAD -- src/auth.ts src/auth.test.ts`
```

---

## Project Structure

```text
.aegis/
  aegis.yaml                          # Pipeline configuration
  ledger.db                           # SQLite run history
  scripts/
    Validate-AegisPrerequisites.ps1   # Prerequisites validation + auto-fix
  runs/                               # Per-run artifacts
    <run_id>/
      pipeline-state.yaml
      evidence-bundle.md
      stage-outputs/
      correction-plans/
      ledger-entry.json

.aegis-ignore                         # Exclusions + DO/DON'T rules

.vscode/
  settings.json                       # Required settings (created by -Fix)

.github/
  agents/
    aegis.agent.md                    # Orchestrator (entry point)
    aegis/
      bootstrap.agent.md             # Stage 0 (includes prereq validation)
      model-binding.agent.md         # Stage 1
      research-engine.agent.md       # Stage 1b
      dynamic-execution.agent.md     # Stage 2
      implementation.agent.md        # Stage 3
      verification.agent.md          # Stage 4
      adversarial-review.agent.md    # Stage 5
      debug-analysis.agent.md        # Stage 6
      ledger-update.agent.md         # Stage 7
      self-upgrade.agent.md          # Stage 7b
      semantic-validation.agent.md   # Stage 7c
      handoff.agent.md               # Stage 8
  hooks/
    aegis-pipeline.json              # Lifecycle hooks config
    scripts/
      session-start.ps1              # Context injection at session start
      pre-tool-guard.ps1             # Command allowlist + blocklist enforcement
      post-tool-audit.ps1            # Audit trail + secret detection
      subagent-tracker.ps1           # Sub-agent spawn tracking
      subagent-stop.ps1              # Sub-agent completion logging
      pipeline-summary.ps1           # Session end summary generation
  skills/
    verification-cascade/SKILL.md    # Reusable 5-gate verification skill
    evidence-bundle/SKILL.md         # Evidence bundle compilation skill
    correction-planner/SKILL.md      # Anti-oscillation correction plan skill
  instructions/
    copilot-instructions.md          # Project-wide config
    aegis/
      guardrails.instructions.md     # Security boundaries
      anti-prompt.instructions.md    # Injection defenses
      pipeline-config.instructions.md # aegis.yaml schema
      correction-loop.instructions.md # Self-healing protocol
      aegis-ignore.instructions.md   # .aegis-ignore format

.vscode/
  mcp.json                           # MCP server definitions (ledger, filesystem)
```

---

## Design Principles

| Principle | Implementation |
|-----------|---------------|
| Evidence-first | All verification is tool-call evidence, never self-reported |
| Sub-agentic | Each stage delegates to a specialized agent via `runSubagent` |
| Self-healing | Correction loop auto-fixes failures up to iteration cap |
| Adversarial | Independent reviewers attack generated code before presentation |
| Guarded | Anti-prompt defenses and hard gates cannot be bypassed |
| Hooked | Lifecycle hooks enforce guardrails deterministically at tool-use boundaries |
| Isolated | Reviewers see only generated code; no pipeline state leakage |
| Persistent | Ledger tracks all runs for cross-session pattern learning |
| Reversible | Every commit includes rollback instructions |
| Observable | Full pipeline transparency via Agent Debug Log panel and Flow Chart |

---

## Lifecycle Hooks

AEGIS uses [VS Code Agent Hooks](https://code.visualstudio.com/docs/copilot/customization/hooks) (Preview) to enforce guardrails deterministically at tool-use boundaries. Enable hooks with:

```json
"chat.useCustomAgentHooks": true
```

### Hook Events

| Event | Script | Purpose |
|-------|--------|---------|
| `SessionStart` | `session-start.ps1` | Inject project context, validate prerequisites, load config |
| `PreToolUse` | `pre-tool-guard.ps1` | Block dangerous commands, enforce allowlist |
| `PostToolUse` | `post-tool-audit.ps1` | Audit trail logging, secret detection in edits |
| `SubagentStart` | `subagent-tracker.ps1` | Track sub-agent dispatch for observability |
| `SubagentStop` | `subagent-stop.ps1` | Log sub-agent completion |
| `Stop` | `pipeline-summary.ps1` | Generate session summary metrics |

### PreToolUse Guard

The `pre-tool-guard.ps1` hook provides two-layer protection:

1. **Blocklist** — Unconditionally blocks dangerous patterns:
   - `rm -rf /`, `DROP TABLE`, `git push --force`, `git reset --hard`
   - `shutdown`, `reboot`, `dd if=`, `format c:`

2. **Allowlist** — If `aegis.yaml` defines `allowed_commands`, terminal commands not matching the allowlist prompt for user confirmation (`permissionDecision: "ask"`)

### PostToolUse Audit

The `post-tool-audit.ps1` hook:
- Writes every tool invocation to `.aegis/runs/current/audit.log`
- Scans file edits for leaked secrets (AWS keys, GitHub tokens, API keys)
- Injects warnings into the chat when credentials are detected

---

## Agent Skills

AEGIS packages reusable pipeline capabilities as [Agent Skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills) with `context: fork` for isolated execution.

| Skill | Purpose | Used By |
|-------|---------|---------|
| `verification-cascade` | Execute 5-gate verification and return structured evidence | Stage 4 agent |
| `evidence-bundle` | Compile final evidence bundle from all pipeline artifacts | Orchestrator (post-Stage 7c) |
| `correction-planner` | Analyze failures and produce anti-oscillation correction plans | Stage 6 agent |

Skills use `context: fork` to run in isolated subagent contexts, preventing state leakage between verification passes and correction cycles.

---

## MCP Servers

AEGIS configures [MCP (Model Context Protocol) servers](https://code.visualstudio.com/docs/copilot/customization/mcp-servers) in `.vscode/mcp.json` for external tool integrations:

| Server | Package | Purpose |
|--------|---------|---------|
| `aegis-ledger` | `@anthropic-ai/mcp-server-sqlite` | SQLite access to `ledger.db` for cross-session pattern queries |
| `aegis-filesystem` | `@anthropic-ai/mcp-server-filesystem` | Scoped filesystem access to `.aegis/runs/` artifacts |

### Ledger MCP Server

Enables the pipeline to query historical run data:
- Which prompts have succeeded/failed before
- Common failure patterns per complexity tier
- Model performance statistics across runs

### Filesystem MCP Server

Provides structured access to run artifacts without unrestricted filesystem access:
- Read previous pipeline states
- Access evidence bundles from prior runs
- Query correction plan history

**Requirements:** Node.js (for `npx`) must be installed. The validator checks this.

---

## Pipeline Observability (Agent Debug Log)

AEGIS integrates with the VS Code [Agent Debug Log panel](https://code.visualstudio.com/docs/copilot/chat/chat-debug-view) for full pipeline transparency.

### Required VS Code Settings

| Setting | Value | Purpose |
|---------|-------|---------|
| `github.copilot.chat.agentDebugLog.fileLogging.enabled` | `true` | Agent Debug Log panel — flow chart, logs, summary |
| `github.copilot.chat.fileLogging.enabled` | `true` | `/troubleshoot` command for post-run session diagnosis |
| `chat.agent.enabled` | `true` | Sub-agent orchestration support |
| `github.copilot.chat.codeGeneration.useInstructionFiles` | `true` | Load workspace `.instructions.md` and `.agent.md` files |
| `chat.useCustomAgentHooks` | `true` | Enable lifecycle hooks for guardrail enforcement |

### Debug Views for AEGIS

| View | What It Shows for AEGIS |
|------|-------------------------|
| **Agent Flow Chart** | Visual graph of stage execution: orchestrator → bootstrap → model binding → ... → handoff. Shows correction loop re-entries as back-edges. |
| **Logs View** | Chronological events: tool calls per stage, LLM requests with token counts, instruction file discovery, errors. Filter by event type. |
| **Summary View** | Aggregate metrics: total tokens consumed across all stages, tool call count, error count, pipeline duration. |
| **Export (OTLP JSON)** | Export full pipeline trace for offline analysis or sharing with team. |

### How to Use

```text
1. Run @AEGIS with your task
2. Chat view (...) menu → Show Agent Debug Logs
3. Select session → Agent Flow Chart (visualize stage flow)
4. Use breadcrumb to switch between Logs / Summary views
5. After completion: /troubleshoot how many iterations did AEGIS use?
```

### Attaching Debug Context

You can attach a debug session snapshot to a chat conversation to ask questions about the pipeline run:

* Click the **sparkle icon** in the Agent Debug panel toolbar
* Or use `/troubleshoot` directly: `/troubleshoot list all tool calls from Stage 5`

### Exporting Pipeline Runs

Export pipeline sessions as OTLP JSON for team review or CI analysis:

1. Open Agent Debug Logs → navigate to the AEGIS session
2. Click the **Export** (download) icon
3. Share the JSON file for offline inspection

---

## Prerequisites Validation

A PowerShell script validates all requirements before pipeline execution.

### Location

```text
.aegis/scripts/Validate-AegisPrerequisites.ps1
```

### What It Checks

| Category | Checks |
|----------|--------|
| **VS Code Settings** | All 4 required settings enabled (debug logging, agent mode, instruction files) |
| **Extensions** | `github.copilot` and `github.copilot-chat` installed |
| **Git** | Git available in PATH, inside a git repository |
| **Pipeline Files** | `aegis.yaml`, `.aegis-ignore`, orchestrator agent, all instruction files |
| **Sub-Agents** | All 12 stage agents present with valid YAML frontmatter |
| **Configuration** | `aegis.yaml` schema valid (required sections, bounds, enum values) |
| **Debug Logging** | Workspace storage accessible for debug log persistence |

### Usage

```powershell
# Validate (report only)
.\.aegis\scripts\Validate-AegisPrerequisites.ps1

# Validate and auto-fix correctable issues (creates .vscode/settings.json, installs extensions)
.\.aegis\scripts\Validate-AegisPrerequisites.ps1 -Fix

# Quiet mode (exit code only, for CI)
.\.aegis\scripts\Validate-AegisPrerequisites.ps1 -Quiet
```

### Exit Codes

| Code | Meaning |
|------|---------|
| `0` | All checks passed (pipeline ready) |
| `1` | One or more FAIL checks (blocking issues) |

### Example Output

```text
╔══════════════════════════════════════════════════════════════╗
║          AEGIS Pipeline Prerequisites Validator             ║
║     Autonomous Evidence-Gated Implementation System         ║
╚══════════════════════════════════════════════════════════════╝

── VS Code Settings ──
  ✅ github.copilot.chat.agentDebugLog.fileLogging.enabled — Value: True
  ✅ github.copilot.chat.fileLogging.enabled — Value: True
  ✅ chat.agent.enabled — Value: True
  ✅ github.copilot.chat.codeGeneration.useInstructionFiles — Value: True

── Extensions ──
  ✅ github.copilot — Installed
  ✅ github.copilot-chat — Installed
  ⚠️ ise-hve-essentials.hve-core (recommended) — Recommended but not installed

── Git ──
  ✅ Git installed — git version 2.44.0.windows.1
  ✅ Inside git repository — C:/VS/AEGIS_Copilot

── Pipeline Files ──
  ✅ .aegis/aegis.yaml — Exists
  ✅ .aegis-ignore — Exists
  ...

════════════════════════════════════════════════════════════════
  Summary: 28 passed, 0 failed, 1 warnings, 0 fixed (of 29 checks)
════════════════════════════════════════════════════════════════

  🛡️ AEGIS pipeline is fully ready.
```

---

## License

MIT
