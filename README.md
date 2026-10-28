# LLM SAST Skills

A collection of agent skills that turn your LLM coding assistant into a fully functional SAST scanner to find vulnerabilities in your codebase. Works natively with Claude Code, Codex, Opencode, Cursor and any other assistant that supports agent skills. No third-party tools required.

The main goal of this project is to find as many vulnerabilities as possible at the lowest possible cost.

![Process in Claude Code](demo.gif)

## How It Works

The `sast-scan` skill (Claude Code plugin) or `AGENTS.md` (Opencode and other IDEs) orchestrates the entire assessment workflow automatically. The assessment runs in three steps:

1. **Codebase Analysis** -- The `sast-analysis` skill maps the technology stack, architecture, entry points, data flows, and trust boundaries. It writes its findings to `sast/architecture.md`.

2. **Vulnerability Detection (parallel)** -- All 14 vulnerability detection skills run in parallel as subagents. Each skill follows a two-phase approach: first a recon/discovery phase to find candidate sections, then a verification phase to confirm exploitability. Results are written to `sast/*-results.md`.

3. **Report Generation** -- The `sast-report` skill consolidates all findings into a single `sast/final-report.md`, ranked by severity with full remediation guidance and dynamic test instructions.

## What It Detects

| Skill | Vulnerability Class |
|---|---|
| sast-analysis | Codebase reconnaissance, architecture mapping, threat modeling |
| sast-sqli | SQL Injection |
| sast-graphql | GraphQL injection |
| sast-xss | Cross-Site Scripting (XSS) |
| sast-rce | Remote Code Execution (command injection, eval, unsafe deserialization) |
| sast-ssrf | Server-Side Request Forgery |
| sast-idor | Insecure Direct Object Reference |
| sast-xxe | XML External Entity |
| sast-ssti | Server-Side Template Injection |
| sast-jwt | Insecure JWT implementations |
| sast-missingauth | Missing authentication and broken function-level authorization |
| sast-pathtraversal | Path / directory traversal |
| sast-fileupload | Insecure file upload |
| sast-businesslogic | Business logic flaws (price manipulation, workflow bypass, race conditions, etc.) |
| sast-report | Consolidated final report ranked by severity |


## Installation

### Claude Code (plugin, recommended)

Install once, then run it from inside any project — no copying needed:

```bash
/plugin marketplace add gdakuzak/sast-skills      # or a local path to this repo
/plugin install sast@sast-skills
```

Then, in the project you want to assess:

> /sast:sast-scan

or just ask "Run vulnerability scan". Individual checks can be run on their own too (e.g. `/sast:sast-sqli`), after `/sast:sast-analysis`.

It skips any step whose output file already exists, so you can safely re-run it after fixing issues. Add `sast/` to your `.gitignore`.

### Codex

```bash
codex plugin marketplace add gdakuzak/sast-skills
codex plugin add sast@sast-skills
```

Then, inside your project, run `codex` and ask "Run vulnerability scan" (or invoke the `sast-scan` skill).

### Cursor

```bash
git clone https://github.com/gdakuzak/sast-skills ~/.cursor/plugins/local/sast
```

Run **Developer: Reload Window**, then in your project ask "Run vulnerability scan". For the CLI: `cursor-agent --plugin-dir ~/.cursor/plugins/local/sast`.

### Other assistants (Opencode...)

Copy your project into the `sast-files` folder, then open `sast-files` as your workspace. `AGENTS.md` orchestrates the workflow using the skills in `sast-files/.agents/skills`.

```bash
cp -r /path/to/your/project sast-files/
```

> **Note:** If your project already contains an `AGENTS.md` file, remove it before running the assessment.

Then ask "Run vulnerability scan".

## Update

| Assistant | Command |
|---|---|
| Claude Code | `claude plugin marketplace update sast-skills && claude plugin update sast@sast-skills`, then restart |
| Cursor | `git -C ~/.cursor/plugins/local/sast pull`, then **Developer: Reload Window** |
| Codex | See `codex plugin --help`; re-running `codex plugin add sast@sast-skills` also works |

> **Maintainers:** bump `version` in `.claude-plugin/plugin.json`, `.codex-plugin/plugin.json` and `.cursor-plugin/plugin.json` on every release, otherwise clients won't see the update.

## Output

All output is written to a `sast/` folder in your project root:

| File | Description |
|---|---|
| `sast/architecture.md` | Technology stack, architecture, entry points, data flows |
| `sast/*-results.md` | Per-vulnerability-class findings with proof and remediation |
| `sast/final-report.md` | Consolidated report ranked by severity |

## Releases

Current version: **v1.0.0**. See [GitHub Releases](https://github.com/gdakuzak/sast-skills/releases) for all versions. Versioning follows [SemVer](https://semver.org/).

## Changelog

### v1.0.0

Changes compared to the upstream project:

- Packaged the skills as a Claude Code plugin (`.claude-plugin`), installable via `/plugin marketplace add`
- Added Codex (`.codex-plugin`) and Cursor (`.cursor-plugin`) plugin manifests
- Moved the skills from `sast-files/.claude/skills` to `skills/`; the `CLAUDE.md` orchestrator became the `sast-scan` skill
- Updated the installation and update instructions for each assistant

## Credits

This is a fork of [utkusen/sast-skills](https://github.com/utkusen/sast-skills) by [Utku Sen](https://github.com/utkusen), licensed under MIT. All vulnerability detection skills come from the original project.
