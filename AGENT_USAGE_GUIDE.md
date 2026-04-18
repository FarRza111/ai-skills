# Custom Agent Usage Guide

## Your Custom Agents

You have created three custom agents that work together to research and summarize restaurant menu prices:

### 1. **RestaurantMenuResearcher** (Subagent)
- **Location**: `.github/agents/restaurant-menu-researcher.agent.md`
- **User-invocable**: No (only called by other agents)
- **Tools**: `web`, `read`
- **Purpose**: Fetches restaurant websites and extracts menu pricing data
- **Triggers**: "restaurant menu", "menu prices", "fetch menu", "restaurant prices"

### 2. **Summarizer** (Subagent)
- **Location**: `.github/agents/summarizer.agent.md`
- **User-invocable**: No (only called by other agents)
- **Tools**: None (conversational only)
- **Purpose**: Takes data and creates concise summaries
- **Triggers**: "summarize", "condense", "key points", "main points"

### 3. **RestaurantMenuOrchestrator** (Main Agent)
- **Location**: `.github/agents/restaurant-menu-orchestrator.agent.md`
- **User-invocable**: Yes (appears in agent picker)
- **Tools**: `web`, `agent`
- **Allowed Subagents**: RestaurantMenuResearcher, Summarizer
- **Purpose**: Coordinates the workflow—delegates to researcher, then to summarizer

---

## How to Use Your Agents

### In VS Code Copilot Chat:

1. **Open Copilot Chat** (Cmd+I or click the chat icon)

2. **Select the RestaurantMenuOrchestrator agent**:
   - Click the agent picker (shows "@" or agent icon)
   - Look for "RestaurantMenuOrchestrator" in the list
   - Select it

3. **Provide a restaurant URL**:
   ```
   Here is a restaurant link: https://humalbistro.ee/
   ```

### What Happens (Context Window):

```
┌─────────────────────────────────────────────────────────────┐
│ USER INPUT (You)                                            │
│ @RestaurantMenuOrchestrator                                 │
│ Here is a restaurant link: https://humalbistro.ee/         │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ ORCHESTRATOR AGENT (Context Window)                         │
│ - Has access to: web tools, agent invocation                │
│ - Reads agent instructions from:                            │
│   .github/agents/restaurant-menu-orchestrator.agent.md      │
│ - Knows it should:                                          │
│   1. Delegate to RestaurantMenuResearcher                   │
│   2. Then delegate to Summarizer                            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ SUBAGENT 1: RestaurantMenuResearcher (New Context)         │
│ - Spawned as subagent (isolated context)                    │
│ - Has access to: web, read tools                            │
│ - Receives: URL from orchestrator                           │
│ - Fetches website, extracts menu data                       │
│ - Returns: Structured menu with prices                      │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ ORCHESTRATOR (Back to main context)                         │
│ - Receives menu data from researcher                        │
│ - Prepares to invoke summarizer                             │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ SUBAGENT 2: Summarizer (New Context)                       │
│ - Spawned as subagent (isolated context)                    │
│ - Has access to: no tools (pure analysis)                   │
│ - Receives: Raw menu data                                   │
│ - Analyzes and condenses information                        │
│ - Returns: Summary with price ranges, key points            │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ ORCHESTRATOR (Final response)                               │
│ - Receives summary from Summarizer                          │
│ - Formats for user                                          │
│ - Cites source                                              │
└─────────────────────────────────────────────────────────────┘
                            ↓
┌─────────────────────────────────────────────────────────────┐
│ OUTPUT (To You)                                             │
│ Restaurant: Humal Bistro                                    │
│ Source: https://humalbistro.ee/                             │
│                                                             │
│ • Snacks: €4-€7                                             │
│ • Starters: €9-€16                                          │
│ • Mains: €18-€35                                            │
│ • Desserts: €9-€18                                          │
│                                                             │
│ Notable: French-inspired menu, features bouillabaisse...    │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Concepts

### Context Isolation
- Each subagent runs in its own context window
- Subagents don't see the full conversation history
- They receive only the input passed to them
- Return single output back to parent

### Tool Restrictions
- **RestaurantMenuResearcher**: Can only fetch web + read files
- **Summarizer**: Has NO tools (pure text analysis)
- **Orchestrator**: Can fetch web + invoke subagents

### Discovery via Description
- Agents are discovered by their `description` field keywords
- Parent agents match trigger phrases to decide which subagent to invoke
- If you say "summarize this", the Orchestrator knows to call Summarizer

---

## Testing Your Agents

### Quickstart:
1. Open Copilot Chat in VS Code
2. Type `@` and find "RestaurantMenuOrchestrator"
3. Paste a restaurant URL
4. Watch the workflow execute

### Expected Behavior:
- Should fetch the restaurant menu automatically
- Should extract all prices
- Should return a concise summary

### If Agents Don't Appear:
- Reload VS Code window (Cmd+Shift+P → "Reload Window")
- Check file path: must be `.github/agents/*.agent.md`
- Check YAML frontmatter syntax (no tabs, proper quotes)

---

## Advanced: User vs. Workspace Scope

### Current Setup (Workspace)
- **Location**: `/Users/farizrzayev/Desktop/far/.github/agents/`
- **Scope**: Only available in this workspace
- **Shared**: Yes, if committed to Git, team members get these agents

### Alternative (User Profile)
- **Location**: `~/Library/Application Support/Code/User/prompts/*.agent.md`
- **Scope**: Available in ALL workspaces
- **Shared**: No, personal to your VS Code profile

To make agents user-level, move them to your user profile folder:
```bash
mkdir -p ~/Library/Application\ Support/Code/User/prompts/
mv .github/agents/*.agent.md ~/Library/Application\ Support/Code/User/prompts/
```

---

## Next Steps

1. **Test the orchestrator** with different restaurant URLs
2. **Customize agent behavior** by editing the .agent.md files
3. **Add more agents** for other research tasks
4. **Check agent invocation** in Copilot Chat history to debug if needed

## Troubleshooting

| Issue | Solution |
|-------|----------|
| Agent doesn't appear in picker | Reload VS Code window |
| Agent ignores instructions | Check YAML syntax, ensure description has keywords |
| Subagent not invoked | Make sure `agents: [...]` lists subagent names correctly |
| Tool access denied | Check `tools: [...]` includes needed tool aliases |

---

**Your agents are now ready to use!** Start with: `@RestaurantMenuOrchestrator` + any restaurant URL.
