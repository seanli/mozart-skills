# mozart-skills

Skills for the [Mozart](https://github.com/seanli/mozart) agent orchestrator, distributed via [skills.sh](https://skills.sh/).

## Available Skills

| Skill | Reference | Description |
|-------|-----------|-------------|
| [agent-authoring](skills/agent-authoring/SKILL.md) | `seanli/mozart-skills@agent-authoring` | Agent authoring, .soul file writing, and multi-agent workflow design |
| [ragie-rag](skills/ragie-rag/SKILL.md) | `seanli/mozart-skills@ragie-rag` | Retrieve content from Ragie.ai RAG platform |

## Usage

In your `.soul` file:

```
SKILL seanli/mozart-skills@agent-authoring
```

Or install at runtime:

```
install_skill seanli/mozart-skills@agent-authoring
```

## Structure

```
skills/
├── agent-authoring/
│   └── SKILL.md
└── ragie-rag/
    └── SKILL.md
```

Each skill is a directory containing a `SKILL.md` file with the skill's instructions, workflows, and examples.
