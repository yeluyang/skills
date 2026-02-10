# codebase-walkthrough

Guided codebase understanding — detect project type, map dependencies and external systems, extract core concepts, enumerate API surfaces, and deep-dive into request paths interactively.

## Components

- **Skill**: `codebase-walkthrough` — structured 7-step interactive workflow for understanding an unfamiliar codebase, with adaptive routing based on project type

## Installation

```
/plugin marketplace add yeluyang/skills
/plugin install codebase-walkthrough@yeluyang-skills
```

Then restart Claude Code.

## Usage

Invoke the skill when onboarding to an unfamiliar codebase or exploring a new project before contributing. The skill will guide you through 7 steps (adapted to project type):

1. Project Overview
2. Dependencies
3. Build & Test
4. Core Concepts
5. API Surface
6. Deep Dive
7. Quick Reference Card
