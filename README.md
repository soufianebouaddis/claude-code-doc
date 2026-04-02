# Claude Code Leaked Source Code — Full Breakdown of Anthropic's npm Sourcemap Leak

![Stars](https://img.shields.io/github/stars/soufianebouaddis/claude-code?style=flat-square&logo=github&label=Stars)
![Forks](https://img.shields.io/github/forks/soufianebouaddis/claude-code?style=flat-square&logo=github&label=Forks)
![Last Commit](https://img.shields.io/github/last-commit/soufianebouaddis/claude-code?style=flat-square&label=Last%20Commit)
![Language](https://img.shields.io/github/languages/top/soufianebouaddis/claude-code?style=flat-square&label=Language)
![Lines of Code](https://img.shields.io/badge/Lines-512%2C000%2B-orange?style=flat-square)
![Files](https://img.shields.io/badge/Files-1%2C900%2B-green?style=flat-square)
![License](https://img.shields.io/badge/License-Educational%20Use%20Only-blue?style=flat-square)

> **On March 31, 2026, security researcher Chaofan Shou ([@Fried_rice](https://x.com/fried_rice/status/2038894956459290963)) discovered that Anthropic's Claude Code CLI had its entire source code exposed via a `.map` sourcemap file bundled inside the `@anthropic-ai/claude-code` npm package (v2.1.88). This repository is a full backup and deep-dive analysis of the leaked codebase — 1,900+ TypeScript files, 512,000+ lines of code, including unreleased features never meant to be public.**

> **This repo may be taken down. Fork it now if you want to preserve a copy.**

---

## Table of Contents

- [How the Leak Happened](#how-did-claude-codes-source-code-leak-from-npm)
- [What's Inside](#what-is-inside-the-leaked-claude-code-codebase)
- [BUDDY — Hidden Tamagotchi Pet](#buddy--anthropics-hidden-tamagotchi-inside-claude-code)
- [KAIROS — Always-On Claude](#kairos--claude-codes-unreleased-always-on-mode)
- [ULTRAPLAN — Remote Planning Sessions](#ultraplan--30-minute-remote-planning-sessions)
- [The Dream System](#the-dream-system--claude-literally-dreams)
- [Undercover Mode](#undercover-mode--do-not-blow-your-cover)
- [Multi-Agent Orchestration](#multi-agent-orchestration--coordinator-mode)
- [Full Tool Registry (40+ Tools)](#claude-codes-40-tool-registry)
- [The Permission & Security System](#the-permission-and-security-system)
- [Hidden Beta API Features](#hidden-beta-headers-and-unreleased-api-features)
- [Feature Flags — Internal vs External Builds](#feature-gating--internal-vs-external-builds)
- [Other Notable Findings](#other-notable-findings)
- [Security Implications](#security-implications-of-the-claude-code-source-leak)
- [FAQ](#faq)

---

## How Did Claude Code's Source Code Leak from npm?

When you publish a JavaScript/TypeScript package to npm, build toolchains often generate **source map files** (`.map` files). These files exist so that minified production stack traces can point back to the original source. The problem: **source maps embed the raw original source code verbatim** inside a JSON structure.

```json
{
  "version": 3,
  "sources": ["../src/main.tsx", "../src/tools/BashTool.ts", "..."],
  "sourcesContent": ["// The ENTIRE original source code of each file", "..."],
  "mappings": "AAAA,SAAS,OAAO..."
}
```

That `sourcesContent` array contains everything — every file, every comment, every internal constant, every system prompt. Anthropic's team, likely using Bun's bundler (which generates source maps by default), forgot to add `*.map` to `.npmignore` or disable source map output for production builds. The result: the complete Claude Code source was sitting on the public npm registry, accessible to anyone running `npm pack`.

The irony? Claude Code contains a feature called [**Undercover Mode**](#undercover-mode--do-not-blow-your-cover) specifically designed to prevent internal information from leaking — and then shipped the entire source in a `.map` file.

---

## What Is Inside the Leaked Claude Code Codebase?

From the outside, Claude Code looks like a polished terminal CLI. From the inside it is a **785KB `main.tsx`** entry point, a custom React terminal renderer (using [Ink](https://github.com/vadimdemedes/ink)), 40+ registered tools, a full multi-agent orchestration system, a background memory consolidation engine called **"dream"**, and several unreleased modes gated behind compile-time feature flags.

Key stats about the leaked source:

| Metric | Value |
|--------|-------|
| Total lines of code | **512,000+** |
| Total source files | **1,900+** |
| Main entry point size | **785 KB** (`main.tsx`) |
| Runtime | Bun |
| UI framework | React + Ink (terminal renderer) |
| npm package | `@anthropic-ai/claude-code` v2.1.88 |
| Discoverer | Chaofan Shou [@Fried_rice](https://x.com/fried_rice/status/2038894956459290963) |
| Discovery date | March 31, 2026 |

---

## BUDDY — Anthropic's Hidden Tamagotchi Inside Claude Code

This is the feature that broke the internet — and for good reason.

Claude Code contains a full **Tamagotchi-style companion pet system** called **BUDDY**, gated behind the `BUDDY` compile-time feature flag and not present in any public build. It has a deterministic gacha system, species rarity tiers, shiny variants, procedurally generated stats, and a personality description written by Claude itself on first hatch.

### The Gacha System

Your buddy's species is determined by a **Mulberry32 PRNG** seeded from your `userId` hash with the salt `'friend-2026-401'` — meaning the same user always gets the same buddy, reproducibly:

```typescript
function mulberry32(seed: number): () => number {
  return function() {
    seed |= 0; seed = seed + 0x6D2B79F5 | 0;
    var t = Math.imul(seed ^ seed >>> 15, 1 | seed);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }
}
```

### 18 Species (Hidden via String Obfuscation)

The species names are obfuscated in the source via `String.fromCharCode()` arrays — Anthropic clearly didn't want them surfacing in string searches. Decoded:

| Rarity | Chance | Species |
|--------|--------|---------|
| **Common** | 60% | Pebblecrab, Dustbunny, Mossfrog, Twigling, Dewdrop, Puddlefish |
| **Uncommon** | 25% | Cloudferret, Gustowl, Bramblebear, Thornfox |
| **Rare** | 10% | Crystaldrake, Deepstag, Lavapup |
| **Epic** | 4% | Stormwyrm, Voidcat, Aetherling |
| **Legendary** | 1% | Cosmoshale, Nebulynx |

There's also a **1% shiny chance** independent of rarity — making a Shiny Legendary Nebulynx a **0.01%** roll.

### Stats, Eyes, Hats, and Soul

Each buddy gets procedurally generated:
- **5 stats**: `DEBUGGING`, `PATIENCE`, `CHAOS`, `WISDOM`, `SNARK` (0–100 each)
- **6 eye styles** and **8 hat options** (some rarity-gated)
- **A "soul"** — a personality written by Claude on first hatch, in character

Sprites are **5-line-tall, 12-character-wide ASCII art** with multiple animation frames (idle, reaction). They sit beside your terminal input prompt and can respond when addressed by name.

### Launch Timeline

The code references **April 1–7, 2026** as a teaser window with full launch gated for **May 2026**.

---

## KAIROS — Claude Code's Unreleased Always-On Mode

Inside the `assistant/` directory, there's an entire mode called **KAIROS** — a persistent, always-running Claude assistant that doesn't wait for you to type. It watches, logs, and **proactively acts** on things it notices. Gated behind the `PROACTIVE` / `KAIROS` compile-time flags, completely absent from external builds.

### How KAIROS Works

KAIROS maintains **append-only daily log files**, writing observations and decisions throughout the day. On a regular interval it receives `<tick>` prompts that let it decide whether to act proactively or stay quiet. It has a **15-second blocking budget** — any proactive action that would block your workflow longer gets deferred.

### KAIROS-Exclusive Tools

| Tool | What It Does |
|------|-------------|
| `SendUserFile` | Push files directly to the user |
| `PushNotification` | Send push notifications to the user's device |
| `SubscribePR` | Subscribe to and monitor pull request activity |

**Brief Mode** is a special output mode for KAIROS — extremely concise responses designed for a persistent assistant that shouldn't flood your terminal.

---

## ULTRAPLAN — 30-Minute Remote Planning Sessions

**ULTRAPLAN** is a mode where Claude Code offloads a complex planning task to a remote **Cloud Container Runtime (CCR) session** running Opus 4.6, gives it up to **30 minutes** to think, and lets you approve the result from your browser.

Flow:
1. Claude Code identifies a task needing deep planning
2. Spins up a remote CCR session via the `tengu_ultraplan_model` config
3. Your terminal enters a polling state (checks every **3 seconds**)
4. A browser UI lets you watch the planning happen live and approve/reject
5. On approval, a sentinel value `__ULTRAPLAN_TELEPORT_LOCAL__` "teleports" the result back to your local terminal

---

## The Dream System — Claude Literally Dreams

The `services/autoDream/` directory contains **autoDream** — a background memory consolidation engine that runs as a forked subagent. The naming is intentional. It's Claude dreaming.

### Three-Gate Trigger

The dream only runs when all three conditions pass:

1. **Time gate**: 24 hours since last dream
2. **Session gate**: At least 5 sessions since last dream
3. **Lock gate**: Acquires a consolidation lock (prevents concurrent dreams)

### Four Dream Phases

**Phase 1 — Orient**: `ls` the memory directory, read `MEMORY.md`, skim existing topic files.

**Phase 2 — Gather Recent Signal**: Find new information worth persisting, in priority: daily logs → drifted memories → transcript search.

**Phase 3 — Consolidate**: Write or update memory files. Convert relative dates to absolute. Delete contradicted facts.

**Phase 4 — Prune and Index**: Keep `MEMORY.md` under 200 lines and ~25KB. Remove stale pointers. Resolve contradictions.

From `consolidationPrompt.ts`:

> *"You are performing a dream — a reflective pass over your memory files. Synthesize what you've learned recently into durable, well-organized memories so that future sessions can orient quickly."*

The dream subagent gets **read-only bash** — it can look at your project but cannot modify anything.

---

## Undercover Mode — "Do Not Blow Your Cover"

Anthropic employees (`USER_TYPE === 'ant'`) use Claude Code to contribute to public open-source repositories. **Undercover Mode** (`utils/undercover.ts`) prevents the AI from accidentally revealing internal information in commits and PRs.

When active, it injects this into the system prompt:

```
## UNDERCOVER MODE - CRITICAL

You are operating UNDERCOVER in a PUBLIC/OPEN-SOURCE repository. Your commit
messages, PR titles, and PR bodies MUST NOT contain ANY Anthropic-internal
information. Do not blow your cover.

NEVER include in commit messages or PR descriptions:
- Internal model codenames (animal names like Capybara, Tengu, etc.)
- Unreleased model version numbers (e.g., opus-4-7, sonnet-4-8)
- Internal repo or project names
- Internal tooling, Slack channels, or short links (e.g., go/cc, #claude-code-…)
- The phrase "Claude Code" or any mention that you are an AI
- Co-Authored-By lines or any other attribution
```

**Activation logic**:
- `CLAUDE_CODE_UNDERCOVER=1` forces it ON (even in internal repos)
- Otherwise it activates automatically unless the repo remote matches an internal allowlist
- There is **no force-OFF** — *"if we're not confident we're in an internal repo, we stay undercover"*

**What this confirms**:
1. Anthropic employees actively use Claude Code to contribute to open source — and the AI is told to hide that it's an AI
2. Internal model codenames are animal names: **Capybara**, **Tengu**, and others
3. **"Tengu"** is almost certainly Claude Code's **internal project codename** — it prefixes hundreds of feature flags and analytics events

---

## Multi-Agent Orchestration — Coordinator Mode

The `coordinator/` directory contains a full multi-agent orchestration system, activated via `CLAUDE_CODE_COORDINATOR_MODE=1`. Claude Code transforms from a single agent into a **coordinator** that spawns and manages multiple worker agents in parallel.

| Phase | Who | Purpose |
|-------|-----|---------|
| **Research** | Workers (parallel) | Investigate codebase, find files, understand problem |
| **Synthesis** | Coordinator | Read all findings, craft implementation specs |
| **Implementation** | Workers | Make targeted changes per spec, commit |
| **Verification** | Workers | Test changes work |

From `coordinatorMode.ts`:

> *"Parallelism is your superpower. Workers are async. Launch independent workers concurrently whenever possible — don't serialize work that can run simultaneously."*

Workers communicate via `<task-notification>` XML messages. A shared **scratchpad directory** (gated behind `tengu_scratch`) enables cross-worker durable knowledge sharing. An **Agent Swarm** capability (`tengu_amber_flint` gate) adds in-process teammates with `AsyncLocalStorage` context isolation, tmux/iTerm2 pane-based teammates, team memory sync, and color assignments for visual distinction.

---

## Claude Code's 40+ Tool Registry

All tools live in `tools/`. Here is the complete list from `getAllBaseTools()`:

| Tool | Description |
|------|-------------|
| `AgentTool` | Spawn child agents/subagents |
| `BashTool` / `PowerShellTool` | Shell execution (with optional sandboxing) |
| `FileReadTool` / `FileEditTool` / `FileWriteTool` | File operations |
| `GlobTool` / `GrepTool` | File search (uses native `bfs`/`ugrep` when available) |
| `WebFetchTool` / `WebSearchTool` / `WebBrowserTool` | Web access |
| `NotebookEditTool` | Jupyter notebook editing |
| `SkillTool` | Invoke user-defined skills |
| `REPLTool` | Interactive VM shell (bare mode) |
| `LSPTool` | Language Server Protocol communication |
| `AskUserQuestionTool` | Prompt user for input |
| `EnterPlanModeTool` / `ExitPlanModeV2Tool` | Plan mode control |
| `BriefTool` | Upload/summarize files to claude.ai |
| `SendMessageTool` / `TeamCreateTool` / `TeamDeleteTool` | Agent swarm management |
| `TaskCreateTool` / `TaskGetTool` / `TaskListTool` / `TaskUpdateTool` / `TaskOutputTool` / `TaskStopTool` | Background task management |
| `TodoWriteTool` | Write todos (legacy) |
| `ListMcpResourcesTool` / `ReadMcpResourceTool` | MCP resource access |
| `SleepTool` | Async delays |
| `SnipTool` | History snippet extraction |
| `ToolSearchTool` | Tool discovery |
| `ListPeersTool` | List peer agents (UDS inbox) |
| `MonitorTool` | Monitor MCP servers |
| `EnterWorktreeTool` / `ExitWorktreeTool` | Git worktree management |
| `ScheduleCronTool` | Schedule cron jobs |
| `RemoteTriggerTool` | Trigger remote agents |
| `WorkflowTool` | Execute workflow scripts |
| `ConfigTool` | Modify settings (**internal/ant only**) |
| `TungstenTool` | Advanced features (**internal/ant only**) |
| `SendUserFile` / `PushNotification` / `SubscribePR` | KAIROS-exclusive tools |

Tools are filtered at runtime by feature gates, user type (`ant` vs regular), environment flags, and permission deny rules. A `toolSchemaCache.ts` caches JSON schemas for prompt efficiency.

---

## The Permission and Security System

The permission system in `tools/permissions/` is far more sophisticated than allow/deny:

**Permission Modes**: `default` (interactive prompts), `auto` (ML-based auto-approval via transcript classifier), `bypass` (skip checks), `yolo` (deny all — yes, really)

**Risk Classification**: Every tool action is classified `LOW`, `MEDIUM`, or `HIGH` risk. A **YOLO classifier** — a fast ML-based decision engine — auto-approves based on transcript analysis.

**Protected Files**: `.gitconfig`, `.bashrc`, `.zshrc`, `.mcp.json`, `.claude.json` and others are guarded from automatic editing.

**Path Traversal Prevention**: URL-encoded traversals, Unicode normalization attacks, backslash injection, case-insensitive path manipulation — all handled in code.

**Permission Explainer**: When Claude says "this command will modify your git config" — that explanation is itself generated by a separate Claude call.

---

## Hidden Beta Headers and Unreleased API Features

`constants/betas.ts` reveals every beta feature Claude Code negotiates with the Anthropic API:

```typescript
'interleaved-thinking-2025-05-14'      // Extended thinking
'context-1m-2025-08-07'                // 1M token context window
'structured-outputs-2025-12-15'        // Structured output format
'web-search-2025-03-05'                // Web search
'advanced-tool-use-2025-11-20'         // Advanced tool use
'effort-2025-11-24'                    // Effort level control
'task-budgets-2026-03-13'              // Task budget management
'prompt-caching-scope-2026-01-05'      // Prompt cache scoping
'fast-mode-2026-02-01'                 // Fast mode ("Penguin Mode")
'redact-thinking-2026-02-12'           // Redacted thinking
'token-efficient-tools-2026-03-28'     // Token-efficient tool schemas
'afk-mode-2026-01-31'                  // AFK mode
'cli-internal-2026-02-09'              // Internal-only (ant users)
'advisor-tool-2026-03-01'              // Advisor tool
'summarize-connector-text-2026-03-13'  // Connector text summarization
```

`redact-thinking`, `afk-mode`, and `advisor-tool` are not yet publicly documented.

---

## Feature Gating — Internal vs External Builds

Claude Code uses **compile-time feature flags** via Bun's `feature()` function. The bundler constant-folds and dead-code-eliminates gated branches from external builds. Source maps, however, don't care about dead code elimination — which is exactly why this leak exposed everything.

| Flag | What It Gates |
|------|--------------|
| `PROACTIVE` / `KAIROS` | Always-on assistant mode |
| `KAIROS_BRIEF` | Brief command |
| `BRIDGE_MODE` | Remote control via claude.ai |
| `DAEMON` | Background daemon mode |
| `VOICE_MODE` | Voice input |
| `WORKFLOW_SCRIPTS` | Workflow automation |
| `COORDINATOR_MODE` | Multi-agent orchestration |
| `TRANSCRIPT_CLASSIFIER` | AFK mode (ML auto-approval) |
| `BUDDY` | Companion pet system |
| `NATIVE_CLIENT_ATTESTATION` | Client authenticity checks |
| `HISTORY_SNIP` | History snipping |
| `EXPERIMENTAL_SKILL_SEARCH` | Skill discovery |

`USER_TYPE === 'ant'` additionally gates: staging API access (`claude-ai.staging.ant.dev`), internal beta headers, Undercover Mode, the `/security-review` command, `ConfigTool`, `TungstenTool`, and debug prompt dumping to `~/.config/claude/dump-prompts/`.

---

## Other Notable Findings

### Fast Mode is Internally Called "Penguin Mode"

The API endpoint in `utils/fastMode.ts` is literally:

```typescript
const endpoint = `${getOauthConfig().BASE_API_URL}/api/claude_code_penguin_mode`
```

Config key: `penguinModeOrgEnabled`. Kill-switch: `tengu_penguins_off`. Analytics event: `tengu_org_penguin_mode_fetch_failed`. Penguins all the way down.

### Computer Use — "Chicago"

Claude Code includes a full Computer Use implementation, internally codenamed **"Chicago"**, built on `@ant/computer-use-mcp`. Provides screenshot capture, click/keyboard input, and coordinate transformation. Gated to Max/Pro subscriptions (with an `ant` bypass for internal users).

### Internal Codename History from Migrations

The `migrations/` directory reveals model codename history:
- `migrateFennecToOpus` — **Fennec** (the fox) was an Opus codename
- `migrateSonnet1mToSonnet45` — Sonnet with 1M context became Sonnet 4.5
- `migrateSonnet45ToSonnet46` — Sonnet 4.5 → Sonnet 4.6
- `resetProToOpusDefault` — Pro users were reset to Opus at some point

### The Upstream Proxy

`upstreamproxy/` contains a container-aware proxy relay that uses **`prctl(PR_SET_DUMPABLE, 0)`** to prevent same-UID ptrace of heap memory. Reads session tokens from `/run/ccr/session_token`. Anthropic API, GitHub, npmjs.org, and pypi.org are explicitly excluded from proxying.

### Attribution Header

Every API request from Claude Code includes:

```
x-anthropic-billing-header: cc_version={VERSION}.{FINGERPRINT};
  cc_entrypoint={ENTRYPOINT}; cch={ATTESTATION_PLACEHOLDER}; cc_workload={WORKLOAD};
```

The `NATIVE_CLIENT_ATTESTATION` feature lets Bun's HTTP stack overwrite `cch=00000` with a computed hash — a client authenticity check so Anthropic can verify the request came from a real Claude Code install.

### The Cyber Risk Instruction

`constants/cyberRiskInstruction.ts` has a prominent warning:

```
IMPORTANT: DO NOT MODIFY THIS INSTRUCTION WITHOUT SAFEGUARDS TEAM REVIEW
This instruction is owned by the Safeguards team (David Forsythe, Kyla Guru)
```

This instruction draws clear lines: authorized security testing is fine, destructive techniques and supply chain compromise are not.

---

## Security Implications of the Claude Code Source Leak

This leak is not a data breach in the traditional sense — no user data was exposed. The implications are architectural:

1. **Full internal architecture is now public**: system prompt structure, caching strategy, permission model, and agent orchestration design are all visible to competitors and adversaries
2. **Internal codenames revealed**: Tengu (Claude Code's project name), Capybara, Fennec, Chicago — all now mappable to public model/feature names
3. **Undercover Mode exposed**: confirms Anthropic employees use Claude Code to contribute to open-source repos while the AI actively hides its AI authorship
4. **Security boundary owners named**: the Safeguards team members responsible for the `CYBER_RISK_INSTRUCTION` are now publicly identified
5. **Source maps as an attack vector**: this is not novel but serves as a high-profile reminder — any npm package built with sourcemaps enabled ships its full source

The leak vector (unstripped source maps in an npm package) is a routine DevOps mistake. The lesson for every team shipping proprietary TypeScript: always add `*.map` to `.npmignore` and explicitly set `sourcemap: false` for production builds.

---

## FAQ

**What version of Claude Code was leaked?**
Version `v2.1.88` of `@anthropic-ai/claude-code` on npm. The source was exposed via the `.map` sourcemap file bundled in the published package.

**How was Claude Code's source code leaked?**
A sourcemap file (`.map`) was included in the npm package. Sourcemaps embed the complete original source code as strings inside a JSON structure. Anyone with access to the npm registry could download and extract the full source.

**Who discovered the Claude Code npm leak?**
Chaofan Shou ([@Fried_rice on X](https://x.com/fried_rice/status/2038894956459290963)), a PhD researcher at UC Berkeley's Sky Computing Lab and co-founder of FuzzLand. His tweet on March 31, 2026 received 4.5M+ views.

**What programming language is Claude Code written in?**
TypeScript, compiled and bundled with Bun's bundler. The terminal UI uses React with the Ink library for terminal rendering.

**What is BUDDY in Claude Code?**
BUDDY is a hidden Tamagotchi-style companion pet system gated behind a compile-time feature flag. It features 18 species with rarity tiers (Common through Legendary), procedurally generated stats, and a personality written by Claude on first hatch. It was scheduled for teaser release April 1–7, 2026 with full launch in May 2026.

**What is KAIROS in Claude Code?**
KAIROS is an unreleased "always-on" mode where Claude Code runs persistently in the background, watching your workflow and proactively acting on things it notices — without waiting for you to type a command.

**What does Undercover Mode do?**
It prevents Claude Code from revealing Anthropic-internal information (model codenames, internal tooling, Slack channels, the fact that the author is an AI) in commits and PR descriptions when Anthropic employees contribute to public open-source repositories.

**What is Claude Code's internal project codename?**
Almost certainly **"Tengu"** — the word prefixes hundreds of feature flags and analytics events throughout the source (`tengu_ultraplan_model`, `tengu_penguins_off`, `tengu_scratch`, `tengu_amber_flint`, etc.).

**What internal model codenames were revealed?**
**Capybara**, **Fennel/Fennec** (an Opus variant), and **Tengu** (Claude Code itself). The Undercover Mode instruction also references generic "animal names" as the naming convention for unreleased models.

**Is this repo legal to keep up?**
This repository is maintained for **educational and research purposes only** — analyzing architecture, security practices, and AI system design. The leak vector (unstripped npm sourcemaps) is a widely documented security anti-pattern. No private user data was exposed.

---

## References & Further Reading

- [Original discovery tweet by @Fried_rice](https://x.com/fried_rice/status/2038894956459290963) — March 31, 2026
- [Anthropic's Claude Code Source Code Reportedly Leaked — CyberSecurityNews](https://cybersecuritynews.com/claude-code-source-code-leaked/)
- [Claude Code Source Leaked via npm: 512K Lines Exposed — ByteIota](https://byteiota.com/claude-code-source-leaked-via-npm-512k-lines-exposed/)
- [Anthropic's Claude Code source leaked via npm registry — PiunikaWeb](https://piunikaweb.com/2026/03/31/anthropic-claude-code-source-leaked-npm-registry/)
- [Hacker News discussion thread](https://news.ycombinator.com/item?id=47584540)
- [Affected npm package: @anthropic-ai/claude-code](https://www.npmjs.com/package/@anthropic-ai/claude-code)

---

## Disclaimer

This repository is an **educational archive** created for research purposes: studying AI system architecture, source map security risks, and software engineering practices. All analysis is based on publicly accessible data that was unintentionally made available via a standard npm registry. No proprietary data was actively exfiltrated. If you are from Anthropic's legal team, please open an issue.

---

*Archived by [@soufianebouaddis](https://github.com/soufianebouaddis) · March 31, 2026 · Star ⭐ to help others find this analysis*
