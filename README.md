<img src="assets/hero.png" alt="chipped-mug — Understand the change. A terracotta mug with a chipped rim." width="960">

An Agent Skill that explains what a code change actually does, in plain English. It uses the repository as context, for teammates who know the product but may never have opened the codebase.

Think of the senior developer with the chipped company mug: “Okay. Here’s what actually changed.”

## A change, explained

After installation, ask your coding agent with the repository open:

```text
Use chipped-mug to explain main...selected-change.
```

Here is a synthetic saved-article example: four changed files become this explanation. This is an unedited historical v0.5 response, not a new v0.7 result. The [complete source and patch](examples/reading-room.md) are included.

> This adds pinning to the saved-article library. Pinned articles now appear at the top and carry a “Pinned:” label. Each member has their own pins.
>
> Previously, articles appeared in the order they were saved. That order is still kept within the pinned and unpinned groups. Unpinning an article leaves it saved and puts it back among the unpinned articles.

Give them the part the developer usually explains.

## Not another PR reviewer

It explains what the change does. It does not judge the code, look for bugs, design tests, or compare the implementation with a ticket. Implemented conditions and fallback behavior still belong in the explanation. It does not modify the repository.

## Install and use

You need a coding agent with Git and read/search access to the repository, including the compared revisions. A PR link also needs access to its base and head. There is no separate CLI or runtime.

Clone this repository:

```sh
git clone https://github.com/janmrow/chipped-mug.git
```

Copy its `SKILL.md` into one of these locations, creating the `chipped-mug` directory if needed:

| Local agent | Personal installation |
| --- | --- |
| [Codex CLI / IDE](https://learn.chatgpt.com/docs/build-skills) | `~/.agents/skills/chipped-mug/SKILL.md` |
| [Claude Code](https://code.claude.com/docs/en/skills) | `~/.claude/skills/chipped-mug/SKILL.md` |
| [OpenCode](https://opencode.ai/docs/skills/) | `~/.config/opencode/skills/chipped-mug/SKILL.md` |

For example, to install for Codex from the directory where you cloned this repository:

```sh
mkdir -p ~/.agents/skills/chipped-mug
cp chipped-mug/SKILL.md ~/.agents/skills/chipped-mug/SKILL.md
```

Start a new session in the repository you want explained. In Codex, select the skill with `$chipped-mug` or `/skills`; in Claude Code, use `/chipped-mug`. In OpenCode, ask the agent to load `chipped-mug` with its skill tool and explain the selected change.

Point it at a PR, branch, commit, revision range, or diff. For a branch, naming the base makes the selection clear:

```text
Use chipped-mug to explain the feature branch against main.
```

The diff defines the scope. Without repository access or the needed revisions, the agent can only give a limited explanation or ask for missing context.

These are local installation paths; remote and managed environments may use different routes.

## How it works

Diff is the anchor. The repo is the context. Understanding is the output.

The agent fixes the before and after revisions, reads the selected change, follows the repository context needed to understand it, and explains the meaningful threads in product and system language. Exact technical names appear only when their identity matters.

It may read twenty files. You shouldn’t have to.

The instructions live in [SKILL.md](SKILL.md).

Try it on the PR you were about to ask someone to explain.

## License

[MIT](LICENSE). The bundled Libre Baskerville font is licensed under the [SIL Open Font License 1.1](source/OFL.txt). Editable artwork and export notes are in [source/](source/README.md).
