# Copilot Agent Team

A multi-agent workflow for VS Code Copilot that automates the full development lifecycle: analysis → brainstorming → planning → implementation → code review.

## Quick Start

### 1. Clone

```bash
git clone <repo-url> copilot-agent
```

### 2. Symlink into your workspace

Create a symbolic link from your project's `.github/agents/` to this repo's agent files:

```bash
# From your project root
ln -s /path/to/copilot-agent/agents .github/agents
```

Or copy the files directly:

```bash
cp -r /path/to/copilot-agent/agents/*.agent.md .github/agents/
```

### 3. Select Orchestrator

1. Open VS Code Chat (`Ctrl+Alt+I`)
2. Click the **agent dropdown** at the bottom of the chat input
3. Select **"Orchestrator"**
4. Type your task and press Enter

The Orchestrator will automatically manage the full workflow.

---

## Architecture

The **Orchestrator** is the central brain. It stays active throughout the entire workflow, calling specialized sub-agents automatically via the `agent` tool and using `askQuestion` for user checkpoints.

```
User prompt → Orchestrator (stays active)
    │
    ├─ (1) agent → Analyzer ............ gather codebase context
    ├─ (2) agent → Brainstormer ........ discuss specs with user (via askQuestion)
    ├─ (3) agent → Planner ............. create implementation plan
    ├─ (4) askQuestion → user .......... "Approve plan?"
    ├─ (5) agent → Implementer ......... write code
    ├─ (6) agent → Code Reviewer ....... review implementation
    ├─ (7) loop (5)-(6) if needed ...... fix issues automatically
    └─ (8) askQuestion → user .......... "Done! What next?"
```

**Zero handoff clicks.** All transitions are automated via subagent calls. User interaction happens only through `askQuestion` prompts.

## Agents

| Agent | Model | Tools | Role |
|-------|-------|-------|------|
| **Orchestrator** | Default | agent, vscode, read, search | Central brain — calls subagents, manages workflow |
| **Analyzer** | Default | read, search, web, execute | Gathers codebase context → Context Report |
| **Brainstormer** | Default | read, search, web, vscode | Discusses specs with user (via askQuestion) → Specification Report |
| **Planner** | Claude Opus 4.6 | read, search, web, agent, todo | Creates detailed implementation plan (can call Analyzer) |
| **Implementer** | GPT-5.4 | read, edit, search, execute, todo | Senior Engineer — executes plan precisely |
| **Code Reviewer** | Claude Opus 4.6 | read, search, execute | Senior Engineer — reviews for correctness, bugs, security |

## User Interaction Points

| Step | What Happens | User Action |
|------|-------------|-------------|
| Analysis | Orchestrator calls Analyzer | None (auto) |
| Brainstorming | Brainstormer asks questions via askQuestion | Answer questions, confirm specs |
| Plan Review | Orchestrator shows plan, asks for approval | Click Approve / Adjust / Stop |
| Implementation | Orchestrator calls Implementer | None (auto) |
| Code Review | Orchestrator calls Code Reviewer | None (auto) |
| Fix Loop | Implementer ↔ Code Reviewer iterate | None (auto) |
| Final Report | Orchestrator presents results | Acknowledge |

## Exception Handling

If any subagent gets stuck, the Orchestrator handles it:

| Problem | Orchestrator Action |
|---------|-------------------|
| Missing context | Re-calls Analyzer with specific questions |
| Ambiguous specs | Re-calls Brainstormer for clarification |
| Plan needs revision | Re-calls Planner with updated context |
| Implementation blocked | Asks user via askQuestion |
