# c4c-ship

Code for Creatives skill that pushes a student's site to GitHub then deploys to Cloudflare Pages. Auto-detects whether it's the first ship (sets up the repo + project) or a subsequent ship (one-command update). Falls back to the /c4c-setup steps inline if anything is missing.

## Install

```
mkdir -p ~/.claude/skills && git clone https://github.com/alexdobrenko/c4c-ship-skill ~/.claude/skills/c4c-ship
```

## Use

In Claude Code, `cd` to the folder with your site (the one with `index.html`), then type:

```
/c4c-ship
```

The skill verifies your setup, then either:

- **First time**: walks through `git init`, GitHub repo create, Cloudflare project create, deploy. Saves the project name so the next ship is friction-free.
- **Subsequent ship**: asks what changed, commits, pushes to GitHub, redeploys to Cloudflare. Usually under a minute.

## Update

```
cd ~/.claude/skills/c4c-ship && git pull
```

## Requires

- macOS
- Run `/c4c-setup` first if you haven't (Homebrew, GitHub CLI, Node, Wrangler, GitHub + Cloudflare account auth). The skill will catch a missing setup and either walk you through fixing the specific gap or tell you to run `/c4c-setup` if you're starting from nothing.
