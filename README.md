# Flowlines coding assistant observability

Send Claude Code and Codex CLI sessions to Flowlines with full prompts, assistant responses,
tool activity, and token usage.

The integration supports interactive and non-interactive sessions, including `claude -p` and
`codex exec`. It installs user-level configuration on macOS or Linux and does not modify either
CLI's application code.

## Privacy notice

This integration exports full prompts, assistant messages, tool inputs, and tool outputs from
your machine to Flowlines. That content can contain source code, file contents, commands, and
other sensitive information.

Only continue after you understand the data being exported and have permission to send it to
Flowlines. The installer requires explicit consent before changing any configuration.

## Requirements

- macOS or Linux
- Python 3
- `curl` when installing Codex telemetry
- Claude Code, Codex CLI, or both
- A Flowlines API key

## Install

Clone this repository:

```sh
git clone https://github.com/flowlines-ai/coding-assistant-observability.git
cd coding-assistant-observability
```

Run the installer:

```sh
./skills/flowlines-agent-observability/scripts/install.sh --target auto
```

The installer asks for full-content consent and reads the Flowlines API key through a masked
terminal prompt. `auto` detects which supported CLIs are installed. You can select one explicitly:

```sh
./skills/flowlines-agent-observability/scripts/install.sh --target claude
./skills/flowlines-agent-observability/scripts/install.sh --target codex
./skills/flowlines-agent-observability/scripts/install.sh --target both
```

The default destination is the production Flowlines API at `https://api.flowlines.ai`. To use a
different Flowlines deployment, pass its HTTPS base URL:

```sh
./skills/flowlines-agent-observability/scripts/install.sh \
  --target claude \
  --api-base https://api.example.com
```

### Non-interactive installation

For an approved automated installation, place the API key in a private file and pass explicit
consent:

```sh
chmod 600 /secure/path/flowlines-api-key
FLOWLINES_FULL_CONTENT_CONSENT=yes \
FLOWLINES_API_KEY_FILE=/secure/path/flowlines-api-key \
./skills/flowlines-agent-observability/scripts/install.sh \
  --target claude \
  --non-interactive
```

Avoid putting the API key directly in a shell command because it may remain in shell history or
process metadata.

## Verify

Check the local configuration:

```sh
./skills/flowlines-agent-observability/scripts/doctor.sh
```

Then run one harmless prompt with each configured CLI and confirm that the session appears in
Flowlines. A successful doctor check validates local configuration; receiving a session validates
the complete network and ingestion path.

After installing Codex hooks, open `/hooks` once in Codex and trust the installed user hook
definition. Until it is trusted, Codex may omit full prompt, tool, or assistant-response content.

## What is configured

### Claude Code

The installer enables Claude Code's OTLP log exporter and enhanced trace exporter. Claude sends
the relevant signals to the standard Flowlines OTLP endpoints:

- `/v1/logs`
- `/v1/traces`

The settings apply to interactive Claude Code and `claude -p`.

### Codex CLI

The installer configures Codex's OTLP log exporter for `/v1/logs` and adds fail-open user hooks for:

- `UserPromptSubmit`
- `PostToolUse`
- `Stop`

The hooks send full event content to `/v1/agent-events/codex`. A bounded local spool retries
temporary failures, while short network timeouts ensure telemetry cannot block the agent. The
settings apply to interactive Codex and `codex exec`.

## Local files and secrets

The installer:

- merges existing Claude JSON, Codex TOML, and Codex hook configuration;
- backs up the original files before its first change;
- writes secret-bearing files with mode `0600`;
- stores Codex retry events under the user's Flowlines observability state directory;
- never prints the Flowlines API key.

The API key is used only as an ingestion credential. It must never be included in exported prompt,
hook, or diagnostic content.

## Repair and uninstall

Rerun the installer to repair an installation. It is idempotent and preserves the first-install
backup.

To restore the original configuration:

```sh
./skills/flowlines-agent-observability/scripts/uninstall.sh
```

Uninstall removes the local Codex spool, including any events that have not yet been delivered. If
a managed configuration file changed after installation, uninstall refuses to overwrite it and
reports the preserved backup path.

## Agent skill

The reusable agent skill is located at
`skills/flowlines-agent-observability`. Compatible coding assistants can use its `SKILL.md` to
perform consent-aware installation, diagnosis, repair, and uninstall workflows. The scripts can
also be run directly without installing the skill into an assistant.

## Development

Run the isolated installer test suite on macOS or Linux:

```sh
./skills/flowlines-agent-observability/scripts/test_installer.sh
```

The tests use a temporary home directory and do not contact Flowlines.

## License

MIT
