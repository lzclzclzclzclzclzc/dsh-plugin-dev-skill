# DSH Plugin Development Skill

A Codex skill for building, debugging, and packaging plugins for [DeepSeek Harness (DSH)](https://github.com/deepseek-ai/deepseek-harness), based on its official development documentation and API contracts.

The skill guides Codex through choosing an extension point, implementing a plugin, checking compatibility with the target DSH version, and verifying that the plugin loads and behaves correctly in the intended profile.

## What it covers

- **Model-facing tools:** `defineTool`, parameter schemas, canonical output values, rendering, cancellation, and PTC integration.
- **Configuration and services:** Schemastery `Config`, Cordis dependency injection, lifecycle management, events, and cleanup.
- **LLM adapters:** provider registration, streaming contracts, model capabilities, errors, and cancellation.
- **Web extensions:** Host/Client separation, client bundles, settings forms, and tool presentation.
- **Packaging and troubleshooting:** `dsh.bundle`, patch files, profiles, linked dependencies, distribution, HMR, and uninstall behavior.

The workflow supports both standalone third-party plugins and packages inside the official Harness repository. It adapts the checks to the project instead of imposing monorepo conventions on every plugin.

## Install in Codex

Install the repository as a directory named `dsh-plugin-dev` under `$CODEX_HOME/skills`. When `CODEX_HOME` is unset, the default location is `~/.codex/skills`.

### Ask Codex to install it

```text
Install the Codex skill from https://github.com/lzclzclzclzclzclzc/dsh-plugin-dev-skill.
The skill is at the repository root. Install it as dsh-plugin-dev.
```

### Install with Git

On Windows, run in PowerShell:

```powershell
$skillRoot = if ($env:CODEX_HOME) {
    Join-Path $env:CODEX_HOME 'skills'
} else {
    Join-Path $env:USERPROFILE '.codex\skills'
}
git clone https://github.com/lzclzclzclzclzclzc/dsh-plugin-dev-skill.git (Join-Path $skillRoot 'dsh-plugin-dev')
```

On macOS or Linux:

```sh
git clone https://github.com/lzclzclzclzclzclzc/dsh-plugin-dev-skill.git \
  "${CODEX_HOME:-$HOME/.codex}/skills/dsh-plugin-dev"
```

If that directory already contains an installation, inspect it before replacing it. To update a clean Git-based installation, run `git pull --ff-only` from its directory.

The skill is available to Codex on the next turn. It contains instructions and references and requires no separate build step. Developing and running a DSH plugin still requires the tools and dependencies appropriate to the target project.

## Usage

Invoke the skill explicitly with `$dsh-plugin-dev`, followed by your request:

```text
Use $dsh-plugin-dev to create a standalone DSH plugin that exposes a greeting tool.
Include the bundle manifest and instructions for testing it in the Web profile.
```

```text
Use $dsh-plugin-dev to investigate why my installed plugin stays PENDING
and its tool is unavailable to the agent.
```

```text
Use $dsh-plugin-dev to add a settings page to this plugin.
Check the Client API against my installed DSH version before implementing it.
```

Automatic invocation is also enabled for relevant DSH plugin development requests. The skill instructions and reference files are currently written in Chinese; Codex is instructed to communicate in the user's language.

## Repository contents

| File | Purpose |
| --- | --- |
| [SKILL.md](SKILL.md) | Skill discovery metadata, development workflow, and troubleshooting entry points |
| [agents/openai.yaml](agents/openai.yaml) | Codex display metadata and default prompt |
| [references/tools.md](references/tools.md) | Tool API contracts, examples, and verification guidance |
| [references/packaging.md](references/packaging.md) | Bundle structure, profile installation, dependency resolution, and distribution |
| [references/extensions.md](references/extensions.md) | Configuration, services, events, LLM adapters, and Web extensions |
| [references/sources.md](references/sources.md) | Pinned official sources, version caveats, and community skill comparison |

## Documentation baseline

The guidance was checked on **September 28, 2026**, against official Harness commit [`21638c5`](https://github.com/deepseek-ai/deepseek-harness/commit/21638c56315ae6a2b552d6091945d3144c9af32e).

DSH APIs can change. The skill directs Codex to prefer the target version's types, implementation, and matching official documentation when they differ from this snapshot. It also distinguishes static checks from successful runtime verification and reports missing prerequisites explicitly.

The [source reference](references/sources.md) records the official documents consulted and the three community skills evaluated when creating this skill. This is a community project.

## License

See [LICENSE](LICENSE) for the MIT license retained for the material adapted from DeepSeek's official documentation and examples.
