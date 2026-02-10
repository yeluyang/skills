# hello-world

A sample plugin to verify that the yeluyang-skills marketplace installation flow works correctly.

## Components

- **Skill**: `hello-world` — triggers when the user says "hello world" or wants to test the installation
- **Command**: `/hello` — responds with a confirmation message

## Installation

```
/plugin marketplace add yeluyang/skills
/plugin install hello-world@yeluyang-skills
```

Then restart Claude Code.

## Usage

After installation, either:

- Say "hello world" to trigger the skill
- Run `/hello` to invoke the command
