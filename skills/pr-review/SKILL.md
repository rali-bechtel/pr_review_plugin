---
name: pr-review
description: Review open GitHub pull requests for bugs, security issues, dependency changes, and coding standards, with severity ratings. Lists open PRs, lets the user pick one at a time, saves findings + suggested fixes to JSON, and asks the user which findings (if any) to post as PR comments. Use when the user says "review PRs", "review this PR", "/pr-review", or asks to check open pull requests for a repo.
---

# PR Review

Use the `pr-reviewer` agent to do the read/analyze/record work for one PR. This skill handles repo/PR selection, presenting findings, and posting comments — the agent never posts on its own.

## Steps

0. **Check setup first.** If `repo.config.json` doesn't exist in the project root, or `gh auth status` fails, run the `pr-review-setup` skill instead and stop — don't try to guess the repo or proceed unauthenticated.
1. **Determine the repo.** If `args` names one (`owner/repo`), use it. Otherwise read `repo.config.json` in the project root (`repo_url` field, e.g. `https://github.com/<owner>/<repo>.git`) and parse owner/repo from it. If it also has a `local_path` field, a local clone already exists there — pass it to the agent so it reads files from disk instead of the GitHub API. If it has a `coding_standards_path` field, pass that to the agent too (otherwise the agent uses its bundled defaults).
2. **List open PRs** with `GH_TOKEN="$GITHUB_PAT" gh pr list --repo <owner/repo> --state open --json number,title,headRefName,url` and show them to the user as a table.
   - If `args` already names a specific PR number/URL, skip straight to step 3 with that PR — no need to ask.
3. **Ask the user which PR to review**, one at a time (AskUserQuestion, single-select, options = the open PRs). Do not fan out across multiple PRs in parallel — process exactly one PR per review pass.
4. **Invoke the `pr-reviewer` agent** once for the chosen PR, passing repo, PR number, `local_path` (if any), and `coding_standards_path` (if any). The agent carries forward `posted`/`comment_url` state from any prior `pr_review/pr-<number>-*.md` report for this PR, then writes the refreshed findings + suggested code to `pr_review/pr-<number>-<today>.md` — it does not post anything.
5. **Present the findings** back to the user: group by severity (high → medium → low), each with category, file:line, description, suggested fix, and its status — `✅ posted (<comment_url>)` or `◻ not posted`.
   - If there are zero findings, say so and stop here.
   - If every finding is already posted, say so and skip straight to step 8.
6. **Always ask the user interactively which findings to post** — this step is never skipped and never replaced by a plain-text question. Use the `AskUserQuestion` tool (multi-select), options = the **not-yet-posted** findings (don't offer already-posted ones, nothing to do for them) plus an explicit **"Write my own comment"** option. If the user picks "Write my own comment" (or types their own text via the tool's built-in "Other" input), that's a custom comment, not one of the agent's findings.
7. **For each selected finding, dedupe then post:**
   - First fetch existing comments: `GH_TOKEN="$GITHUB_PAT" gh api repos/<owner>/<repo>/pulls/<number>/comments --jq '.[] | {path, line, body}'`. If one already matches this finding's `path`+`line` and its body already contains this finding's `rule_id` (or category+description), skip posting — it's already there (e.g. posted by a prior run before the local JSON was updated) — just backfill `posted: true` and its `html_url` into the JSON.
   - Otherwise post it: `GH_TOKEN="$GITHUB_PAT" gh api repos/<owner>/<repo>/pulls/<number>/comments -f body="[category(severity)] <description>\n\nSuggested fix:\n<suggested_code>" -f commit_id="<head_sha>" -f path="<path>" -F line=<line> -f side=RIGHT`. Never approve or request changes — comments only.
   - After each post (or dedupe-skip), update that finding's `posted: true` and `comment_url` in the `pr_review/pr-<number>-<today>.md` JSON with `Edit`, so the state is durable for next time and nothing gets double-posted on a future run.
   - **For a custom comment**, ask the user (plain text is fine here, it's free-form input, not a findings list) which file:line it belongs to, or whether it's a general PR-level comment. Post a file:line comment the same way as above (skip the dedupe check — it's user-authored, not a tracked finding), or a general comment with `GH_TOKEN="$GITHUB_PAT" gh pr comment <number> --repo <owner>/<repo> --body "<text>"`. Append it to the JSON's `findings` array as `{"id": "user-<n>", "category": "user-comment", "severity": null, "rule_id": null, "file": "<path or null>", "line": <number or null>, "description": "<the user's text>", "suggested_code": null, "posted": true, "comment_url": "<html_url>"}` with `Edit`, so it's tracked like everything else.
8. **Ask again if the user wants another PR reviewed** before ending — loop back to step 3 if so.

Keep this lightweight: don't re-fetch diffs or re-analyze here, that's the agent's job. The skill only lists, asks, presents, and posts.
