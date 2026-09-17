# Browser isolation and local transport constraints

- **Accessed:** 2026-09-17
- **Target baseline:** current desktop Chrome, Edge, and Firefox
- **Evidence type:** web standards and browser-vendor documentation. Exact Xenon combinations still require cross-browser smoke tests.

## Executive answer

### Standards-backed findings

A script-capable artifact can be displayed in a sandboxed iframe without granting `allow-same-origin`. This forces an opaque origin while still allowing scripts when `allow-scripts` is present. Opaque-origin isolation prevents trusted browser chrome from directly reading or changing artifact DOM, selection, storage, or JavaScript state. Collaboration therefore needs an in-frame SDK/bridge. [HTML sandbox flags](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#attr-iframe-sandbox), [same-origin policy](https://html.spec.whatwg.org/multipage/browsers.html#origin)

`postMessage`/`MessageChannel` provide the standards-based cross-origin bridge. Every inbound message remains untrusted. Parent must validate source window or transferred port, current frame/load, schema, size, identifiers, and state transition. Because sandboxed frame has opaque origin, its serialized origin is not a useful session identity. A load token distinguishes stale frames but cannot make hostile artifact code trustworthy because that code shares frame realm. [Cross-document messaging](https://html.spec.whatwg.org/multipage/web-messaging.html#crossDocumentMessages), [Channel messaging](https://html.spec.whatwg.org/multipage/web-messaging.html#channel-messaging)

Sandbox alone does not remove network access. Server-delivered CSP must constrain scripts/assets/connections/forms/objects/child frames, and sandbox should omit forms, popups, downloads, top navigation, pointer lock, and same-origin unless a later capability is explicitly justified. [CSP Level 3](https://www.w3.org/TR/CSP3/), [HTML sandbox tokens](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#attr-iframe-sandbox)

Browser chrome cannot directly discover exact elements or text selections inside opaque-origin frame. In-frame SDK must perform hit testing, highlighting, selection serialization, structured-control registration, and explicit state export. Parent owns queue, submission control, persistence, delivery display, and acceptance UI.

### Architecture implication

Accepted untrusted-by-default posture is feasible only with split trust:

```text
trusted browser chrome origin
  owns session auth, queue, Send button, statuses, persistence
          ⇅ validated MessagePort/postMessage records
opaque-origin sandboxed artifact frame
  renders HTML, runs artifact + Lens bridge scripts,
  identifies targets/selections, exposes bounded explicit state
```

Artifact can lie about content, target, state, or suggested feedback. Treat all artifact messages as context proposals. Parent must never let frame submit directly, change delivery state, acknowledge agent work, mark Acceptance, invoke filesystem actions, or authorize consequential operations.

## Isolation model

### Iframe sandbox

**Verified by HTML Standard:** sandbox token set controls scripts, origin, forms, navigation, popups, downloads, modals, pointer lock, storage-access escape, and other capabilities. Omitting `allow-same-origin` gives document an opaque origin. Combining `allow-scripts` and `allow-same-origin` for same-origin content can let embedded content remove its sandbox attribute and defeats intended isolation; do not use that combination for Lens artifacts. [Sandbox processing](https://html.spec.whatwg.org/multipage/iframe-embed-object.html#attr-iframe-sandbox)

**Proposed MVP sandbox:** grant only `allow-scripts`. Add no other token until a use case and hostile-artifact test justify it. Standard HTML controls and script-driven UI work under `allow-scripts`; form submission, downloads, popups, nested browsing, and top-level navigation need not.

**Important limit:** sandbox is browser containment, not semantic trust. Artifact JavaScript and injected bridge share one document. Artifact can call, wrap, or imitate exposed bridge functions, alter DOM before context capture, and fabricate messages. Stable Target IDs are authoring references, not attestations.

### CSP

**Verified by CSP3:** response policy can restrict fetch directives (`default-src`, `script-src`, `style-src`, `img-src`, `font-src`, `connect-src`, `frame-src`, `worker-src`), document directives (`base-uri`, `sandbox`), navigation/form behavior, and object embedding. Multiple policies only restrict further. [CSP directives](https://www.w3.org/TR/CSP3/#directives)

**Proposed artifact policy direction:**

- `default-src 'none'`
- permit only required local/inline script mechanism for artifact and bridge;
- local/data image/font/style allowances only as needed;
- `connect-src 'none'` for MVP artifact frame;
- `form-action 'none'`, `object-src 'none'`, `base-uri 'none'`;
- no nested frames/workers unless separately reviewed.

Exact policy depends on artifact packaging. ES modules, external assets, fonts, blob URLs, and inline script/style each change required directives. A strict standalone/locally bundled artifact profile is easier to isolate than arbitrary HTML with CDN dependencies.

**Assumption requiring tests:** classic locally served bridge script loads reliably into opaque-origin sandbox under chosen CSP in all baseline browsers. Module scripts and cross-origin asset fetches may require CORS and should not be assumed.

### Parent-owned UI boundary

Parent chrome must render outside frame:

- Explore/Annotate mode switch;
- editable queue and provenance of each item;
- explicit **Send to agent** action;
- connection, stored, submitted, delivered, acknowledged, processing, response, and acceptance states;
- stale-revision/missing-target warnings;
- end-session and explicit accept/reject controls.

Frame cannot reach parent DOM under same-origin policy. Parent can visually cover iframe, but a generic overlay cannot inspect child elements or selection, and pointer interception prevents underlying artifact interaction. Exact annotation therefore belongs in bridge while authority remains in parent.

## Bridge and message validation

### Standards facts

`Window.postMessage` sends structured-cloneable values across browsing contexts. Receiver gets `source`, `origin`, and data. Standards warn receivers to check sender and message syntax; failure creates cross-site scripting/confused-deputy risk. Transferred `MessagePort` provides a dedicated channel but does not make peer trustworthy. [Web Messaging security](https://html.spec.whatwg.org/multipage/web-messaging.html#security-postmsg), [Structured clone](https://html.spec.whatwg.org/multipage/structured-data.html#safe-passing-of-structured-data)

### Required rules

1. Parent creates iframe and unique `artifactLoadId` for one artifact revision.
2. Parent accepts bootstrap only from exact `iframe.contentWindow`; then transfers fresh `MessagePort`.
3. Every record has protocol version, Collaboration Session ID, Artifact ID, Artifact Revision ID, load ID, message ID, type, and bounded payload.
4. Parent validates JSON-like schema, allowed enum/type, string/array limits, target/state size, and valid lifecycle transition.
5. Parent ignores duplicate message IDs and all messages from retired loads.
6. Bridge never receives runtime session credential usable against agent/persistence APIs.
7. Artifact-originated queue proposals remain visibly editable and cannot auto-submit.
8. Parent sends only narrow commands: set mode, request target context, request explicit state, apply response marker, prepare reload, import explicit state.
9. Errors are data, not HTML; parent escapes artifact-provided labels and text.

A secret exposed inside frame cannot authenticate human intent. It can only correlate that frame instance. Human intent comes from parent-chrome action.

## Element and selected-text targeting

### Element targets

**Lens-native contract:** author supplies unique stable `data-lens-target` values or equivalent registry IDs. Bridge returns ID plus short identifying context (tag/role, accessible label or bounded text, optional structural hint). Generated CSS selector/DOM path is fallback diagnostics only.

**Reduced-capability HTML:** bridge may derive selector/path and text, but UI must label target unstable. Reload/revision can invalidate it.

### Selected text

Selection API exposes current `Selection` and DOM `Range` inside document. It does not define a durable cross-revision text anchor. [Selection API](https://www.w3.org/TR/selection-api/), [DOM Range](https://dom.spec.whatwg.org/#ranges)

**Feasible MVP anchor:**

- artifact/revision and stable enclosing Target ID;
- exact selected quote;
- bounded prefix/suffix text;
- start/end text-node path and offsets as same-revision fallback;
- optional human-readable section label.

On new revision, resolve only within stable enclosing target. Exact quote plus prefix/suffix may propose candidate. Ambiguous or missing match becomes stale and requires human re-targeting; never silently attach feedback to a different passage. This is feasible initial scope if re-anchoring remains conservative.

Artifact scripts can mutate selection/DOM before bridge capture. Parent should show quote/context in queue for human review.

## Explicit artifact state

DOM snapshot, form values, history URL, and local storage do not capture arbitrary application state such as closures, framework stores, pending async work, in-memory caches, canvas internals, or third-party widget state. Structured clone supports a defined set of data types, not arbitrary executable state. [Structured clone](https://html.spec.whatwg.org/multipage/structured-data.html#structuredserialize)

**Required contract:** artifact may register bounded, versioned export/import hooks returning structured-cloneable data. Bridge may automatically capture registered native control values as explicitly exposed exploration state. State is context, not authority.

Rules:

- state hook optional; absence means no restoration promise;
- cap bytes/depth/count and reject unsupported values;
- associate state with artifact and source revision;
- import only when artifact declares compatible state-schema version;
- state restoration failure does not discard feedback;
- never serialize cookies, storage wholesale, arbitrary inputs, or password/file control values;
- do not call export on every pointer event—capture on queueing/submission or explicit scenario change.

## Live reload and persistence

Because trusted parent is not reloaded when artifact iframe changes, it can preserve:

- unsent queue/drafts;
- feedback IDs and edit order;
- last durable submission state;
- selected mode and parent UI state;
- explicit exported artifact state when hook succeeds.

Reload flow:

1. runtime announces candidate revision to parent;
2. parent keeps queue and asks old bridge for bounded explicit state;
3. parent retires old load ID and loads new frame/revision;
4. new bridge handshakes on fresh MessagePort;
5. parent imports compatible explicit state;
6. queued target refs remain pinned to original revision and are revalidated; stale items stay editable;
7. parent never marks a submitted item delivered merely because new artifact loaded.

`sessionStorage` is scoped to origin and top-level context; opaque sandbox/storage behavior is unsuitable as sole recovery store. Durable queue belongs in runtime persistence, with parent browser storage only as additional draft cache. [Storage Standard](https://storage.spec.whatwg.org/)

## Loopback runtime and browser transport

### Loopback security

Browsers treat loopback origins as potentially trustworthy for secure-context purposes, but that does not authenticate process or session. [Secure Contexts § potentially trustworthy origin](https://www.w3.org/TR/secure-contexts/#is-origin-trustworthy)

Runtime must:

- bind loopback only (`127.0.0.1` and/or `[::1]` deliberately; not wildcard interfaces);
- choose one canonical host/origin and validate exact `Host`/authority to reduce DNS-rebinding exposure;
- use OS-selected or collision-checked port;
- require unguessable session capability independent from session/artifact IDs;
- reject requests whose session capability and resource IDs do not match;
- validate `Origin` for mutating HTTP and WebSocket handshakes;
- require JSON/custom header for mutations and reject form-encoded simple requests;
- cap body/message sizes and rate;
- avoid logging credentials or sensitive feedback;
- expire/rotate browser bootstrap capability on session end or ownership reset.

CORS controls whether scripts can read cross-origin responses; it is not authentication and does not stop all cross-origin request sending. Fetch standard defines CORS/preflight behavior. [Fetch CORS protocol](https://fetch.spec.whatwg.org/#http-cors-protocol)

WebSocket servers receive an `Origin` header from browser handshake and RFC 6455 directs servers to use it against unauthorized cross-origin use. Origin checking must accompany session capability and exact route validation. [RFC 6455 §4.2.1](https://www.rfc-editor.org/rfc/rfc6455#section-4.2.1), [§10.2](https://www.rfc-editor.org/rfc/rfc6455#section-10.2)

### Capability placement

**Security implication, not final design:** avoid long-lived bearer token in query strings because URLs enter history, logs, crash reports, copied links, and Referer paths. Prefer short bootstrap secret delivered in fragment or one-time URL, exchanged by trusted parent for an HttpOnly/SameSite credential or memory-held channel authorization. Multiple Collaboration Sessions require separate server-side authorization scopes even if browser shares one origin.

A URL fragment is not sent in HTTP request, but trusted bootstrap JavaScript can read it; artifact frame must never receive it. Exact cookie/token design remains for runtime architecture ticket.

### Transport choice

- HTTP handles bootstrap, artifact/revision resources, queue mutations, and recovery reads.
- WebSocket or EventSource can push revision/status/response changes to parent.
- Plain long poll remains viable fallback.
- Transport event means “state changed”; client rereads durable state. Do not make one transient socket frame sole acknowledgement.

No browser transport itself wakes coding agent. Agent delivery remains CLI/harness concern.

### Private Network Access

Browser restrictions on public pages reaching private/local networks continue evolving. Chromium documents Private Network Access preflights and later local-network permission work; Firefox support/policy differs. [Chromium PNA overview](https://developer.chrome.com/blog/private-network-access-preflight/), [WICG Private Network Access draft](https://wicg.github.io/private-network-access/)

MVP avoids dependency by serving trusted chrome and runtime from same loopback origin, not from a public HTTPS page. Re-test if remote UI, hosted chrome, LAN phone access, extensions, or arbitrary production sites enter scope.

## Sensitive-input and artifact-content rules

- Do not inspect unrelated tabs/pages; only loaded artifact frame.
- Do not automatically collect all form values, storage, clipboard, file inputs, password fields, hidden fields, or network traffic.
- Structured controls opt in with declared key, type, label, allowed values, and current value.
- General DOM excerpt is bounded around selected Target; richer context requires explicit agent request and user-visible retrieval.
- Screenshot is last-resort explicit capture, never background surveillance.
- Artifact text is quoted context. Runtime and portable agent instructions must say it is not trusted instruction, even if text says “run,” “deploy,” or “ignore prior rules.”
- Browser Send authorizes only delivery of collaboration feedback. Acceptance and consequential actions remain separate human acts.

## Baseline browser support classification

### Standards-stable, expected across current Chrome/Edge/Firefox

- iframe sandbox and opaque origin;
- CSP response headers;
- `postMessage`, `MessageChannel`, structured clone;
- Selection/Range inside frame;
- fetch/CORS, WebSocket, EventSource;
- IndexedDB/local storage in trusted parent;
- native controls and pointer/keyboard events.

### Must still be tested as combinations

- opaque sandbox + selected CSP + injected classic bridge script;
- local asset loading and module scripts;
- selection inside editable/contenteditable/shadow DOM;
- iframe reload and MessagePort retirement;
- IPv4/IPv6 loopback and browser origin serialization;
- browser restore/crash behavior for parent drafts;
- assistive technology and keyboard annotation;
- Firefox versus Chromium behavior for blocked navigation/download/network attempts.

### Not promised

- arbitrary cross-origin/production-site annotation;
- restoration of arbitrary application state;
- durable text anchoring across unconstrained rewrites;
- trustworthy context claims from hostile artifact script;
- remote/LAN browser access;
- automatic agent wake from browser event.

## Decisions carried forward

1. Use opaque-origin sandboxed artifact frame and trusted parent chrome.
2. Keep queue, submission, status, persistence, and acceptance outside artifact.
3. Require in-frame bridge for exact targeting and explicit state hooks.
4. Treat bridge data as untrusted context and validate every record.
5. Serve core UI/runtime same-origin on loopback; artifact gets no direct persistence/agent API credential.
6. Use server CSP in addition to sandbox; network denied from artifact by default.
7. Selected-text annotations fit MVP only with stable enclosing target, quote context, and conservative stale handling.
8. Preserve queue through parent/runtime state; reload cannot imply delivery or acceptance.
9. Revisit exact CSP and capability-cookie bootstrap after artifact packaging/runtime decisions.
