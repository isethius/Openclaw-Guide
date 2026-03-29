# CLI Subscription Auth and Coding Tool Paths

> **This guide documents the real operational coding workflows in this OpenClaw environment** — subscription-backed CLI auth/setup, direct CLI coding workflows, and the critical distinction between CLI execution and API model inference. It describes the CLI and model paths actually used here, not every possible OpenClaw integration path.

> ⚠️ **Version-sensitive content.** CLI flags, auth behavior, provider model strings, and OpenClaw integration details may change between versions. Commands and patterns here are verified for this environment as of March 2026. Treat them as current working knowledge, not eternal guarantees — re-verify after major updates. Specific items to watch:
> - CLI flag names (`--full-auto`, `--permission-mode bypassPermissions`)
> - Provider model strings (`openai-codex/gpt-5.4`, `anthropic/claude-opus-4-6`)
> - Auth expiry behavior and timescales
> - ACP bridge command syntax
> - `claude auth status` availability (version-dependent)

---

## 1. Purpose and Scope

This guide covers:

- **Subscription-backed CLI auth and setup** — how Codex CLI and Claude Code CLI authenticate
- **Direct CLI coding workflows** — what the OpenClaw agent actually runs when it needs to write or debug code
- **Distinction between CLI execution and API model inference** — two fundamentally different mechanisms that share names but not behavior
- **Operational reality of this environment** — what's primary, what's available, what's backup
- **Version-sensitive nature of CLI/auth behavior** — where to expect breakage over time

This guide does **not** cover:
- Full OpenClaw installation (see [Hetzner Setup Guide](hetzner-setup-combined.md))
- Telegram/channel configuration (see [Post-Setup Guide](post-setup-guide.md))
- General model selection and provider config (see [Post-Setup Guide](post-setup-guide.md))

---

## 2. Two Different Concepts: CLI Execution vs API Inference

This is the most important framing in this document. These two mechanisms share names (Codex, Claude) but are completely different in how they work, what auth they use, and what they produce.

| | CLI Execution | API Model Inference |
|---|---|---|
| **Mechanism** | `exec` tool → shell → real binary | Gateway → model selection → API call |
| **What runs** | A real program on the VPS | A remote API endpoint |
| **Auth** | The CLI binary's own login/tokens | Gateway-managed API keys or OAuth |
| **Binary required** | Yes — must be installed on VPS | No — only needs network + credentials |
| **Agent interaction** | Agent runs a shell command, reads stdout/stderr | Agent sends a prompt, receives a response |
| **Analogy** | Running `git commit` or `npm test` | Calling the OpenAI API |

**Keep these separate in your mental model.** When someone says "use Codex," they might mean either one — and the failure modes, auth requirements, and operational behavior are completely different.

---

## 3. Real Operational Paths in This Environment

### A. Codex CLI — Primary Coding Workflow

This is the path that has handled **most actual coding work** in this environment.

```
OpenClaw agent
  → exec tool
    → codex --full-auto exec 'task'
      → real codex binary on the VPS
```

**How it works:**
1. The OpenClaw agent decides it needs to write, edit, or debug code
2. It uses the `exec` tool to run a shell command
3. The shell invokes the **real Codex CLI binary** installed on the VPS
4. Codex CLI executes the coding task autonomously (`--full-auto` mode)
5. Results return to the agent via stdout/stderr

**Key details:**
- This is **direct CLI execution** — the agent is running a program, same as `git`, `npm`, or `python`
- Codex CLI handles its own auth (subscription/OAuth-style), separate from the OpenClaw gateway
- `--full-auto` tells Codex to execute without interactive confirmation — essential for agent-driven use
- The Codex binary must be installed on the VPS for this path to work

---

### B. Codex-Backed API Fallback — Separate Path

This is a **completely different mechanism** from the Codex CLI workflow above. No CLI binary is involved.

```
OpenClaw gateway
  → model selection (primary unavailable or fallback configured)
    → openai-codex/... model string
      → API call using Codex-backed auth/tokens
        → response returned to agent
```

**How it works:**
1. The OpenClaw gateway needs to route a prompt to a model
2. The primary model (e.g., `anthropic/claude-opus-4-6`) is unavailable or rate-limited
3. The gateway falls back to a Codex-backed model string (e.g., `openai-codex/gpt-5.4`)
4. An API call is made using the configured auth tokens
5. The response comes back through the gateway's normal model pipeline

**Key details:**
- **No CLI binary is executed** — this is pure API inference through the gateway
- Auth is managed by the gateway — OAuth tokens, API keys in `openclaw.json` or SecretRefs
- This is the same mechanism as any other model (Anthropic, OpenAI, Google) — just with a Codex-backed endpoint
- This is **model inference, not tool execution**

> ⚠️ **Shared auth dependency:** In this environment, Codex CLI execution (path A) and Codex-backed API fallback (this path) appear to depend on the same Codex auth state. If that login expires, **both paths may fail at the same time.** See [Auth Model § Codex Auth](#codex-auth) for details.

---

### C. Claude Code CLI — Verified and Available

Claude Code CLI is **installed, verified, and available** on this VPS. It is **not currently the main real-world coding workflow** — in our experience, most actual coding work has gone through Codex CLI.

```
OpenClaw agent
  → exec tool
    → claude --print --permission-mode bypassPermissions 'task'
      → real claude binary on the VPS
```

**How it works:**
1. The OpenClaw agent uses the `exec` tool to run a shell command
2. The shell invokes the **real Claude Code CLI binary** on the VPS
3. `--print` outputs results to stdout (non-interactive mode)
4. `--permission-mode bypassPermissions` skips interactive permission prompts
5. Results return to the agent via stdout/stderr

**Key details:**
- Same mechanism as Codex CLI — direct CLI execution via `exec`, not API inference
- Claude Code CLI handles its own auth/login, separate from the OpenClaw gateway
- The binary is verified and available on the VPS
- Current evidence shows it is available but not the main deployed coding path

**Current status:**

| Aspect | Status |
|--------|--------|
| Binary installed | ✅ Yes |
| Verified working | ✅ Yes |
| Primary coding path | ❌ No — Codex CLI is primary |
| Available for use | ✅ Yes — ready if needed |

---

### D. Claude API Inference — Primary Model Inference Path

This is the **primary model** the OpenClaw gateway uses for general inference. It is not CLI execution.

```
OpenClaw gateway
  → model selection
    → anthropic/claude-opus-4-6
      → direct Anthropic API call
        → response returned to agent
```

**Key details:**
- **No CLI binary is involved** — this is a direct API call to Anthropic
- Auth is via Anthropic API key, managed by the gateway (in `openclaw.json` or SecretRefs)
- This is the same mechanism as any other API model — standard gateway model routing
- This is **completely separate** from Claude Code CLI execution (path C above)

---

## 4. ACP Status

ACP (Agent Communication Protocol) is a structured protocol for agent-to-tool communication. Its current status in this environment:

| Aspect | Status |
|--------|--------|
| Enabled/configured | ✅ Yes |
| Tested | ✅ Yes |
| Deprecated | ❌ No |
| Current preferred workflow | ❌ No |
| Operational role | Backup / experimental / future option |

**Key points:**
- ACP is **not the current day-to-day coding workflow** — direct CLI execution proved more reliable in this environment
- ACP is **not equivalent to direct CLI execution** — it's a different mechanism with protocol overhead
- ACP is **not deprecated** — it's retained as a backup and may be useful for future tool integrations that benefit from structured protocol negotiation
- Do not document or present ACP as the main coding workflow

---

## 5. Verification Commands

### Codex CLI

```bash
which codex          # Is the binary installed?
codex --version      # What version?
codex --help         # Available flags
```

Example verified coding invocation:
```bash
codex --full-auto exec 'describe what files are in this directory'
```

### Claude Code CLI

```bash
which claude         # Is the binary installed?
claude --version     # What version?
claude auth status   # Check auth state, if supported by your installed version
```

Example verified invocation pattern:
```bash
claude --print --permission-mode bypassPermissions 'describe what files are in this directory'
```

---

## 6. Auth Model

### Codex Auth

Codex CLI uses **subscription/OAuth-style auth**. In this environment:

- Codex CLI authenticates via its own login flow — this is separate from OpenClaw gateway auth
- The Codex-backed API fallback model (used by the gateway) is tied to this same auth state
- Codex-backed auth can expire and should be monitored or refreshed proactively
- In practice, Codex-backed auth appears to expire on a timescale of days, not hours, and should be monitored or refreshed proactively

> ⚠️ **Operational warning — shared auth single point of failure:**
> In this environment, Codex CLI execution and Codex-backed API fallback appear to depend on the same Codex auth state. If that login expires, **both the direct Codex CLI coding workflow and the Codex-backed model fallback may fail at the same time.** Monitor auth status proactively and set up a refresh mechanism rather than discovering the failure during production use.

**Mitigation:**
- Set up a monitoring check or refresh cron for Codex auth
- Do not assume Codex CLI working means Codex API fallback is also healthy (or vice versa) — they share auth but may surface failures differently
- When Codex auth fails, check both paths before assuming only one is affected

### Claude Code Auth

Claude Code CLI uses **its own CLI auth/login flow**. This is entirely separate from Anthropic API inference auth.

| Auth Type | Mechanism | Used By |
|-----------|-----------|---------|
| Claude Code CLI auth | CLI login flow (`claude auth`) | Claude Code CLI binary (path C) |
| Anthropic API key | API key in `openclaw.json` / SecretRefs | Gateway model inference (path D) |

**Key distinction:**
- Having a valid Anthropic API key in your OpenClaw config does **not** mean Claude Code CLI is authenticated
- Having Claude Code CLI logged in does **not** mean your Anthropic API key is valid
- These are independent auth systems that happen to involve the same provider

---

## 7. Configuration Patterns

### Model Primary/Fallback

```json5
// In openclaw.json — model section
{
  agents: {
    defaults: {
      model: {
        primary: "anthropic/claude-opus-4-6",
        fallbacks: ["openai-codex/gpt-5.4"]
      }
    }
  }
}
```

This configures:
- Primary model → Anthropic API inference (path D)
- Fallback model → Codex-backed API inference (path B)

This does **not** configure direct CLI execution (paths A or C). Those are invoked by the agent via `exec`, not through gateway model routing.

### ACP Configuration (Reference Only)

ACP is an **external bridge**, not a JSON config toggle. The `openclaw acp` command connects an IDE or external client to a running OpenClaw Gateway via WebSocket. Configuration lives in the IDE/client, not in `openclaw.json`.

Example IDE integration (Zed):
```json
{
  "agent_servers": {
    "OpenClaw ACP": {
      "type": "custom",
      "command": "openclaw",
      "args": ["acp", "--url", "ws://127.0.0.1:18789"]
    }
  }
}
```

> This is included for reference. ACP is configured but is not the primary coding workflow in this environment. Run `openclaw acp --help` to confirm available flags and options.

### What Is NOT Included Here

This guide intentionally omits:
- Live secrets or token values
- Token file structures or field layouts
- Internal secrets storage layouts
- Paths under library internals
- Exact internal heuristics for tool selection

These are internal implementation details that should not appear in a guide.

---

## 8. Common Mistakes to Avoid

### ❌ Confusing CLI execution with API inference

**Wrong:** "Codex can be used as a coding tool or a fallback model."
**Right:** "Codex CLI is a binary you execute via `exec`. Codex model fallback is an API endpoint the gateway routes to. They share a name but are completely different mechanisms."

### ❌ Presenting ACP as the main coding path

**Wrong:** "OpenClaw uses ACP to connect to Codex/Claude for coding tasks."
**Right:** "OpenClaw uses `exec` to run Codex CLI directly. ACP is configured but is a backup/experimental option."

### ❌ Treating Codex fallback as the same thing as Codex CLI

**Wrong:** "The Codex fallback runs the Codex binary when Claude is unavailable."
**Right:** "The Codex fallback is an API call through the gateway — no CLI binary is involved. It's model inference, not tool execution."

### ❌ Assuming Claude Code CLI is a major production coding path here

**Wrong:** "OpenClaw uses Claude Code and Codex interchangeably for coding."
**Right:** "Codex CLI is the primary coding path. Claude Code CLI is verified and available but is not currently the main operational coding workflow."

### ❌ Conflating Anthropic API inference with Claude Code CLI auth

**Wrong:** "Claude is authenticated because the Anthropic API key works."
**Right:** "Anthropic API key auth (for model inference) and Claude Code CLI auth (for `claude` binary execution) are completely separate systems."

---

## 9. Quick Reference Table

| Path | Mechanism | Auth | Binary Required | Current Role |
|------|-----------|------|-----------------|--------------|
| Codex CLI | `exec` → shell → `codex` | Codex subscription/OAuth | Yes (`codex`) | ★ Primary coding |
| Claude Code CLI | `exec` → shell → `claude` | Claude CLI login | Yes (`claude`) | Verified, available |
| Codex model fallback | Gateway → API call | Gateway-managed (shared w/ Codex CLI) | No | Model fallback |
| Claude API inference | Gateway → API call | Anthropic API key | No | Primary model |
| ACP | Structured protocol (`openclaw acp`) | Varies | Varies | Backup / experimental |

> ⚠️ **Reminder — shared Codex auth:** Codex CLI and Codex model fallback share auth state. If that login expires, both the primary coding path and the model fallback may break at the same time. Monitor proactively.

---

## 10. Operational Path Diagram

```
┌─────────────────────────────────────────────────────────────┐
│               CLI EXECUTION (exec tool)                      │
│          Agent runs a real binary on the VPS                 │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  A. Codex CLI ★ PRIMARY CODING PATH                        │
│     exec → codex --full-auto exec 'task'                    │
│     Real binary · Own auth · Most production coding use     │
│                                                             │
│  C. Claude Code CLI (verified, available)                    │
│     exec → claude --print --permission-mode ... 'task'      │
│     Real binary · Own auth · Ready but not primary          │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│            API MODEL INFERENCE (gateway)                     │
│      Gateway routes prompts to remote model endpoints        │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  B. Codex model fallback                                     │
│     Gateway → openai-codex/... → API call                   │
│     No CLI binary · Gateway-managed auth · Model routing    │
│                                                             │
│  D. Claude API inference (primary model)                     │
│     Gateway → anthropic/claude-opus-4-6 → API call          │
│     No CLI binary · Anthropic API key · Standard routing    │
│                                                             │
└─────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────┐
│            ACP (backup / experimental)                       │
│       Structured protocol · Not the current workflow         │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  E. ACP integration                                          │
│     Enabled · Tested · Retained as backup                    │
│     Not primary · Not equivalent to CLI or API inference     │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

---

## 11. Public vs Internal Notes

### ✅ Safe to document publicly
- CLI invocation patterns (flags, command structure)
- Auth model distinctions (Codex vs Claude, CLI vs API)
- ACP status (enabled, not primary)
- Verification commands (`which`, `--version`, `--help`)
- Configuration shape (model primary/fallback, ACP enabled)

### ⚠️ Version-sensitive — re-verify before publishing
- See the version-sensitive warning at the top of this document for the full list

### 🔒 Internal-only — omit from public guide
- Token values and credential content
- Token file formats and field layouts
- Internal secrets storage paths and structure
- Exact heuristics for tool/model selection
- Private internal module paths

---

> **This document reflects the operational reality of this specific OpenClaw environment.** It does not represent every possible OpenClaw integration or configuration. Other environments may use different tools, different primary paths, or rely more heavily on ACP.

*Last updated: 2026-03-29*
