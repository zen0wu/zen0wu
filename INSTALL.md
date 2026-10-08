# Use this with Codex

Keep this checkout as the source. Link to it; don't copy individual instruction files.

## One-time setup

Run from this repository:

```sh
zeno_repo="$(pwd -P)"
mkdir -p "$HOME/.codex" "$HOME/.agents/skills"
ln -s "$zeno_repo/AGENTS.md" "$HOME/.codex/AGENTS.md"
ln -s "$zeno_repo/skills" "$HOME/.agents/skills/zeno"
```

If `~/.codex/AGENTS.md` already points here, leave it alone and skip that `ln` command. If either destination exists with another target or contains a regular file or directory, inspect it before making changes; these commands deliberately do not overwrite it.

The first link supplies the always-on guidance. The second exposes the skills under this checkout, including future additions. It does not load all their bodies into each prompt. Neither link depends on the names of files inside a skill.

Codex supports [user skills and symlinked skill folders](https://learn.chatgpt.com/docs/build-skills#where-codex-loads-local-skills). Newly installed skills should be available on the next turn; restart Codex if they do not appear. Check the skill picker for `coding` and `architecture`.

Editing a rule or reference changes the source immediately; it does not replace copies already present in an ongoing chat. Ask for a fresh read when testing an update, or start a new chat to check the complete setup. If the checkout itself moves, update the two links.

## What lives where

- `README.md`: the human introduction, linking directly to the skill guides.
- `AGENTS.md`: the compact agent constitution and task routing.
- `skills/*/SKILL.md`: the operational rules for that kind of work.
- `skills/*/references/`: examples to consult when relevant.

Maintain detailed operational rules and examples in the skills, shared by human readers and agents. Other agents can follow the relative skill links in `AGENTS.md` without native skill discovery, provided this checkout is accessible.
