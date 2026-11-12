# agent-secrets-macos

A Claude skill that lets AI agents **use** your secrets without ever **seeing** them, on macOS.

Three files, no dependencies, only tools built into macOS. You can read all of it in five minutes, and you should: never install a skill you haven't read.

## Why

AI agents run on your machine, with your permissions, next to your API keys.
A hidden instruction in a README, a web page, an MCP server or a skill (prompt injection) can make an agent read a `.env` file and leak its contents.

This skill changes three things:

- Secrets live in the macOS Keychain, not in `.env` files.
- The agent handles secret *names*, never their values. You type the values yourself, in your own Terminal.
- Each Keychain read asks for your confirmation, so a human stays in the loop.

## What's inside

| File | Role |
|------|------|
| `SKILL.md` | The rules the agent follows |
| `scripts/secret` | Store, list, delete, audit and import secrets |
| `scripts/with-secrets` | Inject secrets into one command only |

## Usage

```bash
chmod +x scripts/*

# Store a secret (hidden input, never passed as an argument)
scripts/secret store openai

# Run a command with the secret injected into that command only
scripts/with-secrets OPENAI_API_KEY=openai -- python app.py

# Migrate a .env file
scripts/secret audit .env    # key names only, no values
scripts/secret import .env   # then delete .env and rotate exposed keys
```

When macOS asks for confirmation, click **Allow**, not **Always Allow**.

## Install

**Claude Code:** copy the whole `agent-secrets-macos` folder (SKILL.md and scripts/) into `~/.claude/skills/`.

**Claude app:** download `agent-secrets-macos.skill` from the latest release, then upload it in Claude's skills settings.

## Limits

- A skill is a set of instructions, not a security barrier. A manipulated agent can ignore it. The Keychain confirmation dialog is the real safeguard.
- Once injected, a secret is readable by the process that receives it.
- Static API keys (OpenAI, Stripe…) still have to be sent to the service. Only hardware-bound keys (SSH with FIDO2 or Secure Enclave) truly never leave the device.
- macOS only.

## License

MIT
