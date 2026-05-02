# Design Decisions — claude-github-sync

## Don't name your env var `GITHUB_TOKEN`

`GITHUB_TOKEN` is a reserved env var that the `gh` CLI reads as its authentication credentials. Setting it in Claude Code's `settings.json` env section silently hijacked all local `gh` auth with the read-only PAT — breaking local pushes without any error message. Renamed to `CLAUDE_CONFIG_TOKEN` to avoid the conflict.

**Lesson:** Check for reserved env var names before naming anything `*_TOKEN` in a tool that has its own auth system.

---

## rsync over git clone for bootstrap

The bootstrap script uses `rsync -a --exclude='.git'` to overlay repo contents into `~/.claude/` rather than a clean clone. This is intentional: `settings.json` is gitignored on the repo side but must survive the bootstrap on the local machine side (it contains the PATs needed for subsequent runs). A clean clone would wipe it.

`rsync` overlays without deleting untracked local files. New files from the repo land; local-only files survive.

---

## Blacklist over whitelist for gitignore

The `~/.claude/` repo started with a whitelist approach (`*` then `!exceptions`) since that's the safer default for repos where sensitive content could appear unexpectedly. But as the skills directory grew, the whitelist needed updating for every new skill added — friction that would eventually cause misses.

Switched to blacklist: include everything, explicitly exclude the ephemeral runtime dirs. The tradeoff is that an unexpected new sensitive directory could slip in, but the specific directories to exclude are well-understood and stable.

The vault uses blacklist from the start for the same reason.

---

## Commit before running a self-rsyncing script

The bootstrap script rsyncs the repo over `~/.claude/`. If you edit the script locally and run it before committing, the rsync pulls the old version from the repo and overwrites your edits. Always commit first.

This is the kind of footgun that looks obvious in retrospect and isn't at all obvious in the moment.

---

## `.claude/` dirs appear inside project folders

Claude Code creates a `.claude/settings.json` in any directory it works in. If your vault or any project directory is in scope, these project-level Claude dirs will appear and nearly slip into the vault commit. Added `**/.claude/` to the vault gitignore after one nearly did.

---

## Two separate repos, not one

Config and vault are kept in separate repos for access control granularity: a cloud session can bootstrap config-only (read-only PAT for `kiron-claude-config`) without needing vault access. The vault token is optional — set it only when the session needs career context or portfolio history.
