# claude-github-sync

Two private GitHub repos and a bootstrap script that give any Claude Code cloud session — on any device — the same skills, context, and memory as a local session. Run one command and Claude knows who you are, what you've built, and what you're working toward.

## What this unlocks

Every cloud session used to start cold. No skills, no history, no context. Claude was capable but generic — a stranger every time.

Now a 30-second bootstrap gives any session full access to:

- **All custom skills and workflows** — every command, agent behavior, hook, and automation built up over time
- **Everything that's been built** — project decisions, architecture choices, build logs, lessons learned
- **Projects that exist only as ideas** — anything captured in the vault can be picked up and executed on the spot, not just from the Mac
- **Active context** — open loops, current focus, what's in flight and what's paused
- **Career and professional context** — target companies, key contacts, positioning priorities, what matters right now

The practical result: a cloud session on mobile or any remote device has the same starting point as sitting at the Mac. An idea captured last week can be executed this afternoon. A project that only lived in notes now has everything it needs to become real — from anywhere.

## The problem

Custom Claude Code skills, hooks, and `CLAUDE.md` live only on the local machine. Opening a cloud session at `claude.ai/code` means starting cold: no custom skills, no persistent config, no memory of anything.

The knowledge vault — career priorities, project history, professional contacts, open ideas — was also local-only. Any session that needed strategic context had to reconstruct it from scratch every time.

## How it works

Two private GitHub repos:

- **`kiron-claude-config`** — backs up `~/.claude/`: skills, hooks, agents, CLAUDE.md, and the bootstrap script itself
- **`kiron-vault`** — backs up the full Obsidian knowledge vault (excluding daily notes, health data, and trash)

A bootstrap script (`bootstrap-cloud-session.sh`) runs in any fresh cloud session:

1. Shallow-clones `kiron-claude-config` to a temp dir
2. `rsync`s it into `~/.claude/` — preserving untracked local files like `settings.json`
3. Optionally shallow-clones the vault to `~/Kiron_Vault/` if the vault token is set

```bash
export CLAUDE_CONFIG_TOKEN=<your-read-only-PAT>
bash <(curl -s https://raw.githubusercontent.com/kironiyer-source/kiron-claude-config/main/bootstrap-cloud-session.sh)
```

Config-only bootstrap (skills, hooks, CLAUDE.md) takes about 10 seconds. Adding the vault gives Claude the full picture — who you are, what you've built, what you're working toward — and takes around 30 seconds total.

## Auth design

Two read-only fine-grained PATs — one per repo — stored in `settings.json` under the `env` key. `settings.json` is gitignored on both repos, so tokens never leave the local machine. Cloud sessions read the env vars at bootstrap time. Local pushes use the `gh` CLI keyring OAuth token.

The vault token is optional. A session only needs it when personal or strategic context matters — career decisions, picking up an ideated project, anything that requires knowing the full picture.

## gitignore strategy

Both repos use **blacklists** (exclude specific folders) rather than whitelists (include only named folders). This is the right call when you're including most content and only blocking specific directories — whitelist would require updating the gitignore every time a new skill or folder is added.

`~/.claude/` excludes ~40MB of runtime/ephemeral data: `projects/`, `plugins/`, `cache/`, `telemetry/`, `sessions/`, `settings.json`, `history.jsonl`.

The vault excludes private folders, `.obsidian/` (machine-specific app config), `**/.claude/` (Claude Code creates project dirs inside any directory it works in), and OS artifacts.

## Key design decisions

See [DESIGN.md](./DESIGN.md) for the full decision log.
