# Understanding the MCP + pyATS Script (Beginner-Friendly Guide)

This document explains the Python script **step-by-step** in very simple terms, so even a **non-technical person** can understand what it does and how to learn from it to build their own scripts.

---

## 1. What This Script Does (Big Picture)

This script creates a **tool server** that can:

- Connect to Cisco devices using **pyATS**
- Run safe **SHOW commands**
- Apply **configuration**
- Perform **basic or detailed health checks**

It is designed to work with:
- Cisco Modeling Labs (CML)
- pyATS testbed files
- MCP (Model Context Protocol) tools

---

## 2. Required Imports

```python
import os
from typing import Literal
from mcp.server.fastmcp import FastMCP
from pyats.topology import loader
```

---

## 3. Creating the MCP Tool Server

```python
mcp = FastMCP("cml-pyats-tools")
```

Creates a tool server that exposes Python functions as callable tools.

---

## 4. Testbed File Handling

```python
TESTBED_PATH = os.environ.get("PYATS_TESTBED", "testbed/testbed.yaml")
```

Uses an environment variable if available, otherwise a default path.

---

## 5. Device Connection Helper

The `_connect()` function:
- Loads the testbed
- Validates the device name
- Applies SSH-safe defaults
- Connects and returns the device object

---

## 6. MCP Tools

### show_command
Runs safe, read-only `show` commands.

### apply_config
Applies configuration and optionally saves it.

### health_check
Runs basic or detailed health checks using predefined commands.

---

## 7. Best Practices Demonstrated

- Input validation
- Reusable helper functions
- Safe cleanup with `try/finally`
- Scalable and readable design

---

## 8. Final Takeaway

If you understand this script, you are already thinking like a **network automation engineer**.

---
