# Salable Codex Skill

Codex skill distribution repo for `salable-monetization`.

## Prerequisites

You'll need a Salable API key, use your secret key:

```bash
export SALABLE_API_KEY="your_api_key_here"
```

## Setting Up MCP for Codex

Add the Salable MCP server with a single command:

```bash
codex mcp add salable \
  --url https://beta.salable.app/api/mcp \
  --bearer-token-env-var SALABLE_API_KEY
```

## Install from GitHub (End Users)

In Codex, paste:

```
Use the `$skill-installer` to install 
- Repo: `Salable/salable-codex-skills`
- Path: `skills/salable-monetization`
```

## Manual Install

```bash
git clone https://github.com/Salable/salable-codex-skills.git
cd salable-codex-skills
mkdir -p ~/.agents/skills
cp -R skills/salable-monetization ~/.agents/skills/
```

Codex usually detects skill changes automatically. If the skill does not appear, restart Codex.
