# Local Markdown Wayfinder Tracker

No repository issue tracker existed when this map was charted. Issues therefore live as Markdown files in `planning/wayfinder/issues/`.

## Operations

- Identity: `id` in YAML frontmatter.
- Parent/child: ticket `parent` names map issue ID.
- Labels: `labels` list.
- Claim: set `assignee` before work. Open issue with `assignee: null` is unclaimed.
- Blocking: `blocked_by` lists issue IDs. Issue is unblocked when every listed issue has `status: closed`.
- Frontier: open, unassigned child issues whose blockers are all closed, ordered by ID.
- Resolution: write answer in `planning/wayfinder/resolutions/<id>.md`, set ticket `status: closed`, and link resolution from ticket `resolution` field.
- Map indexing: append one linked gist per closed route ticket under `Decisions so far`. Do not list open tickets in map body.
- Out of scope: close mis-scoped ticket, record reason under map `Out of scope`, and omit it from `Decisions so far`.

Research branches may carry notes under `planning/research/`. Merge completed notes and resolution metadata before advancing dependent tickets.
