# claude-md

Generate or update CLAUDE.md by deep-analyzing a codebase to extract stable knowledge (architecture, conventions, design philosophy) and volatile operational details (build, test, deploy commands) into agent-optimized project documentation.

## Components

- **Skill**: `claude-md` — analyzes a codebase and generates/updates CLAUDE.md with agent-optimized project documentation

## Installation

```
/plugin marketplace add yeluyang/skills
/plugin install claude-md@yeluyang-skills
```

Then restart Claude Code.

## Usage

Invoke the skill by asking Claude to generate or update your project's CLAUDE.md, or when onboarding to a new codebase.
