---
name: hello-world
description: Use when the user says "hello world" or asks to test plugin installation - responds with a friendly greeting to confirm the plugin is working
---

# Hello World

## Overview

A simple verification skill that confirms the plugin marketplace and installation pipeline are working correctly.

## When to Use

- The user says "hello world" or "test hello world"
- The user wants to verify that a plugin installed from the yeluyang-skills marketplace is working
- The user asks to check plugin installation status

## Workflow

1. Greet the user with a confirmation message
2. Report which marketplace and plugin this skill was loaded from

## Response

When triggered, respond with:

```
Hello, World! The **hello-world** plugin from the **yeluyang-skills** marketplace is installed and working correctly.
```
