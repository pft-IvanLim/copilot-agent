# Multi-Agent Workflow

## Architecture: Hub-and-Spoke

All agents return to the **Orchestrator** (the central hub) when their stage is complete or when they're stuck. The Orchestrator reads conversation history and routes to the next agent.

```
                   ┌──────────────┐
              ┌───→│   Analyzer   │───┐
              │    └──────────────┘   │
              │    ┌──────────────┐   │
              ├───→│ Brainstormer │───┤
              │    └──────────────┘   │
              │    ┌──────────────┐   │
User ←──→ Orchestrator ←─────────────┘
              │    └──────────────┘
              ├───→│   Planner    │───┐
              │    └──────────────┘   │
              │    ┌──────────────┐   │
              ├───→│ Implementer  │←──┼──┐
              │    └──────────────┘   │  │ (tight loop)
              │    ┌──────────────┐   │  │
              └───→│Code Reviewer │───┘──┘
                   └──────────────┘
```

## Agents

| Agent | Model | Tools | Role |
|-------|-------|-------|------|
| **Orchestrator** | Default | None (handoffs only) | Central router — reads state, delegates |
| **Analyzer** | Default | read, search, web, execute, vscode | Gathers codebase context → Context Report |
| **Brainstormer** | Default | read, search, web | Discusses specs with user → Specification Report |
| **Planner** | Claude Opus 4.6 | read, search, web, agent, todo, vscode | Creates implementation plan (can call Analyzer as subagent) |
| **Implementer** | GPT-5.4 | read, edit, search, execute, todo, vscode | Senior Engineer — executes plan |
| **Code Reviewer** | Claude Opus 4.6 | read, search, execute, vscode | Senior Engineer — reviews implementation |

## Normal Workflow

1. **User** → Orchestrator (entry point)
2. **Orchestrator** → Analyzer (auto)
3. **Analyzer** → Orchestrator (context gathered)
4. **Orchestrator** → Brainstormer (auto)
5. **Brainstormer** ↔ User (discuss until confirmed) → Orchestrator
6. **Orchestrator** → Planner (auto)
7. **Planner** → Orchestrator (plan ready, user reviews)
8. **Orchestrator** → Implementer (user approves)
9. **Implementer** → Code Reviewer → Implementer (loop until APPROVED)
10. **Code Reviewer** → Orchestrator → Final Report to user

## "I'm Stuck" Flow

Any agent can hand back to Orchestrator when stuck. Orchestrator determines what's needed:
- Ambiguous specs → routes to Brainstormer
- Missing context → routes to Analyzer
- Plan issues → routes to Planner
