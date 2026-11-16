# Chapter 2: Create a Python Virtual Environment

> **Navigation:** [← Previous: Chapter 1: Create a Google API Key](./01-create-api-key.md) | [Table of Contents](../README.md) | [Next: Chapter 3: Project Setup & ADK Dependencies →](./03-project-setup-and-dependencies.md)

## Overview

Isolating project dependencies in a Python virtual environment (`venv`) prevents package version conflicts across projects.

- **Prerequisites:** Python **3.10+** installed on your system.

---

## Step-by-Step Guide

### 1. Create the Virtual Environment

Open your terminal (Command Prompt, PowerShell, or Bash) in your project root directory and run:

```bash
python -m venv .venv
```

*(Note: Either `venv` or `.venv` works—just use the same folder name consistently across your commands and `.gitignore`.)*

### 2. Activate the Virtual Environment

Choose the command for your operating system and shell:

| OS / Shell | Activation Command |
| :--- | :--- |
| **Windows (Command Prompt)** | `.venv\Scripts\activate` |
| **Windows (PowerShell)** | `.\.venv\Scripts\Activate.ps1` |
| **Linux / macOS (Bash/Zsh)** | `source .venv/bin/activate` |

Once activated, your terminal prompt will display the environment prefix:

```text
(.venv) C:\Projects\PersonalFinanceAdvisor>
```

### 3. Verify Base Installation

Check installed packages inside your active environment:

```bash
pip list
```

---

> **Navigation:** [← Previous: Chapter 1: Create a Google API Key](./01-create-api-key.md) | [Table of Contents](../README.md) | [Next: Chapter 3: Project Setup & ADK Dependencies →](./03-project-setup-and-dependencies.md)
