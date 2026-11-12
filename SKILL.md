---
name: agent-secrets-macos
description: Keep API keys, tokens and passwords out of the agent's reach on macOS. Use this skill whenever a task involves an API key, token, password, credential, .env file, secret, SSH key or service authentication on a Mac, including when the user pastes a key into the chat, asks to configure an SDK or MCP server, or a project contains a .env file, even if the user does not mention security.
---

# Agent secrets on macOS

Goal: the agent can **use** secrets without ever **seeing** them.

Secrets live in the macOS login Keychain, under the service prefix `agent-secrets/`.
The agent only handles secret *names* (e.g. `openai`), never their values.
Everything here uses tools built into macOS. No package to install.

## Rules

1. **Never display a secret.** Do not run `security find-generic-password -w`, `cat .env`, `printenv`, `env` or anything else that would put a secret value into the conversation or a log.
2. **Never write a secret** into code, config files, `.env`, commit messages, shell history or command-line arguments.
3. **The user enters secrets, not the agent.** To store one, give the user the command to run in *their own* Terminal (see below). Do not ask them to paste the value into the chat.
4. **If a secret is pasted in the chat**, do not reuse it. Tell the user it is now exposed, recommend revoking and regenerating it, then storing the new one with `secret store`.
5. **Use secrets only through `with-secrets`**, which injects them into the one command that needs them.
6. **Do not suggest "Always Allow"** on Keychain dialogs. Clicking "Allow" each time is what keeps a human in the loop.

## Commands

Scripts are in `scripts/`. Make them executable once: `chmod +x scripts/*`.

### Store a secret (user runs this in their own Terminal)

```bash
scripts/secret store openai
```

The value is typed with hidden input and passed to Keychain through stdin, never as an argument.
Items are created with an empty trusted-app list (`-T ""`), so on a standard login keychain macOS asks for confirmation whenever a program reads them.

### Use a secret

```bash
scripts/with-secrets OPENAI_API_KEY=openai -- python app.py
scripts/with-secrets GITHUB_TOKEN=github STRIPE_KEY=stripe -- npm run deploy
```

The values exist only in the environment of that child process. The agent sees the command's output, not the secret.
Make sure the command itself does not print the secret.

### Other commands

```bash
scripts/secret list            # names only, never values
scripts/secret exists openai   # exit code 0 if present
scripts/secret delete openai
```

## Migrating a .env file

1. Show which keys exist, without values: `scripts/secret audit .env`
2. The user imports them in their own Terminal: `scripts/secret import .env`
3. Replace the code's reliance on `.env` with `with-secrets` in scripts, Makefile or package.json.
4. Delete the `.env` file and make sure `.env*` is in `.gitignore`.
5. If the file was ever committed or shared, tell the user to **rotate** those keys: removing the file does not remove it from git history.

## SSH and Git: keys that cannot leave the hardware

For SSH, prefer keys that are generated inside hardware and can never be exported:

- With a FIDO2 security key: `ssh-keygen -t ed25519-sk` (or `-t ecdsa-sk`). GitHub and GitLab accept these.
- Without one: keys stored in the Mac's Secure Enclave via a dedicated app such as Secretive (open source), which acts as an SSH agent.

Never generate an unprotected `id_ed25519` without a passphrase for the agent to use.

## Limits (tell the user when relevant)

- This skill is a set of instructions, not a security barrier. An agent manipulated by a prompt injection can ignore it. The Keychain confirmation dialog is the real safeguard, so read it before clicking "Allow".
- Once a secret is injected into a process, that process (and any code it runs) can read it.
- Services using static API keys (OpenAI, Stripe…) still need the key to be sent. Only hardware-bound keys (SSH, signatures) truly never leave.
- macOS only.
