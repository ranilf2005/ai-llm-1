# Security Policy

## Scope and intended use

This project connects a large language model to real network devices. It is
intended for **lab and demonstration environments** such as Cisco Modeling Labs.
It is sample code: it is not a Cisco product, is not supported by Cisco TAC, and
is not hardened for production networks.

## Reporting a vulnerability

Please **do not** open a public GitHub issue for a security vulnerability.

Instead, use GitHub's private vulnerability reporting on the repository
(**Security → Report a vulnerability**). Include:

- a description of the issue and its impact,
- the affected version or commit,
- reproduction steps or a proof of concept.

You can expect an acknowledgement within 5 business days and a status update
within 15 business days.

## Built-in safeguards

| Safeguard | Behaviour |
|---|---|
| Read-only by default | `apply_config` refuses to run unless `PYATS_MCP_ALLOW_CONFIG=true`. |
| Separate save gate | Writing to startup-config also requires `PYATS_MCP_ALLOW_SAVE=true`. |
| Show-command allow-list | Only a single `show` command is accepted; pipe modifiers are limited to `begin`, `count`, `exclude`, `include`, `section`, so output cannot be written to device storage. |
| Injection guards | `;`, `&`, backticks, `$`, `<`, `>` and newlines are rejected in commands. |
| Destructive-config deny-list | `reload`, `write erase`, `erase`, `format`, `delete`, `boot system`, `config-register`, `crypto key zeroize`, local `username` changes, `enable secret` changes, `no aaa`, `no ip ssh` and `no line vty` are always rejected. |
| Device allow-list | `PYATS_MCP_ALLOWED_DEVICES` restricts which testbed devices are reachable. |
| SSH host key verification | Enabled by default; disabling it requires `PYATS_MCP_INSECURE_SSH=true`. |
| No credentials in the repo | The testbed resolves credentials from environment variables via `%ENV{}`; `.env`, `testbed/testbed.yaml` and `.vscode/mcp.json` are git-ignored. |
| Bounded output | Responses are truncated at `PYATS_MCP_MAX_OUTPUT_CHARS`. |

## Operator responsibilities

- Run the server against lab devices you own, on an isolated management network.
- Use a device account with the **least privilege** needed. Read-only usage does
  not require privilege 15.
- Keep `PYATS_MCP_ALLOW_CONFIG=false` unless you are actively demonstrating
  configuration changes, and set `PYATS_MCP_ALLOWED_DEVICES` when you do.
- Leave SSH host key verification enabled outside throwaway labs.
- Review every tool call your AI assistant proposes before approving it. Treat
  model-generated commands as untrusted input.
- Rotate any credential that has ever been pasted into a chat session.

## Known limitations

- The deny-list is a safety net, not a complete authorisation model. Enforce real
  authorisation on the device with AAA and command authorisation.
- Device output returned to the model may contain sensitive data such as
  interface addressing and routing information. Do not point the server at
  devices whose configuration must not leave your environment.

## Supported versions

Security fixes are applied to the latest release on the `main` branch.
