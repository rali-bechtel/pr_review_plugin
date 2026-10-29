---
name: pr-review-setup
description: Set up or reconfigure the pr-review plugin for this project — checks that gh is installed and authenticated, asks for the target repo (an existing local clone path, or a repo URL/owner-repo to clone), asks for a coding-standards path (or the plugin's bundled defaults), and writes repo.config.json. Use when the user says "set up pr review", "configure pr-review", "/pr-review-setup", or when /pr-review can't find a usable repo.config.json.
---

# PR Review Setup

One-time (or re-run anytime) configuration step for this plugin. Produces `repo.config.json` in the project root that `/pr-review` and the `pr-reviewer` agent both read.

## 1. Check prerequisites first — stop if they fail

- Run `gh --version`. If it's missing, tell the user to install the GitHub CLI (https://cli.github.com/) and stop.
- Run `gh auth status`. If it reports not logged in, tell the user to run `gh auth login` (or set `GH_TOKEN`/`GITHUB_PAT` for this plugin's convention) and stop — do not continue setup until auth succeeds.
- Run `git --version`. If missing, tell the user git is required (for cloning/diffing the target repo) and stop.

Only proceed past this section once all three succeed.

## 2. Determine the target repo

Ask with `AskUserQuestion` (single-select):

> "How should I get the project code to review?"
- "Already cloned locally" — I'll give a path to an existing clone.
- "Clone a repo for me" — give a repo URL or `owner/repo` and I'll clone it.

Then, in a normal follow-up message (free text, not AskUserQuestion — paths/URLs aren't a fixed option set), ask for the actual value:

- **Already cloned locally**: ask for the local path. Verify it exists and is a git repo with `git -C <path> rev-parse --is-inside-work-tree`. If that fails, say so and ask again. Get `repo_url` for the config with `git -C <path> remote get-url origin` (ok if it fails — leave `repo_url` out).
- **Clone a repo for me**: ask for a repo URL (`https://github.com/owner/repo.git`) or `owner/repo`. Derive `local_path` as `project_code/<repo-name>` (repo name = last path segment, `.git` stripped). If that path already exists and is a git repo, reuse it instead of re-cloning. Otherwise clone:
  - Full URL: `git clone <repo_url> <local_path>`
  - `owner/repo` shorthand: `gh repo clone <owner/repo> <local_path>`

## 3. Determine coding standards

If `${user_config.coding_standards_dir}` is set (the user saved a personal default via `/plugin configure`), offer it as an option. Ask with `AskUserQuestion` (single-select):

> "Where are your coding standards?"
- "Use my saved default (`${user_config.coding_standards_dir}`)" — only shown if that's set.
- "Use this plugin's built-in TypeScript/React standards" (bundled at `${CLAUDE_PLUGIN_ROOT}/docs/coding-standards/`)
- "Use a different path for this project" — give a path to a file or folder in or near this project.

If the user picks a different path, ask (free text) for it, confirm it exists with `Read`/`Glob`, and store that as `coding_standards_path`. If they pick the saved default, store `${user_config.coding_standards_dir}`'s actual path as `coding_standards_path` (so the project's config is self-contained even if the personal default later changes). If they pick the built-in option, omit `coding_standards_path` from the config (the agent falls back to the bundled defaults automatically). Mention `/plugin configure pr-review@pr-review-tools` as the way to set or change the saved personal default.

## 4. Write repo.config.json

Write (or overwrite) `repo.config.json` in the project root:

```json
{
  "repo_url": "<string, omit if unknown>",
  "local_path": "<string>",
  "coding_standards_path": "<string, omit to use bundled defaults>"
}
```

## 5. Confirm

Summarize what was written (repo, local path, coding standards source) and tell the user `/pr-review` is ready to use.
