# coding-team

Inspired by Stavros's [How I write software with LLMs](https://www.stavros.io/posts/how-i-write-software-with-llms/).

A Claude Code plugin that runs a multi-agent development team: an architect plans the work, a developer implements it, and two independent reviewers (Claude + OpenAI Codex) approve it before moving on.

## Prerequisites

- [Claude Code](https://claude.ai/download) v1.0.33+
- [OpenAI Codex CLI](https://github.com/openai/codex) installed and authenticated (used by the `codex-reviewer` agent)

## Install

```bash
/plugin marketplace add maxdubakov/multi-agent-development-claude-plugin
/plugin install coding-team@multi-agent-development
```

Or test locally:

```bash
claude --plugin-dir ./path-to-this-repo
```

## How it works

The plugin sets `architect` as the default agent. When you describe what you want to build, the architect goes in this loop:

1. **Discovery** -- architect asks clarifying questions, then presents requirements, constraints, success criteria, and non-goals for your approval.
2. **Planning** -- architect breaks the work into tasks and writes Task Brief files to `.coding-team/<topic>/`.
3. **Implementation** -- `developer` implements one task at a time, following the Task Brief.
4. **Parallel review** -- `code-reviewer` (Opus) and `codex-reviewer` (Codex CLI) review the diff independently.
5. **Feedback loop** -- if either reviewer requests changes, architect verifies if the ask is valid, and decides if they are needed. If yes - it sends developer back with to implement them.
6. **Next task** -- only after both reviewers approve does the architect move to the next task.

## Agents

| Agent | Model | Role |
|-------|-------|------|
| `architect` | Opus | Orchestrates the workflow, writes Task Briefs, never touches code |
| `developer` | Sonnet | Implements tasks, runs validation, reports back |
| `code-reviewer` | Opus | Reviews diffs for correctness, security, simplicity |
| `codex-reviewer` | Haiku (proxy) | Relays review to OpenAI Codex CLI and returns its output |
| `repo-scout` | Sonnet | Scans the repo and maintains `ARCHITECTURE.md` |

## Permissions

Plugins cannot ship pre-configured permissions, so you'll need to allow these yourself on first use. The agents need:

- **Bash commands**: `git diff`, `git log`, `git status`, `git rev-parse`, and your project's lint/test/build commands
- **Read/Write/Edit**: for modifying source files and writing Task Briefs
- **Glob/Grep**: for codebase exploration

The easiest approach: when Claude asks for permission during the first run, allow and choose "always allow" for the tools you're comfortable with.

## License

MIT
