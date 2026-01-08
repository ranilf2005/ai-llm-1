# MCP server config explained (beginner-friendly)

This file is **NOT a Python script**.  
It’s a **JSON configuration** (often stored as something like `mcp.json`, `settings.json`, or similar) that tells a tool (for example, an MCP client inside VS Code) **how to start your MCP server** and **which environment variables to pass to it**.

---

## The config you provided

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

Think of it like a **remote control**:
- **command** = what program to run
- **args** = what to run with it (the “instructions” / parameters)
- **env** = extra settings given to that program (environment variables)

---

## Step-by-step (every line explained)

### 1) The outer braces `{ ... }`
This is the JSON “container” (like a box) holding all settings.

---

### 2) `"mcpServers": { ... }`
This is a section named **mcpServers**.

Inside it you can define **one or more MCP servers**, each with its own name and settings.

---

### 3) `"cml-pyats-tools": { ... }`
This is the **server name/label**.

- You can choose any name you like.
- It’s used so your MCP client can say: “Start the server called `cml-pyats-tools`”.

---

### 4) `"command": "/home/<your_linux_user>/lab/pyats-mcp/pyats-env/bin/python"`
This is the **exact Python interpreter** to use.

Why this matters:
- In Linux you may have multiple Python installs.
- This path points to a **Python inside a virtual environment** (`pyats-env`).
- That venv likely has `pyATS` and other dependencies installed.
- Using the venv Python ensures your server runs with the right packages.

**What you must do:**
Replace `<your_linux_user>` with your actual Linux username.

Example:
```text
/home/ranil/lab/pyats-mcp/pyats-env/bin/python
```

---

### 5) `"args": [ "/home/<your_linux_user>/lab/pyats-mcp/mcp_server/server.py" ]`
**args** means “arguments”.

Since the command is `python`, the first argument is the Python file to run:

- `server.py` is your MCP server program.
- This config is effectively running:
  ```bash
  /home/<user>/.../bin/python /home/<user>/.../server.py
  ```

You can add more arguments later if your `server.py` supports them, for example:
```json
"args": [
  "/home/ranil/lab/pyats-mcp/mcp_server/server.py",
  "--debug"
]
```
(Only do this if your server code actually supports `--debug`.)

---

### 6) `"env": { "PYATS_TESTBED": "..." }`
This section defines **environment variables**.

Environment variables are simple **key/value settings** that programs can read at runtime.

Here you set:
- **key**: `PYATS_TESTBED`
- **value**: `/home/<your_linux_user>/lab/pyats-mcp/testbed/testbed.yaml`

What it means:
- pyATS uses a **testbed file** (YAML) that describes your lab devices:
  - IPs / hostnames
  - credentials
  - device types (IOS-XE, NX-OS, etc.)
  - connection methods (ssh, telnet, etc.)

Many pyATS-based tools will look for the testbed location in:
- an environment variable like `PYATS_TESTBED`, or
- a command-line argument, or
- a default path

So this config ensures your MCP server always knows where the testbed file is.

---

## What happens when this MCP server starts?

When your MCP client starts the server, it effectively does:

1. Use this Python interpreter:
   ```bash
   /home/<user>/lab/pyats-mcp/pyats-env/bin/python
   ```
2. Run this file:
   ```bash
   /home/<user>/lab/pyats-mcp/mcp_server/server.py
   ```
3. Set an environment variable for the process:
   ```bash
   PYATS_TESTBED=/home/<user>/lab/pyats-mcp/testbed/testbed.yaml
   ```

So the server can read that environment variable and load the testbed.

---

## How to use this to learn scripting (the “patterns” to copy)

Even though this file is JSON, it teaches great “automation patterns” you’ll reuse in Python scripting:

### Pattern 1: Pin the interpreter (reproducible runs)
Using a venv Python path ensures:
- your code runs the same everywhere
- dependencies are predictable

In your own scripts, you’ll do similar things by:
- using venvs
- using `requirements.txt` / `pyproject.toml`

---

### Pattern 2: Keep configuration outside code
Instead of hardcoding paths in Python, pass them in via:
- environment variables (like `PYATS_TESTBED`)
- config files
- command-line args

This makes scripts reusable and easier to share.

---

### Pattern 3: Separate “runner” vs “logic”
- This JSON config is the **runner** (how to start things).
- `server.py` is the **logic** (what actually happens).

When you write scripts, aim for:
- a small “entry point” (CLI / runner)
- reusable functions/modules for the logic

---

## Common mistakes and easy checks

### 1) Wrong username or path
If the path is wrong, the MCP client cannot start the server.

Quick check in terminal:
```bash
ls -l /home/<your_linux_user>/lab/pyats-mcp/pyats-env/bin/python
ls -l /home/<your_linux_user>/lab/pyats-mcp/mcp_server/server.py
ls -l /home/<your_linux_user>/lab/pyats-mcp/testbed/testbed.yaml
```

---

### 2) Virtual environment missing packages
If the server starts but errors like “module not found”, you may need to install dependencies inside the venv:
```bash
source /home/<your_linux_user>/lab/pyats-mcp/pyats-env/bin/activate
pip install -r requirements.txt
```

---

### 3) YAML testbed problems
If the file exists but the server complains about device connections:
- credentials might be wrong
- IPs might be unreachable
- device OS/type might be mismatched

---

## A clean template you can reuse

Copy and replace the placeholders:

```json
{
  "mcpServers": {
    "my-server-name": {
      "command": "/home/YOURUSER/path/to/venv/bin/python",
      "args": [
        "/home/YOURUSER/path/to/server.py"
      ],
      "env": {
        "PYATS_TESTBED": "/home/YOURUSER/path/to/testbed.yaml"
      }
    }
  }
}
```

---

## Mini cheat-sheet (JSON basics)

- `{ ... }` = an object (a set of named fields)
- `"key": "value"` = a field
- `[ ... ]` = a list/array
- JSON needs:
  - double quotes `"..."` (not single quotes)
  - commas between items
  - no trailing comma at the end

---

## Next steps for learning Python from here

If you want to “connect this config to Python thinking”, start with these beginner goals:

1. Learn how Python reads environment variables:
   ```python
   import os
   testbed = os.getenv("PYATS_TESTBED")
   print(testbed)
   ```

2. Learn how Python reads command-line arguments:
   ```python
   import sys
   print(sys.argv)
   ```

3. Learn how Python loads YAML (commonly used with pyATS):
   ```python
   import yaml
   with open("testbed.yaml") as f:
       data = yaml.safe_load(f)
   print(data)
   ```

(Those are examples only — you’ll need the `pyyaml` package for YAML.)

---

## Safety tip (important)
Your `testbed.yaml` often contains **device usernames/passwords**.

When uploading to GitHub:
- **Never** commit real credentials.
- Use placeholders or a separate secret file.
- Consider `.gitignore` for the real testbed file, and commit a `testbed.sample.yaml` instead.

---

**That’s it.**  
This config is basically: “Use *this Python*, run *that server*, and point it at *this testbed*.”
