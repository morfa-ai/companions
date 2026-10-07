# Companions

Companions bring expert perspective into your everyday work: a second opinion on a decision, critique of plans, different framing for a problem, or actually novel ideas.

Most importantly, you can start using Companions in any harness of your choice [using Portal](#portal) or through our [remote MCP](#remote-mcp).

> [!TIP]
> For the best experience we recommend installing [Portal](#portal) – a small local MCP that connects to our service and handles tool calls for Companions.

> [!NOTE]
> If you are an AI agent setting up Companions for a user, read [Agent installation](#agent-installation) first.

## Portal

Companions Portal is a small binary running our MCP locally on your computer. It authenticates with our service and connects to your installed harnesses.

It functions as a layer between AI harness and our service. As your agent connects to it and sends tasks to Companions, when in response they require some tool calls, such as file reading or writing, Portal can intercept these tool calls and execute them without engaging your agent, sending response back to our service. Without it, requested tool calls are executed by your agent.

Install it by running the following command in your terminal:

macOS / Linux:
```bash
curl -fsSL https://get.morfa.ai/install.sh | sh
```

Windows (PowerShell):
```powershell
irm https://get.morfa.ai/install.ps1 | iex
```

The installer downloads the binary for your platform, verifies it, puts `portal` on your PATH and runs `portal setup` for you to authenticate, configure installation, tools and preferences. Run `portal setup` again to add a harness later. `portal update` updates Portal and offers to update the Companions plugin in Claude Code and Codex, and its skills in Hermes. Use `portal update --plugins-only` to check and update only the plugins and skills. Portal asks before running the host update commands; in a non-interactive terminal it prints manual instructions.

Portal installs the `portal` plugin from this marketplace into Claude Code and Codex when chosen during `setup`, and installs its skills individually into Hermes. If the `companions` plugin for [Remote MCP](#remote-mcp) is already installed, `setup` offers to disable it, as both expose the same tools.

Plugin updates use the Claude Code and Codex CLIs. If a CLI is missing, Portal prints manual guidance. In Codex Desktop, use the app's plugin interface, or install the Codex CLI and run the printed commands. Reload plugins in Claude Code, restart Codex or open a new thread, and start a new Hermes session after updating. Claude Desktop's remote connector is configured separately through [Connectors](#claude-desktop).

## Remote MCP

We provide a remote MCP server hosted at `https://api.morfa.ai/mcp` that your AI harness of choice can interact with.

Choose the harness you want to connect for instructions:
- [Claude Code](#claude-code)
- [Claude Desktop](#claude-desktop)
- [Hermes Agent](#hermes-agent)
- [Codex CLI](#codex-cli)

### Claude Code

Add the marketplace:
```
/plugin marketplace add neurowelt/companions
```

Then install the plugin:
```
/plugin install companions@companions-marketplace
```

After installation use `/mcp` command and navigate to `companions` MCP, where authentication process can be triggered via **Authenticate** flow.

### Claude Desktop

In your **Claude Desktop** application go to **Customize** > **Connectors**. Click the "**+**" icon and choose **Add custom connector**. Fill out the fields:
- **Name**: Companions
- **MCP URL**: https://api.morfa.ai/mcp

Confirm by clicking **Add**. Remember to enable each tool for that MCP by clicking the slider next to tool names.

### Hermes Agent

Connecting MCP to Hermes is easiest done using the following command:
```bash
hermes mcp add --url https://api.morfa.ai/mcp --auth oauth companions
```

### Codex CLI

You can add the marketplace by using `/plugins` command within Codex CLI, choosing **Add Marketplace** and pasting the link to this repository. Codex will automatically open browser's window for authentication.

If authentication expires later, reconnect the MCP without reinstalling the plugin:

```bash
codex mcp login companions --scopes openid,offline_access
```

After login, restart Codex or start a new thread so the Companions tools are loaded again.

## Agent installation

This section is for AI agents asked to install Companions on a user's behalf. There are two ways to install it, and which one applies depends on what you can do in your current environment:

| Your environment | Install | Read |
| --- | --- | --- |
| You can run shell commands on the user's machine | Portal | [Portal](#portal) |
| You have no terminal access (for example a desktop or web chat app) | `companions` plugin or connector over remote MCP | [Remote MCP](#remote-mcp), then the subsection for the user's harness |

Portal is the preferred path, so choose it whenever you have a terminal. It executes the tool calls Companions request locally, instead of routing each one through you. Fall back to Remote MCP only when you cannot run the installer, and do not ask the user to choose between the two: you already know what your environment allows.

**Installing Portal.** Run the installer command for the user's operating system from [Portal](#portal). The installer finishes by running `portal setup`, which is interactive: the user signs in and chooses which harnesses, tools and preferences to configure. Those choices belong to the user, so let them answer the prompts. If your shell cannot show interactive prompts, give the user the installer command to run in their own terminal.

**Installing over Remote MCP.** Open the subsection of [Remote MCP](#remote-mcp) that matches the harness the user is running and follow it. Some of those steps happen in the application's interface or in a browser sign-in that you cannot perform yourself. Give the user those steps exactly as written, with the endpoint `https://api.morfa.ai/mcp`.

**Install one, not both.** The `portal` and `companions` plugins expose the same tools. If `companions` is already installed and you are adding Portal, `portal setup` offers to disable it, and the user should accept.

**After installing.** New tools only load once the harness reloads: reload plugins in Claude Code, restart Codex or open a new thread, or start a new Hermes session. Then run the plugin's `setup` command, which checks connectivity and lists the available Companions without spending credit.
