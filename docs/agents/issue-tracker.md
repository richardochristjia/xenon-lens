# Issue tracker: GitHub

Issues and specs for this repo live as GitHub issues. Use the `gh` CLI for all operations.

Existing files under `planning/wayfinder/` are historical planning records. Create new work in GitHub Issues.

## Conventions

- Create: `gh issue create --title "..." --body "..."`
- Read: `gh issue view <number> --comments`
- List: `gh issue list --state open`
- Comment: `gh issue comment <number> --body "..."`
- Label: `gh issue edit <number> --add-label "..."`
- Close: `gh issue close <number> --comment "..."`
- Infer repository from `git remote -v`.

## Pull requests as a triage surface

**PRs as a request surface: no.**

## Skill operations

- “Publish to issue tracker”: create a GitHub issue.
- “Fetch relevant ticket”: run `gh issue view <number> --comments`.
- Wayfinder maps and children use GitHub Issues, native sub-issues, dependencies, assignments, comments, and labels.
