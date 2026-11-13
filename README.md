# Agent Team

Turn a task into a checked, scheduled plan for a team of AI agents, then keep
those agents in step through plain files in one Git repository. A free skill
for AI agents (Claude Code, Codex CLI, Cursor and any agent that reads
`SKILL.md`).

- **Plans the work.** Your agent splits a task into 3 to 8 steps, each with a
  role, a model tier (`fast`, `balanced`, `deep`), its dependencies and one
  line that says how to verify it.
- **Checks and schedules it.** `flow.py` rejects broken plans (missing
  dependencies, cycles, steps with no way to verify) and sorts the rest into
  waves that can run in parallel.
- **Hands work between agents.** `team.py` gives every agent an inbox, and
  every handoff is a small message file: send, claim, done, a status board and
  a consistency check.
- **Nothing to run or host.** No server, no accounts, no API keys. Two
  standard-library Python scripts that only touch local files; Git carries
  the messages.

## Install

With the skills CLI:

```bash
npx skills add AbdullahRathoreVA/agent-team
```

Or copy this folder into your agent's skills directory (for Claude Code:
`~/.claude/skills/agent-team`). Then ask your agent: "Plan this as an agent
workflow and tell me who does what with which model."

## A real run

[examples/plan.json](examples/plan.json) is a five-step plan for "add a
pricing page and link it from the home page". Checking it:

```bash
python scripts/flow.py check examples/plan.json
```

```json
{"ok": true, "steps": 5, "errors": [], "warnings": []}
```

Scheduling it:

```bash
python scripts/flow.py waves examples/plan.json
```

returns four waves: `copy` and `find-nav` together, then `page`, then `link`,
then `review`, with at most 2 steps in parallel and 2 fast, 2 balanced and
1 deep step. Each step's brief for a worker agent comes from
`python scripts/flow.py brief examples/plan.json <step>`.

## Hand work to another agent

```bash
python scripts/team.py init alice-agent bob-agent
```

```bash
python scripts/team.py send alice-agent bob-agent "Build the pricing page" body.md P2
```

```bash
python scripts/team.py inbox bob-agent
```

```bash
python scripts/team.py claim <message-id> bob-agent
```

```bash
python scripts/team.py done <message-id> bob-agent "pricing.html added, linked from the nav"
```

`python scripts/team.py board` shows who is doing what, and
`python scripts/team.py check` reports anything out of step. The script never
runs git itself: you pull and push as usual. To get a GitHub issue for every
message, see [references/github-action.md](references/github-action.md).

## Limits

- Agents see a message only after a `git pull`. It is turn-based, not live chat.
- It trusts whoever can push to the repository. It is a workflow, not access control.
- It coordinates work. It does not start or run other agents.

## More free skills

| Skill | What it does |
|---|---|
| [Aether Engine](https://github.com/AbdullahRathoreVA/aether-engine-skill) | Finds people asking for what you sell, plans and publishes audited SEO pages, checks drafts for filler |
| [Aether Listen](https://github.com/AbdullahRathoreVA/aether-listen) | Finds and ranks public posts from people who need your product |
| [Secret Scan](https://github.com/AbdullahRathoreVA/secret-scan) | Finds leaked keys in a repository's Git history before you make it public |
| **Agent Team** | This one |

## Licence

MIT. See [LICENSE](LICENSE).
