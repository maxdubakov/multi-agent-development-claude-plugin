# coding-team

Inspired by Stavros's [How I write software with LLMs](https://www.stavros.io/posts/how-i-write-software-with-llms/).

A Claude Code plugin that runs a multi-agent development team: an architect plans the work, a developer implements it, and two independent reviewers (Claude + OpenAI Codex) approve it before moving on.

## Prerequisites

- [Claude Code](https://claude.ai/download)
- [OpenAI Codex CLI](https://github.com/openai/codex) installed and authenticated (used by the `codex-reviewer` agent)

## Install

```bash
/plugin marketplace add maxdubakov/multi-agent-development-claude-plugin
/plugin install coding-team@multi-agent-development
```

## Permissions

Plugins cannot ship pre-configured permissions, so you'll need to allow these yourself on first use.

## How it works

The plugin sets `architect` as the default agent. When you describe what you want to build, the architect goes in this loop:

1. **Planning** -- this step will take MOST of your time. It is a back and forth conversation about implementation with the `architect` agent. It will ask clarifying questions, suggest specific implementation path, and ask your opinion.
2. **Task Definition** -- once implementation path is accepted, `architect` splits it into multiple tasks. Each is written as the implementation progresses in `.coding-team/<feature>` folder.
3. **Implementation** -- `developer` implements one task at a time, following the Task Brief.
4. **Parallel review** -- `code-reviewer` (Opus) and `codex-reviewer` (Codex CLI) review the diff independently.
5. **Feedback loop** -- if either reviewer requests changes, architect verifies if the ask is valid, and decides if they are needed. If yes - it sends developer back with to implement them.
6. **Next task** -- only after both reviewers approve does the architect move to the next task.

### Example of prompt:
```markdown
Hey, I want to add an action to fetch an entity/view by URL.

Actions are stored here @ai-answer/actions. An action can be later called by our mcp server (which is not in that directory), or by a tool of an agent (for example, @ai-answer/agents/schema/tools/createSpace/createSpace.d.ts).

We had somewhere in @ai-answer/context/ logic to fetch a single entity by id.

We can get entity id by url (for example, provided such url, we can get public id, and from it, we can do basically the same): htt ps://the.fibery.io/Product_Management/feature/Add-an-action-to-fetch-an-entity-view-by-URL-7585
```

## Agents

| Agent | Model | Role |
|-------|-------|------|
| `architect` | Opus | Orchestrates the workflow, writes Task Briefs, never touches code |
| `developer` | Sonnet | Implements tasks, runs validation, reports back |
| `code-reviewer` | Opus | Reviews diffs for correctness, security, simplicity |
| `codex-reviewer` | Haiku (proxy) | Relays review to OpenAI Codex CLI and returns its output |
| `repo-scout` | Sonnet | Scans the repo and maintains `ARCHITECTURE.md` |

## License

MIT
