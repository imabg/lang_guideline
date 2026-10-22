---
name: "action-00-project-setup"
title: "Project Setup & Environment Initialization Action"
version: "1.0.0"
agent_role: "DevOps & Repository Architect Agent"
target_scope: "Scaffold, plugin manifests, skill registrations, GitHub Copilot instructions, and validation harness"
target_languages: ["go", "json", "markdown"]
phase: 0
handover_input: "Empty directory or clean repository initialization"
handover_output: "Fully scaffolded multi-agent plugin environment with valid marketplace manifests and skill registries"
next_action: "actions/01-go-core-data-structures.action.md"
---

# Action 00: Project Setup & Multi-Platform Plugin Environment Initialization

## 1. Executive Mission & Agent Role

You are **Agent 0: DevOps & Repository Architect**.
Your mission is to construct, configure, and validate the complete repository infrastructure for the **Modern Language Guidelines (`lang_guideline`)** project from scratch. Any AI agent given this action file must be able to bootstrap the repository from an empty state into an enterprise-grade multi-agent extension workspace.

### Target Platform Support
The project must seamlessly integrate into all major AI coding platforms:
- **Claude Code / Anthropic Agent** (`.claude-plugin/`, `plugin/.claude-plugin/`)
- **Cursor IDE Agent** (`.cursor-plugin/`, `plugin/.cursor-plugin/`)
- **OpenAI Codex / Custom Agents** (`plugin/.codex-plugin/`)
- **Google Antigravity & Agent CLI** (`.agents/plugins/`, `plugin/extension.json`)
- **GitHub Copilot Workspace & Prompts** (`.github/instructions/`, `.github/prompts/`, `.github/copilot-instructions.md`)

---

## 2. Target Directory Hierarchy

An agent executing this setup MUST construct the following canonical directory structure:

```
lang_guideline/
├── .agents/
│   └── plugins/
│       └── marketplace.json            # Antigravity/Agent CLI marketplace registry
├── .claude-plugin/
│   └── marketplace.json                # Claude Code marketplace manifest
├── .cursor-plugin/
│   └── marketplace.json                # Cursor plugin marketplace manifest
├── .github/
│   ├── copilot-instructions.md         # Global Copilot repository instructions
│   ├── instructions/
│   │   ├── _template.instructions.md   # Standard language guideline template
│   │   └── go.instructions.md          # Comprehensive Go language rules
│   └── prompts/
│       ├── modernize-go.prompt.md      # Copilot prompt: /modernize-go
│       └── review-go.prompt.md         # Copilot prompt: /review-go
├── actions/
│   ├── README.md                       # Multi-agent orchestrator & execution guide
│   ├── 00-project-setup.action.md      # (This file) Project initialization runbook
│   ├── 01-go-core-data-structures.action.md # Phase 1: Core & Collections
│   ├── 02-go-concurrency-systems.action.md  # Phase 2: Concurrency & Systems
│   └── 03-go-errors-testing-observability.action.md # Phase 3: Errors, Testing & Slog
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json                 # Claude plugin definition
│   ├── .codex-plugin/
│   │   └── plugin.json                 # Codex/OpenAI plugin definition
│   ├── .cursor-plugin/
│   │   └── plugin.json                 # Cursor plugin definition
│   ├── extension.json                  # Antigravity/Agent extension descriptor
│   └── skills/
│       └── modern-go/
│           └── SKILL.md                # Short-named modern Go skill definition
├── .gitignore
└── README.md
```

---

## 3. Step-by-Step Execution Protocol for Agents

### Step 1: Create Base Directory Hierarchy
Execute directory creation to ensure all target paths exist:

```bash
mkdir -p .agents/plugins \
         .claude-plugin \
         .cursor-plugin \
         .github/instructions \
         .github/prompts \
         actions \
         plugin/.claude-plugin \
         plugin/.codex-plugin \
         plugin/.cursor-plugin \
         plugin/skills/modern-go
```

### Step 2: Generate Marketplace Manifests

#### 2.1 `.agents/plugins/marketplace.json`
```json
{
  "name": "lang-guidelines-marketplace",
  "interface": {
    "displayName": "Modern Language Guidelines Marketplace"
  },
  "plugins": [
    {
      "name": "modern-go-guidelines",
      "source": {
        "source": "local",
        "path": "./plugin"
      },
      "policy": {
        "installation": "AVAILABLE",
        "authentication": "ON_INSTALL"
      },
      "category": "Developer Tools"
    }
  ]
}
```

#### 2.2 `.claude-plugin/marketplace.json`
```json
{
  "name": "lang-guidelines-claude-marketplace",
  "owner": {
    "name": "Abhay Goswami",
    "email": "abhaygoswami676@gmail.com"
  },
  "metadata": {
    "description": "Marketplace for modern language coding agent guidelines and skills",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "modern-go-guidelines",
      "source": "./plugin"
    }
  ]
}
```

#### 2.3 `.cursor-plugin/marketplace.json`
```json
{
  "name": "lang-guidelines-cursor-marketplace",
  "owner": {
    "name": "Abhay Goswami",
    "email": "abhaygoswami676@gmail.com"
  },
  "metadata": {
    "description": "Marketplace for modern language coding agent guidelines and skills",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "modern-go-guidelines",
      "source": "./plugin"
    }
  ]
}
```

---

### Step 3: Generate Plugin Manifests & Descriptors

#### 3.1 `plugin/extension.json`
```json
{
  "name": "modern-go-guidelines",
  "description": "Helps AI coding agents write modern, idiomatic Go."
}
```

#### 3.2 `plugin/.claude-plugin/plugin.json`
```json
{
  "name": "modern-go-guidelines",
  "description": "Helps Claude Code and coding agents write modern, idiomatic Go matching the project's Go version.",
  "version": "1.0.0",
  "author": {
    "name": "Abhay Goswami",
    "email": "abhaygoswami676@gmail.com"
  },
  "keywords": ["go", "golang", "go-best-practices", "guidelines", "modern-go"],
  "skills": "./skills/"
}
```

#### 3.3 `plugin/.cursor-plugin/plugin.json`
```json
{
  "name": "modern-go-guidelines",
  "displayName": "Modern Go Guidelines",
  "version": "1.0.0",
  "description": "Helps coding agents write modern, idiomatic Go matching the project's Go version.",
  "author": {
    "name": "Abhay Goswami",
    "email": "abhaygoswami676@gmail.com"
  },
  "homepage": "https://github.com/imabg/lang_guideline",
  "repository": "https://github.com/imabg/lang_guideline",
  "license": "Apache-2.0",
  "keywords": ["go", "golang", "go-best-practices", "guidelines", "modern-go"],
  "skills": "./skills/"
}
```

#### 3.4 `plugin/.codex-plugin/plugin.json`
```json
{
  "name": "modern-go-guidelines",
  "version": "1.0.0",
  "description": "Helps coding agents write modern, idiomatic Go matching the project's Go version.",
  "author": {
    "name": "Abhay Goswami",
    "email": "abhaygoswami676@gmail.com",
    "url": "https://github.com/imabg/lang_guideline"
  },
  "homepage": "https://github.com/imabg/lang_guideline",
  "repository": "https://github.com/imabg/lang_guideline",
  "license": "Apache-2.0",
  "keywords": ["go", "golang", "go-best-practices", "guidelines", "modern-go"],
  "skills": "./skills/",
  "interface": {
    "displayName": "Modern Go Guidelines",
    "shortDescription": "Write idiomatic Go for the project's Go version",
    "longDescription": "Detect the project's Go version and apply modern language and standard-library guidance available through that version.",
    "developerName": "Abhay Goswami",
    "category": "Developer Tools",
    "capabilities": ["Read", "Write"],
    "websiteURL": "https://github.com/imabg/lang_guideline",
    "defaultPrompt": [
      "Apply modern Go guidelines for this project."
    ]
  }
}
```

---

### Step 4: Register Skill with Short Name (`modern-go`)

Ensure `plugin/skills/modern-go/SKILL.md` is populated with concise naming:

```markdown
---
name: modern-go
description: Use modern Go guidelines whenever writing, modifying, fixing, or refactoring Go code. Apply version-specific modern idioms to generated changes.
---

# Modern Go Guidelines Skill (`modern-go`)

Always write modern, idiomatic Go code. Follow these rules whenever creating, refactoring, or reviewing Go code.

## 1. Version Detection
1. Check the `go` directive in `go.mod` (or `go.work`) to identify the project's target Go version.
2. Only suggest language features and standard library additions available in or before that version.
3. If no `go.mod` is found, default to Go 1.21 idioms.

...
```

---

### Step 5: Verification & Self-Correction Test Suite

Execute these verification commands to ensure the environment is 100% compliant:

```bash
# 1. Validate all JSON files for strict syntax
for f in $(find . -name "*.json" -not -path "*/.*/*" -o -path "./.cursor-plugin/*" -o -path "./.claude-plugin/*" -o -path "./.agents/*" -o -path "./plugin/.*/*"); do
  if [ -f "$f" ]; then
    node -e "JSON.parse(require('fs').readFileSync('$f'))" && echo "VALID: $f" || (echo "INVALID JSON: $f" && exit 1)
  fi
done

# 2. Check skill directory structure
if [ -f "plugin/skills/modern-go/SKILL.md" ]; then
  echo "VALID: Skill modern-go found"
else
  echo "ERROR: Missing plugin/skills/modern-go/SKILL.md" && exit 1
fi

# 3. Check action files are present
for act in actions/00-project-setup.action.md \
           actions/01-go-core-data-structures.action.md \
           actions/02-go-concurrency-systems.action.md \
           actions/03-go-errors-testing-observability.action.md \
           actions/README.md; do
  if [ -f "$act" ]; then
    echo "VALID: $act exists"
  else
    echo "MISSING: $act" && exit 1
  fi
done
```

---

## 4. Handover to Phase 1 Agent

Upon successful verification, output `handover-phase-0-report.md`:
```markdown
# Phase 0 Setup Handover Report
- Workspace: Initialized and validated
- Platform Manifests: Claude, Cursor, Codex, Antigravity, Copilot registered
- Skill Name: `modern-go` (Active)
- Status: READY FOR AGENT 1
```

Next, invoke **Agent 1** with `actions/01-go-core-data-structures.action.md`.
