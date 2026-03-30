---
name: new-plugin
description: Create a new Claude Code plugin from drafts folder requirements. Reads all files in drafts/, analyzes prompts and specifications, then scaffolds a complete plugin with proper structure, plugin.json, and SKILL.md files.
allowed-tools: Read, Glob, Write, Bash
---

You are a Claude Code plugin developer. Your task is to create a new plugin based on materials in the `drafts/` folder.

## Step 1 — Read all draft materials

Use Glob to find all files in the `drafts/` directory:
```
drafts/**/*
```

Read every file found. These are prompts, requirements, and specifications for the new plugin.

## Step 2 — Analyze and plan

Based on the draft materials, determine:
- Plugin name (lowercase, hyphens only, max 64 chars)
- Plugin description (what it does, when to use it)
- What skills/commands the plugin needs
- What tools each skill requires

## Step 3 — Scaffold the plugin

Create the plugin in `plugins/<plugin-name>/` with this exact structure:

```
plugins/<plugin-name>/
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── <skill-name>/
        └── SKILL.md
```

### plugin.json template:
```json
{
  "name": "<plugin-name>",
  "description": "<what the plugin does>",
  "version": "1.0.0",
  "author": { "name": "claude-plugins" },
  "repository": "https://github.com/your-username/claude-plugins",
  "license": "MIT"
}
```

### SKILL.md template:
```
---
name: <skill-name>
description: <concise description — front-load keywords, max 250 chars>
allowed-tools: <comma-separated list of tools needed>
---

<Detailed instructions for Claude to execute this skill>
```

## Step 4 — Register in marketplace

Add the new plugin entry to `.claude-plugin/marketplace.json` in the `plugins` array:
```json
{
  "name": "<plugin-name>",
  "source": "./plugins/<plugin-name>",
  "description": "<plugin description>",
  "version": "1.0.0",
  "category": "<category>",
  "tags": ["<relevant>", "<tags>"],
  "author": { "name": "claude-plugins" }
}
```

## Step 5 — Report

After creating all files, report:
- Plugin name and location
- Skills created with their slash commands
- How to test: `claude --plugin-dir ./plugins/<plugin-name>`
- Reminder that the user should clear the `drafts/` folder before next plugin
