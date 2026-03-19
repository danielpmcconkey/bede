# Bede — Setup Checklist

> Skill files are built. These are the remaining steps to make Bede operational.

## 1. Discord Bot Creation (Dan — browser required)

**Follow this order exactly. Do not deviate.** (See discord-bot-private-problem.md for why.)

1. Discord Developer Portal → New Application → name "Bede" → Create
2. **IMMEDIATELY** go to Installation (left sidebar)
3. Set Install Link to **None**
4. Save
5. Go to Bot → toggle Public Bot **OFF**
6. Toggle Message Content Intent **ON**
7. Save
8. OAuth2 → URL Generator → scopes: `bot` → permissions: Send Messages, Read Message History, View Channels
9. Copy generated URL → open → invite to `dan-home` server
10. Copy the bot token — you'll need it for step 3

## 2. Create #archives Channel (Dan — Discord)

- Create `#archives` channel in `dan-home`
- Deny `@everyone` Send Messages
- Allow only Dan explicitly
- Note the channel ID (right-click → Copy Channel ID with Developer Mode on)

## 3. openclaw.json Config (Dan + agent)

Edit `~/.openclaw/openclaw.json`:

**Add to `agents.list`:**
```json
{
  "id": "bede",
  "name": "bede",
  "workspace": "/media/dan/fdrive/codeprojects/bede/workspace",
  "agentDir": "/home/dan/.openclaw/agents/bede/agent",
  "model": "anthropic/claude-opus-4-6",
  "identity": {
    "name": "Bede",
    "theme": "AI agent / transcript historian",
    "emoji": "📜"
  }
}
```

**Add to `bindings`:**
```json
{
  "agentId": "bede",
  "match": { "channel": "discord", "accountId": "bede" }
}
```

**Add to `channels.discord.accounts`:**
```json
"bede": {
  "token": "<PASTE BOT TOKEN HERE>",
  "groupPolicy": "allowlist",
  "guilds": {
    "<GUILD_ID>": {
      "channels": {
        "<ARCHIVES_CHANNEL_ID>": { "allow": true }
      }
    }
  }
}
```

Guild ID is the same as Zazu/Flintheart: `1476537649803034634`

## 4. Exec Approvals

Add Bede's scripts to exec-approvals.json (wherever that lives — check Zazu's pattern):

```json
{
  "agent": "bede",
  "command": "discover.py",
  "path": "/media/dan/fdrive/codeprojects/bede/workspace/skills/summarize-transcripts/scripts/discover.py"
},
{
  "agent": "bede",
  "command": "update_registry.py",
  "path": "/media/dan/fdrive/codeprojects/bede/workspace/skills/summarize-transcripts/scripts/update_registry.py"
}
```

## 5. Cron Setup

After gateway restart:
```bash
openclaw cron add --name "bede:summarize-transcripts" --cron "0 21 * * *" --agent bede --skill summarize-transcripts
```

## 6. Verify

```bash
openclaw gateway restart
openclaw channels status --probe
```

Test end-to-end in #archives channel.
