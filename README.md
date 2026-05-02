# claude-github-sync

A two-repo system that backs up Claude Code config and an Obsidian knowledge vault to private GitHub repos, enabling any Claude Code cloud session — on any device — to bootstrap with custom skills, hooks, and personal context via a single command.

## The problem

Custom Claude Code skills, hooks, and `CLAUDE.md` live only on the local machine. Opening a cloud session at `claude.ai/code` or on mobile means starting cold: no custom skills, no personal context, no persistent config. Every cloud session is a blank slate.

## How it works

Two private GitHub repos:

- **`kiron-claude-config`** — backs up `~/.claude/`: skills, hooks, agents, CLAUDE.md, and the bootstrap script itself
- **`kiron-vault`** — backs up selected Obsidian vault folders (everything except daily notes, health data, and trash)

A bootstrap script (`bootstrap-cloud-session.sh`) runs in any fresh cloud session:

1. Shallow-clones `kiron-claude-config` to a temp dir
2. `rsync`s it into `~/.claude/` — preserving untracked local files like `settings.json`
3. Optionally shallow-clones the vault to `~/Kiron_Vault/` if the vault token is set

```bash
export CLAUDE_CONFIG_TOKEN=<your-read-only-PAT>
bash <(curl -s https://raw.githubusercontent.com/kironiyer-source/kiron-claude-config/main/bootstrap-cloud-session.sh)
```

After bootstrap, the cloud session has full access to all custom skills, hooks, and vault context.

## Auth design

Two read-only fine-grained PATs — one per repo — stored in `settings.json` under the `env` key. `settings.json` is gitignored on both repos, so tokens never leave the local machine. Cloud sessions read the env vars at bootstrap time. Local pushes use the `gh` CLI keyring OAuth token.

## gitignore strategy

Both repos use **blacklists** (exclude specific folders) rather than whitelists (include only named folders). This is the right call when you're including most content and only blocking specific directories — whitelist would require updating the gitignore every time a new skill is added.

`~/.claude/` excludes ~40MB of runtime/ephemeral data: `projects/`, `plugins/`, `cache/`, `telemetry/`, `sessions/`, `settings.json`, `history.jsonl`.

The vault excludes private folders, `.obsidian/` (machine-specific app config), `**/.claude/` (Claude Code creates project dirs inside any directory it works in), and OS artifacts.

## Key design decisions

See [DESIGN.md](./DESIGN.md) for the full decision log.
