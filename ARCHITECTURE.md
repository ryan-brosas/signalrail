# Signalrail: Linux agent desk

**Status:** architecture proposal, not a working app. **Target:** one trusted operator, one Omarchy workstation and one or more headless Linux hosts. The first release should answer “which agent needs me, on which machine, and how do I get back to it?” without moving an agent, replacing its TUI or exposing the VPS to a new network service.

## Release-one design package

Start with the [product blueprint](docs/product-blueprint.md) for first launch through retirement and the [five-screen design board](docs/screen-board.html) for the proposed desktop experience.

- [Operator flow, wireframes and Reflex feature decisions](docs/operator-flow.md)
- [Broker choice and reversal experiment](docs/decisions/001-release-broker.md)
- [First ticket, evidence gates and unresolved checks](docs/first-release.md)

Release one ends at the local-plus-VPS handoff (gate 2). No Ask composer, broker dispatch, persistent history or model key is required. The read index is in memory, including a last-seen snapshot only while the broker lives. Loss of that cache is not loss of a terminal.

## Desktop experience

A compositor shortcut or launcher opens a keyboard-first Qt Quick panel. Group rows by Needs input, Working, Other and Unreachable. Show machine, project, agent and status in text, not just color. Example: `VPS | api/tests | Pi | Needs input: permission`. Two projects with the same basename on different machines stay distinct. Enter opens handoff details; Observe opens a terminal without typing or resizing. Interactive launch is a separate non-default action with the current address visible. Preserve keyboard focus by identity when rows move. A changed selection must never receive a keypress intended for the old one.

The inbox puts questions before updates. Reading a card clears only the local unread marker; only the agent can clear its blocked state. No actionable choice inferred from a screenshot or terminal title. A shell or agent without a status report displays **Not reported**, not **Idle**. Notifications conceal prompt text by default. On SSH loss the VPS section shows *Last seen at …* rather than *Done*. Disable actions there, keep local agents usable, keep drafts in memory, and offer a fresh inspection after reconnect.

Ask is a destination picker first and a composer second. Show the machine and folder above the draft and again on confirmation. An optional Jev ranker selects only among candidate IDs already filtered by the broker; it cannot invent a target, rewrite the prompt, or dispatch. A manual choice beats a late ranking response. With no model key the picker still works. External ranking is opt-in and must explain what text and metadata leave the workstation.

## What to reuse

[Reflex at the inspected revision](https://github.com/monotykamary/reflex/tree/e17584d834599c4fc4187a23086ad1bb28b972bc) separates a Swift desktop from a Bend policy daemon. Its [laws](https://github.com/monotykamary/reflex/blob/e17584d834599c4fc4187a23086ad1bb28b972bc/daemon/LAWS.bend) explicitly exclude the I/O shell. Keep the distinction. The [MIT license](https://github.com/monotykamary/reflex/blob/e17584d834599c4fc4187a23086ad1bb28b972bc/LICENSE) requires retaining notices for copied material. Dependencies have their own terms.

Use [herdr-core](https://herdr.dev/core/) on both machines. The local installation is `hc 0.1.22`; inspected `hc schema`, `hc attach --help`, `hc events --help`, `hc connect --help` and `hc send --help` without enumerating personal sessions. `hello` advertises features, methods, generation and logout information. Terminal records include address, creation time, reported status and turn. `hc attach --observe` is a native observation mode. `terminal.send` accepts a terminal address and optional status guard; its receipt contains a turn, but `started` can be null. The inspected send schema has no expected prompt ID, expected terminal incarnation or idempotency key. The subscription schema exposes filters but no replay cursor. Treat installed capabilities, not upstream screenshots, as the contract.

**Broker recommendation:** use a small, read-only TypeScript broker on the workstation for release one. It needs neither dispatch rules nor a proof runtime. [ADR 001: release broker](docs/decisions/001-release-broker.md) distinguishes the `hc` contracts from assumptions and specifies the disposable Linux test that would reverse this choice in favor of Reflex’s Bend daemon. The routing, confirmation and ledger sections below describe later capabilities, not release-one dependencies. Keep Qt native either way. Reject a web dashboard as the first client: authentication and browser terminal emulation are unnecessary for local access. Reject a second tmux backend for now: PTY persistence alone does not provide equivalent status semantics.

## Topology and contracts

```
Qt Quick rail ─┐                       ┌─ local terminals
CLI ──────────┴─ Unix socket → broker → local hc
                                       └─ hc machine over SSH → VPS hc → VPS terminals
Terminal emulator ─────────────────────── hc attach [--observe] machine/tN
```

The VPS runs hc and the agents, not Qt, a Signalrail database or a public listener. Closing the rail does not signal an agent. The broker is a desktop user service independent of Qt; restart it without restarting hc. Interactive terminal input belongs to the human and emulator, outside the routing policy. Do not claim the broker controls all human keystrokes.

| Logical actor | Owns | Boundary |
|---|---|---|
| Backend adapter | Version negotiation, bounded event ingestion, capability flags per machine | No permission to choose or approve work |
| Index | Live snapshots, freshness and read markers | No reconstruction of live sessions from history alone |
| Router | Ranked candidates or manual selection | No effect executor, keys or transport credentials |
| Policy and ledger | Validated transitions and durable one-use claims | No direct terminal I/O |
| Worker | Final validation, one backend call, classified receipt | No alternate destination or automatic resend |
| Qt/CLI | Presentation, explicit selection and confirmation | No raw backend RPC passthrough |

Use a versioned, size-bounded local socket protocol under `$XDG_RUNTIME_DIR/signalrail/` in a private directory with peer UID validation. **Release-one IPC allowlist (exhaustive):** `List()` returns `{brokerEpoch, revision, hosts, rows}`; `Watch(after)` streams index changes or `ResyncRequired`; `Doctor()` reports observed versions, connectivity and capability limits without applying repairs. Enforce exact method names and per-method argument schemas at ingress. Reject every other method or invalid argument before any request-driven `hc` call, process launch or queued effect. No raw RPC, batch forwarding, alternate write listener or administrative bypass. Observe and interactive attach are explicit terminal launches by the desktop client, not broker methods. `Prepare` and `Confirm` below are future design only and must be rejected in v1. New major version without compatible negotiation fails closed. Enforce a single broker instance and use argument vectors for terminal launches, never an interpolated shell string.

**Identity:** a machine alias can be renamed, a conversation resumed in a new process, a kept terminal replaced by a shell, and a short address reused. Keep machine registration, terminal incarnation and conversation ID separate. Use backend registration/generation/creation/identity fields only to the extent a disposable test establishes their scope. The local core generation alone cannot establish remote process identity. Re-registration or a continuity gap invalidates outstanding confirmations. When the adapter cannot prove continuity, display the snapshot as historical; require fresh human selection to open a terminal. A preflight check is not an atomic write guard.

**Convergence:** subscribe before fetching a snapshot, buffer invalidations, refresh affected rows afterwards, and periodically reconcile against a full list. Bound buffers, coalesce repeated updates and request a full snapshot on overflow. Reject responses from prior epochs. An open local subscription is not a VPS heartbeat. If remote data may be cached, expose freshness as unknown. When the broker starts, rebuild its index from hc rather than replaying a stale SQLite view.

## Lost receipts and safe writes

Use SQLite on local disk in `$XDG_STATE_HOME/signalrail/` for read markers, host references and intent records. One owner, durable commits, unique attempt per intent. Suggested records: `intent(id,target,epoch,kind,payloadFingerprint,expiresAt,state)` and `attempt(intentId UNIQUE,claimedAt,outcome,backendTurn)`. Avoid prompt bodies, screen history and credentials at rest. Use a keyed digest if content binding must persist. Expire old metadata; expired confirmation tokens never become valid after garbage collection. Drafts are volatile by default.

1. Only an explicit confirmation of the current operation can advance `prepared → claimed`. Cancelled or expired intents are terminal. Validate state, insert the unique attempt and commit **before** making the backend call.
2. A worker may classify a claim as `submitted`, `rejected` (positive evidence of no effect), or `unknown`. A success means backend submission, not agent completion. Crash after commit but before sending is indistinguishable on restart from crash after sending but before saving the receipt. In both cases mark unknown. Never auto-reclaim an abandoned attempt for delivery.
3. Offer “May have been sent: inspect terminal.” If the user chooses to send again after checking, that is a new intent. Reads can retry with capped backoff and jitter; ambiguous writes cannot. This trades an occasional missed send for avoiding a duplicate. It is **not exactly-once remote delivery**.
4. An idle check cannot atomically bind a send to a particular incarnation. Keep broker-originated sends disabled until the backend demonstrates an equivalent guard or gains an operation such as `send_if(incarnation, status, intentId, payload)` with receiver-side deduplication. Question answers also require expected prompt version and choice validation at the receiver. Until then the card opens the live terminal. Never substitute `--force`, simulated number keys or a screen hash for that contract.

## Formal assurance and tests

**G1: handoff eligibility.** Adopt gdp-wired as a development/CI check for the first Signalrail-owned handoff rule. Given a selected row and the latest normalized observation, an unavailable host, a changed epoch, a different target reference or a cleared selection must yield **no launch offer**. A fresh matching observation may offer Observe; refreshing a row or opening its details must never itself request an interactive terminal. Only a distinct operator action may request one. This checks the application's decision given its inputs, not whether hc's observation is truthful or the process survives until attach.

Keep the rule in the component that makes the decision. For Rust or Qt/C++, extract and check the shipped function with the pinned `gdp-wired-cli check`; require exhaustive translation of that function rather than a sampled model. For native TypeScript, execute the checked rule itself. If reusing an existing proven Bend decision, keep its proof instead of writing a duplicate TS model; apply G1 to the new Linux-side handoff guard. Do not introduce a proof daemon or install a checker on the VPS. R1 remains independent of this tooling.

If selecting TS, use [gdp-wired at the inspected local revision](https://github.com/monotykamary/gdp-wired/tree/bf42c65cfc0edf46041b271333720333edbc1462) for the actual transition function, not a lookalike model. Each proposed law has a broken variant that must yield a witness:

| Law to establish | Mutation it must reject |
|---|---|
| Every claim binds one unexpired confirmation to the same target, action and payload. | Keep the token after changing the machine or draft. |
| At most one claim per intent across legal transitions. | Reset unknown to prepared on reconnect. |
| Epoch changes invalidate pending preparations; stale hosts cannot be claimed. | Address reuse preserves a token for the old process. |
| Ranking, observation, cancellation and expiry cannot emit a write. | A high model score skips confirmation. |
| Receipts apply only to the attempt that produced them. | Match a late receipt by terminal address alone. |

Check step and sequence laws, include admissible restored states, validate serialized state, and pin tool versions. CI should require checked proofs without unreviewed assumptions, broken-variant witnesses and replay of generated proof artifacts in Lean. If Bend is retained, run its original checker and add laws to the code that actually executes; don't maintain rival proof models. Reserve bespoke Lean for a specific refinement or induction obligation the automatic tool cannot discharge. No proof has been run for Signalrail yet. The trusted boundary includes parsing, clock/epoch sampling, database transactions, backend identity, SSH and OS behavior. Test those with real processes, concurrency and injected failures; they are not corollaries of a reducer proof.

## Linux integration and security

Package Qt 6/C++/QML with qmake6, `.desktop` entry and Arch PKGBUILD on Omarchy. Use a normal window first, optional edge placement later. An explicit Hyprland bind toggles the panel; a launcher entry works on other compositors. Do not assume a portable global double-tap shortcut. Follow Omarchy theme changes, support fallback colors, visible focus, accessible names and no hover-only actions. Use a configured terminal launcher and test it with paths containing spaces. A headless CLI can inspect the local broker without a graphical session. [XDG paths](https://specifications.freedesktop.org/basedir-spec/latest/) separate runtime, state and config.

The backend is more powerful than this UI: it can execute commands, read/write files and kill terminals. Allowlist adapter operations rather than proxying arbitrary methods. Treat labels, paths, status words and model output as untrusted display data: plain text, bounded lengths, no escape interpretation outside a terminal. No default clipboard capture, transcript indexing, SSH-agent forwarding or credential copying. Keep SSH host-key checks and resolve changed keys explicitly. Same-UID malware can operate hc directly: this is a single-user tool, not a tenant isolation boundary. Install a user unit only with user approval. `doctor` should explain capability mismatches, connectivity and logout persistence, not auto-update or restart a host. Distinguish an SSH disconnect, user logout, server reboot and application exit in both UI and tests.

## Milestones and release evidence

| Gate | What ships | Pass condition |
|---|---|---|
| 0: feasibility | Read-only `list` and `doctor` spike; broker choice ADR | Capture redacted schema/fixtures from an isolated `HC_SESSION`. Test reused addresses, kept shell replacement, missing status, duplicate names, and event/snapshot races. Compare a Linux daemon build against the small-adapter option. |
| 1: useful local panel | Keyboard rail, observe and attach | Open a disposable real Pi session, close/reopen the panel, verify Pi stays alive. Focus survives refresh, ordinary shells aren't “working agents,” and Observe doesn't write or resize. Measure idle wakeups and update latency; initial local target p95 under 1 s, not yet measured. |
| 2: local + VPS | Unified list with machine badges, stale state and reconnection | With an approved SSH alias and test account, run same-named sessions on both machines. Interrupt SSH and restart the desktop broker. Local stays usable, remote is not marked done, and reconnection never silently selects the wrong process. Test logout separately. The [R4a attention transition](docs/first-release.md#r4a-attention-transition) must also pass; reconnect and handoff alone are insufficient. |
| 3: guarded dispatch | Manual drafts, optional routing and only supported writes | Concurrent confirms from two clients; crash before/after claim, transmission and receipt; stale prompt and reused address. Proof checks plus real backend guard evidence required. Otherwise retain attach-only behavior. |
| 4: broader parity | Grouped inbox, explicit resume, optional history/search and provider-specific question cards | Fixtures for each supported agent/version, malformed and stale inputs, correct folder on resume, opt-in indexing and deletion. Don't advertise support based on one provider's transcript shape. |

Phone pairing, file transfers, sandbox lanes, migration and embedded terminals are later decisions. Each adds a separate trust or lifecycle boundary. The first meaningful demo is simpler: one real local Pi and one on the VPS, visibly distinct, each accessible from the rail, both unaffected by closing it.

**Open checks:** VPS distro/architecture, hc version and logout policy; authoritative remote generation semantics; installed terminal launch contract; Linux buildability of reflexd and its helpers. No VPS connection, installation, proof execution or application deployment was performed to produce this design.
