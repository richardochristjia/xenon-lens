# Xenon Lens

Xenon Lens is a local collaboration environment where a person and an originating coding agent iterate through interactive HTML artifacts without coupling the collaboration model to one agent harness.

## Language

**Collaboration Session**:
An isolated conversation loop between one person and one originating agent consumer. A session may contain multiple artifacts and remains distinct from any artifact it contains.
_Avoid_: Browser session, agent run, workspace

**Originating Agent**:
The single agent consumer entitled to receive submitted feedback for a collaboration session. Ownership may move only through explicit handoff.
_Avoid_: Active model, backend

**Lens-native Artifact**:
Agent-authored HTML that follows the Xenon Lens collaboration contract, including stable targets and optional explicit state hooks.
_Avoid_: Web page, prototype

**Artifact Revision**:
A specific published version of one artifact within a collaboration session.
_Avoid_: Reload, session version

**Target**:
A stable, addressable part of an artifact to which feedback can refer.
_Avoid_: Selector, DOM node

**Feedback Item**:
One human comment, question, or structured answer with its own identity and optional target context. It remains distinct when submitted with other items.
_Avoid_: Prompt, message

**Feedback Batch**:
An explicit submission containing one or more feedback items for the originating agent.
_Avoid_: Summary, prompt

**Agent Response**:
An originating agent's answer or artifact change associated with submitted feedback. It does not imply human acceptance.
_Avoid_: Approval, resolution

**Acceptance**:
An explicit human judgment that a proposal, answer, or change is accepted. Submission and agent response are not acceptance.
_Avoid_: Send, acknowledgement
