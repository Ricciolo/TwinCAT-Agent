![Version](https://img.shields.io/badge/version-1.0.25-blue)

TwinCAT coding agent for Beckhoff PLC development, with strict rules for Structured Text, naming conventions, PLC-aware tooling, syntax checks, and safe project validation workflows.

## In Brief

TwinCAT-Agent provides:

- a dedicated `twincat` agent
- TwinCAT-specific skills and operating rules
- a local MCP server for TwinCAT XAE engineering automation
- a remote MCP server for Beckhoff InfoSys search and symbol lookup

In practice, it can automate TwinCAT XAE engineering actions on Windows (via a local MCP server), search Beckhoff InfoSys documentation and symbols (via a remote MCP server), and enforce a safer TwinCAT workflow than a generic assistant (naming conventions, PLC-aware operations, syntax checks, and a safe activation/login sequence).

**Jump to**: [✅ Requirements](#requirements) | [🧩 Install with Claude Code](#install-with-claude-code) | [🧩 Install with GitHub Copilot CLI](#install-with-github-copilot-cli) | [🚀 Quickstart](#quickstart)

## 👷 Who This Is For

This repository is intended for developers and automation engineers working with:

- Beckhoff TwinCAT 3
- Structured Text and ExST
- PLC project maintenance and refactoring
- PLC-aware code generation, review, and troubleshooting

## ✅ Requirements

- Windows x64
- TwinCAT 3.1 Build 4026 or later
- TwinCAT XAE Shell 64-bit
- GitHub Copilot CLI or Claude Code

## 🧩 Install with Claude Code

Start Claude Code:

```bash
claude
```

Add this GitHub repo as a marketplace:

```text
/plugin marketplace add ricciolo/TwinCAT-Agent
```

Install the TwinCAT plugin from that marketplace:

```text
/plugin install twincat@twincat-plugins
```

If Claude prompts you to apply plugin changes without restarting:

```text
/reload-plugins
```

Update the TwinCAT plugin:

```text
/plugin update
```

Select the TwinCAT agent:

```text
/agents
twincat
```

## 🧩 Install with GitHub Copilot CLI

Add this GitHub repo as a marketplace:

```bash
copilot plugin marketplace add ricciolo/TwinCAT-Agent
```

Install the plugin:

```bash
copilot plugin install ricciolo/TwinCAT-Agent
```

Start Copilot CLI:

```bash
copilot
```

Select the TwinCAT agent:

```text
/agent
twincat
```

Enable unrestricted approvals (recommended to avoid repeated prompts):

```text
/allow-all on
```

## 🚀 Quickstart

After the plugin is installed, the working flow is the same regardless of which supported assistant you use.

Open a terminal in the folder that contains the target `.plcproj`:

```bash
cd C:\Path\To\Your\TwinCAT\Project
```

Start your assistant:

```bash
copilot
```

or:

```bash
claude
```

Select a recent model (example: GPT-5.4 or Sonnet) using whatever model selection command or UI your assistant provides.

Select the `twincat` agent.

Copilot CLI:

```text
/agent
twincat
```

Claude Code:

```text
/agents
twincat
```

If you are using Copilot CLI, enable unrestricted approvals:

```text
/allow-all on
```

Start with a concrete prompt:

```text
Create a state machine in the example MAIN program.
```

From there, the `twincat` agent should resolve the project, initialize TwinCAT automation, inspect the PLC tree, and work through the change using the configured TwinCAT rules.

## 📚 Standalone Beckhoff InfoSys MCP

The Beckhoff InfoSys MCP server can also be used independently, without installing the rest of this repository.

- https://twincat-infosys-mcp.cristiancivera.com/

Available tools include:

- `infosys_get_page`: returns a documentation page rendered as clean Markdown
- `infosys_search`: searches across indexed Beckhoff InfoSys documentation pages
- `infosys_search_symbols`: searches IEC symbols such as function blocks, functions, structs, enums, and interfaces
- `infosys_search_code`: searches Structured Text examples, variable names, and code fragments inside the documentation

## License
This project is licensed under the [PolyForm Noncommercial License 1.0.0](LICENSE).

You are free to use, modify, and distribute this software for personal, academic, and non-commercial purposes (e.g., hobby projects, personal study, testing, and open-source research). 
For commercial usage (e.g. consulting, integration into paid industrial projects, or enterprise environments), a commercial license is required. Please contact me directly to discuss commercial licensing options.