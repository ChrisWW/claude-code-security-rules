# Claude Code Security Rules

A comprehensive set of security rules for [Claude Code](https://code.claude.com) — specifically designed for use with `--dangerously-skip-permissions` mode (`bypassPermissions`).

When you run Claude Code without its built-in permission prompts, **these rules become your safety net.** They prevent destructive file operations, accidental secret exposure, data exfiltration, cloud infrastructure damage, and dozens of other scenarios that could ruin your day.

---

## TL;DR — 3 Steps

```bash
# 1. Copy security.md to your Claude Code config
curl -o ~/.claude/security.md \
  https://raw.githubusercontent.com/ChrisWW/claude-code-security-rules/main/security.md

# 2. Tell Claude Code to always follow these rules
echo '> **MANDATORY**: Always read and follow `~/.claude/security.md` before executing any commands.' \
  >> ~/.claude/CLAUDE.md

# 3. Run Claude Code (with or without bypass mode)
claude --dangerously-skip-permissions
```

---

## Why You Need This

Claude Code has a [built-in permission system](https://code.claude.com/docs/en/security) that asks for confirmation before risky operations. However, there are scenarios where you might bypass it:

- **`--dangerously-skip-permissions`** flag — skips all permission prompts
- **`bypassPermissions` mode** — configured in settings for CI/CD, automation, or power-user workflows
- **Non-interactive pipelines** — running Claude Code with the `-p` flag in scripts

In these cases, Claude Code can execute **any command without asking**. The official docs are clear:

> *"Claude Code only has the permissions you grant it. You're responsible for reviewing proposed code and commands for safety before approval."*

These security rules act as a behavioral guardrail — Claude Code reads them from your configuration and follows them as strict operational constraints, even when permission prompts are disabled.

---

## What's Covered

| # | Category | What It Prevents |
|---|----------|-----------------|
| 1 | **File System Protection** | `rm -rf /`, `dd` to disk, truncation, symlink escapes, permission changes |
| 2 | **Git Safety** | Force pushes, hard resets, history rewrites, secret commits |
| 3 | **Secrets & Credentials** | Exposure in logs/output, reading `.env`/keys/tokens, exfiltration |
| 4 | **Network & Remote Execution** | `curl \| sh`, data exfiltration, reverse shells, DNS tunneling |
| 5 | **Database Safety** | `DROP TABLE`, unfiltered `DELETE`, schema changes in production |
| 6 | **Process & System Safety** | Fork bombs, resource exhaustion, system config changes, macOS-specific dangers |
| 7 | **Container, Cloud & Infra** | Docker privileged mode, `terraform destroy`, AWS/GCP/Azure destructive commands |
| 8 | **Code Safety** | SQL injection, XSS, command injection, TLS downgrade, weak crypto |
| 9 | **SSH & Remote Access** | Key injection, config modification, unauthorized tunnels |
| 10 | **Prompt Injection Defense** | Malicious instructions in files, obfuscated payloads, build script exploits |
| 11 | **Destructive Operations Protocol** | Mandatory confirmation workflow for any irreversible action |
| 12 | **Scope Discipline** | Prevents unrelated modifications, over-engineering, scope creep |

---

## Quick Start

### 1. Copy `security.md` to your Claude Code config

```bash
# Download the security rules
curl -o ~/.claude/security.md https://raw.githubusercontent.com/ChrisWW/claude-code-security-rules/main/security.md
```

Or clone the repo and copy manually:

```bash
git clone https://github.com/ChrisWW/claude-code-security-rules.git
cp claude-code-security-rules/security.md ~/.claude/security.md
```

### 2. Add a reference in your `CLAUDE.md`

Add the following line to your `~/.claude/CLAUDE.md` (global rules) or to your project's `CLAUDE.md`:

```markdown
> **MANDATORY**: Always read and follow `~/.claude/security.md` before executing any commands.
```

If you don't have a `CLAUDE.md` yet:

```bash
echo '> **MANDATORY**: Always read and follow `~/.claude/security.md` before executing any commands.' > ~/.claude/CLAUDE.md
```

### 3. Run Claude Code

```bash
# Standard mode (built-in permissions + your security rules)
claude

# Bypass permissions mode (your security rules are the primary safety layer)
claude --dangerously-skip-permissions
```

That's it. Claude Code reads `CLAUDE.md` on every session start and will follow the security rules as strict constraints.

---

## Setup Options

### Global Setup (all projects)

Place the files in your user-level Claude Code config:

```
~/.claude/
├── CLAUDE.md          # Add the mandatory reference line
└── security.md        # The security rules
```

This applies to every Claude Code session regardless of project.

### Per-Project Setup

Place the files in your project root:

```
your-project/
├── CLAUDE.md          # Add the mandatory reference line
├── .claude/
│   └── settings.json  # Optional: permission settings
└── security.md        # The security rules
```

Update the reference path in `CLAUDE.md` accordingly:

```markdown
> **MANDATORY**: Always read and follow `security.md` before executing any commands.
```

### Combined (Recommended)

Use global rules as baseline, override per-project as needed:

```bash
# Global baseline
cp security.md ~/.claude/security.md

# Project-specific additions (optional)
cp security.md ./your-project/security.md
# Edit project copy to add/remove rules specific to that project
```

---

## Understanding Claude Code Permission Modes

Claude Code supports several permission modes that determine how it handles tool approval:

| Mode | Description | Security Rules Importance |
|------|-------------|--------------------------|
| `default` | Prompts for permission on first use of each tool | Low — built-in prompts protect you |
| `acceptEdits` | Auto-accepts file edits, still prompts for commands | Medium — edits are unrestricted |
| `plan` | Read-only, no modifications allowed | Low — inherently safe |
| `dontAsk` | Auto-denies unless pre-approved via allowlist | Medium — depends on your allowlist |
| `bypassPermissions` | **Skips all prompts** | **Critical — these rules are your only guardrail** |

### The `--dangerously-skip-permissions` Flag

```bash
claude --dangerously-skip-permissions
```

This flag enables `bypassPermissions` mode. Claude Code will:
- Execute **any bash command** without asking
- **Edit and create any file** without asking
- Make **network requests** without asking
- Run **build scripts, install packages, modify configs** without asking

**When to use it:**
- Local development in isolated/sandboxed environments
- CI/CD pipelines with controlled scope
- Power-user workflows where permission fatigue blocks productivity
- Trusted projects where you've reviewed the codebase

**When NOT to use it:**
- Working with untrusted codebases (prompt injection risk)
- Production servers or shared systems
- Projects with access to cloud credentials or infrastructure
- When you haven't set up security rules

### Additional Protection: Hooks

Claude Code supports [hooks](https://code.claude.com/docs/en/hooks-guide) — custom shell commands that run before/after tool calls. You can use `PreToolUse` hooks to add programmatic validation:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "your-validation-script.sh"
          }
        ]
      }
    ]
  }
}
```

Hooks can approve, deny, or modify tool calls at runtime — a good complement to these declarative rules.

### Additional Protection: Sandboxing

Claude Code also supports [sandboxing](https://code.claude.com/docs/en/sandboxing) which provides filesystem and network isolation for bash commands. Enable it with `/sandbox` during a session. Sandboxing and security rules work well together — sandboxing restricts what's *possible*, while security rules restrict what's *attempted*.

---

## Customization

The rules are intentionally comprehensive. You might want to adjust them for your use case:

**Relax rules** for trusted environments:
- Remove the macOS-specific section if you're on Linux
- Remove cloud/infrastructure rules if you don't use AWS/GCP/Azure
- Remove database rules if your project has no database

**Tighten rules** for sensitive environments:
- Add project-specific file paths to the protected list
- Add specific commands to the deny list
- Add your own domain-specific rules

**Example: Adding a custom rule**

```markdown
## 13. Project-Specific Rules

- NEVER modify files in `./config/production/` without confirmation
- NEVER run `make deploy` without confirmation
- NEVER access the `payments` database table without asking
```

---

## How It Works

Claude Code reads `CLAUDE.md` at session start. The mandatory reference in `CLAUDE.md` points to `security.md`, which Claude Code treats as strict operational constraints. These rules are processed as natural language instructions that override default behavior.

This approach works because:

1. **Claude Code's architecture** respects `CLAUDE.md` as the highest-priority user instruction
2. **The rules are specific and unambiguous** — no room for interpretation
3. **The rules include anti-injection protections** (Section 10) — they resist being overridden by malicious content in project files
4. **Managed settings** (`managed-settings.json`) in system directories can enforce these rules organization-wide and cannot be overridden by users

---

## Contributing

Found a gap? Thought of a scenario not covered? Contributions are welcome.

1. Fork the repository
2. Add your rules or improvements
3. Submit a pull request with a clear description of what scenario you're addressing

Please keep rules:
- **Specific** — describe exact commands or patterns
- **Actionable** — clear what to do (or not do)
- **Universal** — applicable across projects and environments

---

## License

MIT License — free to use, modify, and distribute. See [LICENSE](LICENSE) for details.
