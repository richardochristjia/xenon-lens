---
id: XL-009
title: Define feedback delivery and recovery semantics
status: open
labels:
  - wayfinder:grilling
parent: XL-001
assignee: null
blocked_by:
  - XL-005
  - XL-008
resolution: null
---

## Question

What state machine and idempotency rules distinguish storing, submitting, delivering, acknowledging, processing, responding, and accepting while recovering safely from busy agents, exits, timeouts, disconnects, duplicates, stale revisions, missing targets, partial responses, and multiple sessions or artifacts?
