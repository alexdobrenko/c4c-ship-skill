# c4c-ship

A Code for Creatives skill. It puts your website online with Cloudflare and saves its files to GitHub. Works on Mac and Windows.

## Install

Ask Claude Code to run this for you, or paste it into your terminal:

```
mkdir -p ~/.claude/skills && git clone https://github.com/alexdobrenko/c4c-ship-skill ~/.claude/skills/c4c-ship
```

Then close Claude Code and open it again, so it sees the new skill.

## Use

Open Claude Code in the folder that has your site (the one with `index.html`) and say:

```
put my site online
```

The first time, Claude:

1. Checks your setup.
2. Shows you exactly which files will be public, and asks before anything goes online.
3. Asks for a name and puts the site online at an address that ends in `workers.dev`.
4. Saves the folder to a private GitHub repo.

After that, say "update my site" whenever you change something. It takes under a minute.

## Update the skill

```
cd ~/.claude/skills/c4c-ship && git pull
```

## Before you use it

Do the Week 3 setup first: a GitHub account, a Cloudflare account, the three tools, and both sign-ins. If something is missing, the skill tells you which part.
