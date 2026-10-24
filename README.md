# qa-kit

A small Claude Code plugin for the end of a piece of work: one command that summarises what changed on your branch, and one subagent that reviews those changes for bugs.

## What it adds

| Piece | Type | What it does |
| --- | --- | --- |
| `/qa-kit:summarize-changes [base]` | Slash command | Lists every file touched on the current branch with a one-line description of the change, short enough to paste into a pull-request description. Compares against `main` unless you pass another base branch. |
| `code-reviewer` | Subagent | Reviews recent changes for bugs, missing error handling and unclear names, and returns a short list grouped by severity (high, medium, low), each item naming the file and the fix. It can read files (`Read`, `Grep`, `Glob`) and run `Bash`, which its instructions limit to read-only git commands (`git status`, `git diff`, `git log`, `git show`) so it can see the diff itself. |

## Structure

```
.
├── .claude-plugin/
│   └── plugin.json      # manifest: name "qa-kit", version 0.1.0
├── commands/
│   └── summarize-changes.md
├── agents/
│   └── code-reviewer.md
└── README.md
```

Only `plugin.json` lives inside `.claude-plugin/`; the component folders sit at the root. The plugin has no bundled scripts, so nothing needs `${CLAUDE_PLUGIN_ROOT}` yet. If a script is added later, reference it as `${CLAUDE_PLUGIN_ROOT}/scripts/...`, never with a hardcoded path.

## How to use it

1. From the root of this repo, start Claude Code with the plugin loaded:
   ```
   claude --plugin-dir .
   ```
2. Summarise your branch:
   ```
   /qa-kit:summarize-changes
   /qa-kit:summarize-changes develop
   ```
3. Get a review by asking in plain words, for example "review my recent changes". Claude hands the job to the `code-reviewer` subagent. You can also name it: "use the code-reviewer agent on my last commit".
4. After editing any plugin file, run `/reload-plugins` to pick up the change without restarting.

## Tested locally

Loaded with `claude --plugin-dir .` (Claude Code 2.1.310) in a copy of the repo with one extra commit to review:

- The plugin loads as `qa-kit` 0.1.0 and registers `/qa-kit:summarize-changes` and the `qa-kit:code-reviewer` agent.
- `/qa-kit:summarize-changes main` listed every changed file with a one-line description.
- "Please review my recent changes." was handed to `qa-kit:code-reviewer` without naming it, and came back grouped by severity.

Two changes to the original `code-reviewer` came out of that test. With only `Read`, `Grep` and `Glob` it could not see the diff (it ended up reading `.git/logs/HEAD`), and Claude kept doing the review itself instead of delegating. Adding `Bash` for read-only git commands and a description that says "use proactively whenever the user asks to review their changes" fixed both.

## Validation

The **Validate plugin** workflow (`.github/scripts/validate-plugin.js`) runs on every push and pull request. It checks that `.claude-plugin/plugin.json` exists, is valid JSON and has a `name`, that no component folder sits inside `.claude-plugin/`, and that at least one component exists. Run it locally with:

```
node .github/scripts/validate-plugin.js
```
