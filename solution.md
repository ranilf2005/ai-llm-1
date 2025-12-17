# AI-Driven Network Automation Lab

## Overview
This document explains an AI-driven network automation lab using VS Code, GitHub Copilot, MCP, Cisco CML, and pyATS to simplify complex network operations.

## Use Case
Engineers use natural language through Copilot Chat to retrieve data or configure routers without directly writing automation scripts.

## Technologies Used
- VS Code: Unified development environment
- GitHub Copilot: AI assistant
- Cisco CML: Virtual lab platform
- Cisco pyATS: Network testing framework
- MCP: AI-to-tool communication layer

## Benefits to Engineers
- Faster troubleshooting
- Reduced scripting knowledge required
- Safe automation
- Improved confidence

## Benefits to Organization
- Consistent validation
- Reduced outages
- Faster onboarding
- Scalable automation

## Conclusion
This lab demonstrates how AI can safely and effectively assist engineers in real network operations.



# AI-Driven Network Automation Lab (VS Code + Copilot + MCP + CML + pyATS)
**Goal:** Use **GitHub Copilot Chat** in **VS Code** to run **pyATS/Unicon** actions against **Cisco CML** routers via an **MCP server**, using natural language prompts.

Your lab (as provided):
- Ubuntu 25.04 workstation: `192.168.30.50`
- Cisco CML router: `iosv-0` at `192.168.30.181` (SSH enabled)
- You can SSH from Ubuntu → router successfully

---

## 1) Folder layout (create once)
We’ll keep everything in a single workspace folder so VS Code, pyATS, and MCP all align.

```bash
mkdir -p ~/lab/pyats-mcp/{testbed,mcp_server,.vscode,logs,scripts}
cd ~/lab/pyats-mcp
```

Expected structure:
```
~/lab/pyats-mcp/
  .vscode/
    mcp.json
  mcp_server/
    server.py
  testbed/
    testbed.yaml
  scripts/
    test_show.py
    test_config.py
  logs/
```

---

## 2) Ubuntu prerequisites
### 2.1 Install base packages
```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip git openssh-client
```

### 2.2 Confirm basic connectivity to the router
```bash
ping -c 3 192.168.30.181
ssh <router_user>@192.168.30.181
```

> If SSH works, move on.

---

## 3) Install & configure pyATS (recommended approach)
Cisco recommends installing pyATS via `pip` (and the `library` extra includes Genie libraries).

### 3.1 Create the virtual environment
You said you already have `pyats-env`, but this is the clean “gold” method:
```bash
cd ~/lab/pyats-mcp
python3 -m venv pyats-env
source pyats-env/bin/activate
python -m pip install --upgrade pip setuptools wheel
```

### 3.2 Install pyATS (Genie libraries included)
```bash
pip install "pyats[library]"
```

### 3.3 Validate installation
```bash
pyats version
python -c "import pyats; print('pyATS OK')"
```

---

## 4) Create your pyATS testbed file
Create: `~/lab/pyats-mcp/testbed/testbed.yaml`

> Replace `<router_user>` and `<router_password>` with your lab credentials.
> If you will use SSH keys, you can still keep password here for now, but keys are preferred.

```yaml
testbed:
  name: cml-lab
  credentials:
    default:
      username: <router_user>
      password: <router_password>

devices:
  iosv-0:
    os: ios
    type: router
    connections:
      cli:
        protocol: ssh
        ip: 192.168.30.181
        port: 22
        arguments:
          connection_timeout: 60
```

---

## 5) Quick pyATS test (without MCP)
This confirms pyATS can log in and execute commands.

Create: `~/lab/pyats-mcp/scripts/test_show.py`
```python
from pyats.topology import loader

tb = loader.load("testbed/testbed.yaml")
dev = tb.devices["iosv-0"]

dev.connect(log_stdout=False)
print(dev.execute("show ip interface brief"))
dev.disconnect()
```

Run:
```bash
cd ~/lab/pyats-mcp
source pyats-env/bin/activate
python scripts/test_show.py
```

✅ If you see command output, pyATS connectivity is working.

---

## 6) (Recommended) Set up SSH keys for smoother automation
This avoids saving passwords in scripts.

### 6.1 Generate a key (if you don’t have one)
```bash
ssh-keygen -t ed25519 -C "pyats-mcp"   # press Enter for defaults
```

### 6.2 Copy the public key to the router (if supported)
```bash
ssh-copy-id <router_user>@192.168.30.181
```

### 6.3 Test passwordless SSH
```bash
ssh <router_user>@192.168.30.181 "show version | include IOS"
```

> If your IOSv image doesn’t support `ssh-copy-id` style workflows easily, you can keep password auth for lab use.

---

## 7) Install MCP Python SDK (in the SAME venv)
```bash
cd ~/lab/pyats-mcp
source pyats-env/bin/activate
pip install mcp
```

---

## 8) Create the MCP Server (Python)
This is the “bridge” that lets Copilot call real tools (pyATS functions).

Create: `~/lab/pyats-mcp/mcp_server/server.py`
```python
import os
from typing import Literal

from mcp.server.fastmcp import FastMCP
from pyats.topology import loader

mcp = FastMCP("cml-pyats-tools")

TESTBED_PATH = os.environ.get("PYATS_TESTBED", "testbed/testbed.yaml")

def _connect(device_name: str):
    tb = loader.load(TESTBED_PATH)
    if device_name not in tb.devices:
        raise ValueError(f"Unknown device '{device_name}'. Check {TESTBED_PATH}")
    dev = tb.devices[device_name]

    # Lab-safe SSH defaults to reduce prompts that break automation
    dev.connections["cli"].setdefault(
        "ssh_options",
        "-o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null"
    )

    dev.connect(log_stdout=False)
    return dev

@mcp.tool()
def show_command(device: str, command: str) -> str:
    """Run a SHOW command and return output."""
    dev = _connect(device)
    try:
        if not command.strip().lower().startswith("show"):
            raise ValueError("Only 'show ...' commands allowed in show_command().")
        return dev.execute(command)
    finally:
        dev.disconnect()

@mcp.tool()
def apply_config(device: str, config: str, save: bool = False) -> str:
    """Apply configuration lines to a device. Set save=True to 'write memory'."""
    dev = _connect(device)
    try:
        out = dev.configure(config)
        if save:
            out += "\n\n--- write memory ---\n"
            out += dev.execute("write memory")
        return out
    finally:
        dev.disconnect()

@mcp.tool()
def health_check(device: str, level: Literal["basic", "detailed"] = "basic") -> str:
    """Simple health checks (extend as you grow)."""
    dev = _connect(device)
    try:
        cmds = ["show version", "show ip interface brief"]
        if level == "detailed":
            cmds += [
                "show processes cpu | include one minute",
                "show memory statistics",
                "show ip route summary",
            ]
        results = []
        for c in cmds:
            results.append(f"\n===== {c} =====\n{dev.execute(c)}")
        return "\n".join(results)
    finally:
        dev.disconnect()

if __name__ == "__main__":
    # IMPORTANT: MCP stdio servers should not print() to stdout.
    # FastMCP uses stdout for the protocol.
    mcp.run()
```

---

## 9) Configure VS Code to run the MCP server
VS Code reads MCP server definitions from `.vscode/mcp.json` in your project folder.

Create: `~/lab/pyats-mcp/.vscode/mcp.json`

> Replace `/home/<your_linux_user>/...` with your real Linux username and path.

```json
{
  "mcpServers": {
    "cml-pyats-tools": {
      "command": "/home/<your_linux_user>/lab/pyats-mcp/pyats-env/bin/python",
      "args": [
        "/home/<your_linux_user>/lab/pyats-mcp/mcp_server/server.py"
      ],
      "env": {
        "PYATS_TESTBED": "/home/<your_linux_user>/lab/pyats-mcp/testbed/testbed.yaml"
      }
    }
  }
}
```

### 9.1 Open the workspace in VS Code
```bash
code ~/lab/pyats-mcp
```

### 9.2 Ensure Copilot extensions are installed
In VS Code extensions:
- **GitHub Copilot**
- **GitHub Copilot Chat**

Sign in to GitHub if prompted.

### 9.3 Trust prompt
VS Code may ask you to trust the MCP server (because it runs code locally). Only trust your own server.

---

## 10) Use Copilot Chat with your MCP tools
Open **Copilot Chat** and try prompts like:

### 10.1 Run a show command
> “Use the tool `show_command` on device `iosv-0` to run `show ip interface brief`.”

### 10.2 Run health checks
> “Run `health_check` on `iosv-0` with level `basic`.”

### 10.3 Apply a safe config (lab)
> “Use `apply_config` on `iosv-0` to add a loopback interface and IP, then save.”

Example config to paste:
```text
interface loopback123
 ip address 10.123.123.1 255.255.255.255
 description created-by-copilot-mcp
```

---

## 11) End-to-end testing steps (your checklist)
### Test A — pyATS connectivity
```bash
cd ~/lab/pyats-mcp
source pyats-env/bin/activate
python scripts/test_show.py
```

### Test B — MCP server loads (VS Code)
- Open Copilot Chat
- Ask a tool-based prompt (show_command)

### Test C — show command output returns
- Confirm output appears in chat

### Test D — config change and verify
- Apply config with `apply_config`
- Then run `show ip interface brief` to confirm loopback is up

---

## 12) Troubleshooting
### 12.1 “MCP server not connecting / tools not visible”
- Confirm `.vscode/mcp.json` is valid JSON
- Ensure **absolute paths**
- Ensure the venv python path is correct:
  ```bash
  ls -l ~/lab/pyats-mcp/pyats-env/bin/python
  ```

### 12.2 “Authentication failed”
- Confirm SSH works from terminal:
  ```bash
  ssh <router_user>@192.168.30.181
  ```
- Re-check credentials in `testbed.yaml`

### 12.3 “Server crashes / disconnects”
- Remove any `print()` in `server.py`
- Avoid writing to stdout because MCP uses stdout for the protocol

---

## 13) Safety guardrails (recommended for team demo)
- Keep `show_command` restricted to `show` commands
- Only allow config changes via `apply_config` with clear intent
- Add an “approval” step for risky actions (future enhancement)

---

## 14) Next improvements (after the demo)
- Add more devices to `testbed.yaml`
- Add tool: `check_bgp_neighbors()` / `verify_routes()` / `backup_running_config()`
- Add reports/logging to `logs/`
- Add role-based controls (who can run config vs show)

---

## Appendix: One-liner to verify everything quickly
```bash
cd ~/lab/pyats-mcp && source pyats-env/bin/activate && python scripts/test_show.py
```

---

### Done ✅
You now have:
- A working pyATS environment
- A testbed for your CML router
- An MCP server exposing network tools
- VS Code configured to let Copilot Chat call those tools using natural language
