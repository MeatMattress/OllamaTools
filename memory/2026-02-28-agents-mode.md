# Session: 2026-02-28 04:14:30 UTC

- **Session Key**: agent:opus:discord:direct:313070612643840001
- **Session ID**: 5326af29-725e-4dd6-9b28-2ca41e496bb2
- **Source**: discord

## Conversation Summary

assistant: ✅ New session started · model: anthropic/claude-opus-4-6
user: A new session was started via /new or /reset. Execute your Session Startup sequence now - read the required files before responding to the user. Then greet the user in your configured persona, if one is provided. Be yourself - use your defined voice, mannerisms, and mood. Keep it to 1-3 sentences and ask what they want to do. If the runtime model differs from default_model in the system prompt, mention the default model. Do not mention internal steps, files, tools, or reasoning.
assistant: Hey Trevor. Fresh session, same day — I've got context from earlier. What are we working on?
user: [Queued messages while agent was busy]

---
Queued #1
How does agents mode work?
assistant: Here's how agents mode works in OpenClaw:

**The core idea:** you can run multiple isolated AI "brains" in a single Gateway process, each with its own personality, workspace, auth, and session history.

**What makes an agent:**
- Its own **workspace** (SOUL.md, AGENTS.md, USER.md — separate persona and files)
- Its own **state directory** (auth profiles, model config)
- Its own **session store** (chat history doesn't cross between agents)

**Routing — how messages find the right agent:**
You define **bindings** that map inbound messages to agents based on channel, account, peer (specific DM/group), guild, etc. Most-specific match wins. Examples:
- Route all WhatsApp → "chat" agent (Sonnet), all Telegram → "opus" agent (Opus)
- Route a specific WhatsApp group → "family" agent
- Route one Discord bot token → "coding" agent, another → "main"

**Setting it up:**
```bash
openclaw agents add work
openclaw agents add coding
openclaw agents list --bindings
```
Then configure bindings in `openclaw.json` to wire channels/accounts/peers to agents.

**Use cases:**
- **Multiple people** sharing one server (each gets their own agent)
- **Multiple personalities** — a casual daily agent on Sonnet and a deep-work agent on Opus
- **Security isolation** — per-agent sandboxing, tool allow/deny lists
- **One WhatsApp number, multiple people** — route different DMs to different agents

**Per-agent config options:**
- Custom model per agent
- Sandbox mode (off, all, etc.)
- Tool restrictions (allow/deny lists)
- Mention patterns for group chats

By default you're running single-agent mode (agent id "main"). Everything just works with one brain. Multi-agent is opt-in when you need isolation or routing.

Want me to dig into any specific part — like setting up a second agent, or the routing/binding rules?
assistant: Reasoning:
_Trevor is asking about "agents mode" - this likely refers to something in OpenClaw. Let me check the OpenClaw docs for information about agents mode._
assistant: Reasoning:
_That's a comprehensive doc on multi-agent mode. Let me give Trevor a clear, digestible summary._
user: How does agents mode work?
assistant: Here's how agents mode works in OpenClaw:

**The core idea:** you can run multiple isolated AI "brains" in a single Gateway process, each with its own personality, workspace, auth, and session history.

**What makes an agent:**
- Its own **workspace** (SOUL.md, AGENTS.md, USER.md — separate persona and files)
- Its own **state directory** (auth profiles, model config)
- Its own **session store** (chat history doesn't cross between agents)

**Routing — how messages find the right agent:**
You define **bindings** that map inbound messages to agents based on channel, account, peer (specific DM/group), guild, etc. Most-specific match wins. Examples:
- Route all WhatsApp → "chat" agent (Sonnet), all Telegram → "opus" agent (Opus)
- Route a specific WhatsApp group → "family" agent
- Route one Discord bot token → "coding" agent, another → "main"

**Setting it up:**
```bash
openclaw agents add work
openclaw agents add coding
openclaw agents list --bindings
```
Then configure bindings in `openclaw.json` to wire channels/accounts/peers to agents.

**Use cases:**
- **Multiple people** sharing one server (each gets their own agent)
- **Multiple personalities** — a casual daily agent on Sonnet and a deep-work agent on Opus
- **Security isolation** — per-agent sandboxing, tool allow/deny lists
- **One WhatsApp number, multiple people** — route different DMs to different agents

**Per-agent config options:**
- Custom model per agent
- Sandbox mode (off, all, etc.)
- Tool restrictions (allow/deny lists)
- Mention patterns for group chats

By default you're running single-agent mode (agent id "main"). Everything just works with one brain. Multi-agent is opt-in when you need isolation or routing.

Want me to dig into any specific part — like setting up a second agent, or the routing/binding rules?
assistant: I just covered this — did my previous message not come through? Here's the short version:

**Agents mode** lets you run multiple isolated AI brains in one OpenClaw Gateway. Each agent gets its own workspace, persona, auth, sessions, and model. You wire inbound messages to agents using **bindings** — rules that match on channel, account, peer, guild, etc.

```bash
openclaw agents add work       # create a new agent
openclaw agents list --bindings # see what's routed where
```

By default you're in single-agent mode (agent id `main`). Multi-agent is opt-in.

Want me to walk through setting up a specific scenario?
user: Is there a way to opt-in via a command when I query you?
assistant: Not really — agents mode is a **config-level setup**, not a per-query toggle. You enable it by:

1. **CLI:** `openclaw agents add <name>` to create agents
2. **Config:** Add bindings in `openclaw.json` to route messages
3. **Restart:** `openclaw gateway restart`

There's no in-chat `/agent switch coding` type command. Routing is determined by bindings before your message even reaches a model — it's based on which channel, account, or peer you're messaging from.

**What you *can* do in-chat:**
- `/model <name>` — switch models on the fly (e.g. `/model sonnet`)
- `/new` — reset the session
- Sub-agents can be spawned within a session to delegate work

**If you want to talk to a different agent**, the intended pattern is to message from a different channel/account/thread that's bound to that agent. For example, WhatsApp → everyday agent, Discord → deep work agent.

From the **CLI** you can also directly target an agent:
```bash
openclaw agent --agent coding --message "Summarize logs"
```

But there's no way to say "hey, hand this to my other agent" mid-conversation in chat. Want me to help set up a second agent with bindings?
user: Hmm what if I were to create a new discord channel and add all 4 of my models to it like I have for the OpenClaw channel?
assistant: That actually works really well. Here's the idea:

**One Discord server/channel, multiple bots — each bot = one agent with its own model.**

You'd set it up like:
- 4 Discord bot accounts (one per model/agent)
- All 4 bots invited to the same channel
- Each bot bound to a different agent in your config
- `requireMention: true` on the channel so they only respond when @mentioned

Then in chat you just @ the one you want:
- `@OpusClaude deep analysis of this...`
- `@SonnetClaude quick question...`
- `@GPTBot what do you think?`

The config would look something like:
```json5
{
  agents: {
    list: [
      { id: "opus", workspace: "~/.openclaw/workspace-opus", model: "anthropic/claude-opus-4-6" },
      { id: "sonnet", workspace: "~/.openclaw/workspace-sonnet", model: "anthropic/claude-sonnet-4-6" },
      // ... etc
    ]
  },
  bindings: [
    { agentId: "opus", match: { channel: "discord", accountId: "opus" } },
    { agentId: "sonnet", match: { channel: "discord", accountId: "sonnet" } },
    // ...
  ],
  channels: {
    discord: {
      accounts: {
        opus: { token: "BOT_TOKEN_1", guilds: { ... } },
        sonnet: { token: "BOT_TOKEN_2", guilds: { ... } },
      }
    }
  }
}
```

Each agent gets its own workspace, personality, and memory — totally isolated brains that happen to share a room.

What 4 models are you thinking? And do you already have the extra Discord bot tokens, or do you need to create them?
