# Contributing

Thanks for your interest in improving this project. Contributions of all sizes
are welcome, from typo fixes to new pyATS-backed MCP tools.

## Code of conduct

This project follows the [Contributor Covenant](CODE_OF_CONDUCT.md). By
participating you agree to uphold it.

## Ways to contribute

- **Report a bug** – open an issue using the bug report template. Include your OS,
  Python version, pyATS version, the device platform and the exact error text.
- **Request a feature** – open an issue using the feature request template and
  describe the network operation you want to drive from natural language.
- **Submit a pull request** – see the workflow below.

## Development setup

```bash
git clone https://github.com/ranilf2005/cisco-pyats-mcp-network-automation.git
cd cisco-pyats-mcp-network-automation

python3 -m venv .venv
source .venv/bin/activate          # Windows: .venv\Scripts\Activate.ps1
python -m pip install --upgrade pip
pip install -r requirements-dev.txt
```

## Before you open a pull request

```bash
ruff check .          # lint
ruff format --check . # formatting
pytest                # unit tests (no device required)
bandit -r mcp_server scripts
```

The guardrail tests in [tests/test_validation.py](tests/test_validation.py) run
without pyATS or a live device, so they must always pass.

## Pull request guidelines

1. Create a branch from `main`: `git checkout -b feature/my-change`.
2. Keep the change focused; one logical change per pull request.
3. Add or update tests for any behaviour you change.
4. Update the README and [CHANGELOG.md](CHANGELOG.md) when behaviour changes.
5. Never commit credentials, testbed files, IP addresses of production devices,
   `.env` files or `.vscode/mcp.json`.
6. Use clear commit messages, for example `feat: add BGP neighbour check tool`.

## Adding a new MCP tool

New tools live in [mcp_server/server.py](mcp_server/server.py). Every tool must:

- be decorated with `@mcp.tool()` and carry a docstring, because the docstring is
  what the language model sees;
- validate all caller-supplied strings through
  [mcp_server/validation.py](mcp_server/validation.py);
- use the `connected_device` context manager so sessions are always closed;
- default to read-only. Anything that changes device state must be gated behind
  an environment variable, as `apply_config` is;
- never write to stdout, which the MCP stdio transport reserves for protocol
  frames. Log to stderr instead.

## Security issues

Please do not open a public issue for a security vulnerability. Follow
[SECURITY.md](SECURITY.md) instead.

## Licensing of contributions

By contributing you agree that your contribution is licensed under the
GNU General Public License v3.0 or later, the same licence as this project.
