# OpenClaw Coding Tool Invocation Model — Corrected Reference

> **Version 14** — Corrected to reflect real-world operational paths as of March 2026.

> **Key correction:** The real coding workflow is **direct CLI execution via the `exec` tool**, not ACP-first. This document separates CLI execution from API model inference — two fundamentally different mechanisms that must not be blurred.

---

## Two Distinct Concepts

Before diving into the workflow paths, understand that OpenClaw interacts with external coding tools in **two completely separate ways**:

| Concept | Mechanism | What Happens |
|---------|-----------|--------------|
| **CLI Execution** | `exec` tool → shell → real binary | The agent runs a real CLI program on the VPS. The binary does the work. |
| **API Model Inference** | Gateway → model selection → API call | The gateway routes a prompt to a remote model endpoint. No CLI binary is involved. |

These are not variations of the same thing. They use different infrastructure, different auth, and produce different results. Keep them separate in your mental model.

---

## Workflow 1: Codex CLI (Primary Real-World Coding Path)

This is the path that has handled **most actual coding work** in production.

```
OpenClaw agent
  → exec tool
    → codex --full-auto exec 'task'
      → real codex binary on the VPS
```

### How it works

1. The OpenClaw agent decides it needs to write, edit, or debug code
2. It uses the `exec` tool to run a shell command
3. The shell invokes the **real Codex CLI binary** installed on the VPS
4. Codex CLI executes the coding task autonomously (`--full-auto` mode)
5. Results return to the agent via stdout/stderr

### Key details

- **This is direct CLI execution.** The agent is running a program, same as running `git`, `npm`, or `python`.
- **Codex CLI handles its own auth.** It uses its own API tokens/OAuth, not the OpenClaw gateway's model routing.
- **`--full-auto`** tells Codex to execute without interactive confirmation — essential for agent-driven use.
- **The Codex binary must be installed on the VPS.** If it's not there, this path doesn't work.

### Verification

```bash
which codex          # Is the binary installed?
codex --version      # What version?
codex --help         # Available flags
```

---

## Workflow 2: Codex Model Fallback (Separate Path — API Inference)

This is a **completely different mechanism** from Workflow 1. No CLI binary is involved.

```
OpenClaw gateway
  → model selection (primary fails or fallback configured)
    → openai-codex/... model string
      → API call using Codex-backed auth/tokens
        → response returned to agent
```

### How it works

1. The OpenClaw gateway needs to route a prompt to a model
2. The primary model (e.g., `anthropic/claude-opus-4-6`) is unavailable or rate-limited
3. The gateway falls back to a Codex-backed model string (e.g., `openai-codex/gpt-5.4`)
4. An API call is made using the configured auth tokens
5. The response comes back through the gateway's normal model pipeline

### Key details

- **No CLI binary is executed.** This is pure API inference through the gateway.
- **Auth is managed by the gateway** — OAuth tokens, API keys in `openclaw.json` or SecretRefs.
- **This is the same mechanism as any other model** (Anthropic, OpenAI, Google) — just with a Codex-backed endpoint.
- **Codex-backed auth can expire and should be refreshed proactively** to avoid unexpected fallback failures. Set up a monitoring check or refresh cron rather than discovering it's dead at 2am.

> ⚠️ **Shared auth dependency:** In this environment, Codex CLI execution (Workflow 1) and Codex-backed API fallback (this workflow) appear to depend on the same Codex auth state. If that login expires, both paths may fail at the same time.

### Configuration

```json5
// In openclaw.json
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

---

## Workflow 3: Claude Code CLI (Verified, Not Primary)

Claude Code CLI is **installed, verified, and available** — but it is **not currently the primary real-world coding path.** Most actual coding work has gone through Codex CLI.

```
OpenClaw agent
  → exec tool
    → claude --print --permission-mode bypassPermissions 'task'
      → real claude binary on the VPS
```

### How it works

1. The OpenClaw agent uses the `exec` tool to run a shell command
2. The shell invokes the **real Claude Code CLI binary** on the VPS
3. `--print` outputs results to stdout (non-interactive mode)
4. `--permission-mode bypassPermissions` skips interactive permission prompts
5. Results return to the agent via stdout/stderr

### Key details

- **Same mechanism as Codex CLI** — direct CLI execution via `exec`, not API inference.
- **Claude Code CLI handles its own auth** — separate from the OpenClaw gateway.
- **The binary is verified and available** on the VPS.
- **It is not the current preferred coding workflow** — retained as a ready alternative.

### Verification

```bash
which claude         # Is the binary installed?
claude --version     # What version?
claude --help        # Available flags
```

### Current status

| Aspect | Status |
|--------|--------|
| Binary installed | ✅ Yes |
| Verified working | ✅ Yes |
| Primary coding path | ❌ No — Codex CLI is primary |
| Available for use | ✅ Yes — ready if needed |

---

## Workflow 4: ACP (Backup / Experimental)

ACP (Agent Communication Protocol) is a **separate integration mechanism** for connecting OpenClaw to external coding tools. It is enabled and was tested, but it is **not the current preferred workflow**.

### Current ACP status

| Aspect | Status |
|--------|--------|
| Enabled/configured | ✅ Yes |
| Tested | ✅ Yes |
| Current preferred workflow | ❌ No |
| Operational role | Backup / experimental option |

### What ACP does differently

ACP provides a structured protocol for agent-to-tool communication, as opposed to raw CLI execution. In theory, this enables richer interaction (streaming, structured output, tool negotiation). In practice:

- The direct CLI paths (Workflows 1 and 3) have been more reliable and battle-tested
- ACP adds protocol overhead without clear operational benefit for current use cases
- ACP remains available if future tool integrations require it

### What ACP is NOT

- **Not the primary coding path** — do not document it as such
- **Not equivalent to CLI execution** — it's a different mechanism entirely
- **Not deprecated** — it's retained and functional, just not preferred

---

## Summary: What Actually Runs Today

```
┌─────────────────────────────────────────────────────────┐
│              CLI EXECUTION (exec tool)                   │
│         Agent runs a real binary on the VPS              │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  1. Codex CLI ★ PRIMARY                                 │
│     exec → codex --full-auto exec 'task'                │
│     Real binary, own auth, most production use          │
│                                                         │
│  2. Claude Code CLI (verified, available)                │
│     exec → claude --print --permission-mode ... 'task'  │
│     Real binary, own auth, ready but not primary        │
│                                                         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│           API MODEL INFERENCE (gateway)                  │
│     Gateway routes prompts to remote model endpoints     │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  3. Codex model fallback                                 │
│     Gateway → openai-codex/... → API call               │
│     No CLI binary, gateway-managed auth, model routing  │
│                                                         │
└─────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────┐
│           ACP (backup / experimental)                    │
│      Structured protocol, not the current workflow       │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  4. ACP integration                                      │
│     Enabled, tested, retained as backup                  │
│     Not primary, not equivalent to CLI or API inference  │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

---

## Common Mistakes to Avoid

### ❌ Don't blur CLI execution with API model inference

**Wrong:** "Codex can be used as a coding tool or a fallback model."
**Right:** "Codex CLI is a binary you execute via `exec`. Codex model fallback is an API endpoint the gateway routes to. They share a name but are completely different mechanisms."

### ❌ Don't present ACP as the primary coding path

**Wrong:** "OpenClaw uses ACP to connect to Codex/Claude for coding tasks."
**Right:** "OpenClaw uses `exec` to run Codex CLI directly. ACP is configured but is a backup/experimental option."

### ❌ Don't assume Claude Code CLI is equivalent to Codex CLI in current usage

**Wrong:** "OpenClaw can use either Codex or Claude Code interchangeably for coding."
**Right:** "Codex CLI is the primary coding path. Claude Code CLI is verified and available but has not been the primary operational workflow."

### ❌ Don't conflate gateway model routing with tool execution

**Wrong:** "When OpenClaw needs to code, it calls the Codex model."
**Right:** "When OpenClaw needs to code, the agent runs `codex` via `exec`. Separately, the gateway may fall back to a Codex-backed model for general inference — that's model routing, not coding tool execution."

---

## Quick Reference

| Path | Mechanism | Auth | Binary Required | Current Role |
|------|-----------|------|-----------------|--------------|
| Codex CLI | `exec` → shell → `codex` | Codex's own tokens | Yes (`codex`) | ★ Primary coding |
| Claude Code CLI | `exec` → shell → `claude` | Claude's own auth | Yes (`claude`) | Verified, available |
| Codex model fallback | Gateway → API call | Gateway-managed | No | Model fallback |
| ACP | Structured protocol | Varies | Varies | Backup / experimental |

---

*Last updated: 2026-03-29*
