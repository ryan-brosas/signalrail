# ADR 001: Use a narrow workstation broker for the first release

**Status:** proposed. Applies to listing, attention, observation and interactive terminal handoff, not future prompt dispatch.

## Recommendation

Use a small TypeScript broker on the workstation to translate herdr-core observations into machine-qualified rows. Keep reads and health checks on an allowlist; let a terminal emulator launch native `hc attach` on explicit operator selection. Do not offer a broker send endpoint, one-click approvals, a durable dispatch ledger or a model router in release one. A read-only broker does not need a GDP or Lean runtime. Qt and hc retain the same roles whichever broker wins. No additional VPS service.

The advantage over adapting reflexd is scope, not an assertion that Bend cannot run on Linux. The first release needs an index and freshness policy, not most of Reflex's dispatch machinery. A narrow broker gives the team a small set of backend calls to audit, at the cost of maintaining a protocol adapter and packaging its runtime. Reusing reflexd could instead give us its existing normalization and tested policy, fewer differences from upstream and a path to fuller parity. It would also make our release dependent on its build, helper programs and the ability to exclude writes. A green Bend proof would not establish that the surrounding I/O shell is read-only.

## What the evidence supports

| Source and contract | What it does **not** establish |
|---|---|
| Local `hc 0.1.22`, `hc schema`: `hello` exposes `generation`, `features` and `methods`; terminal records have `address`, `created_ms`, `identity` and `reported`. | That a local generation identifies a remote process, that identity survives a reconnect, or that a missing status means idle. |
| `hc events --help` describes a stream of changes; `events.subscribe` has filters but no replay cursor in this schema. | A lossless snapshot/stream cutover, or proof that cached remote data is fresh. |
| `hc attach --help` documents `machine/tN` and `--observe` as watch-only without typing or resizing. | An atomic check of expected process incarnation at attach time. Selection and launch can race. Human attach is not a broker-verified targeting guarantee. |
| `terminal.send` accepts `terminal`, `text` and optional status guard; its receipt includes `turn` and nullable `started`. | Prompt-version binding, idempotent retry or exactly-once delivery. These are not reasons to port a write-capable daemon into v1. |
| Reflex's pinned [build entry](https://github.com/monotykamary/reflex/blob/e17584d834599c4fc4187a23086ad1bb28b972bc/scripts/build-app.sh) separates daemon and app build modes. Its [law declarations](https://github.com/monotykamary/reflex/blob/e17584d834599c4fc4187a23086ad1bb28b972bc/daemon/LAWS.bend) explicitly exclude backend and runtime assumptions. | A supported Linux daemon package, safe defaults on Linux or correctness of a remote handoff. No Linux build or runtime comparison was run. |

## Reversal test B1 (specified, not run)

**Question:** can the unmodified upstream daemon already serve the required read model on Linux without acquiring write capability? A successful compile alone would not change the choice.

Use one throwaway Linux container matching the workstation architecture, the pinned Reflex commit above, its documented toolchain, a clean home and no mounted user sockets, SSH keys or credentials. Build the daemon-only target and check its existing proofs. Substitute a fixture endpoint for hc: advertise the pinned read capabilities, return `local/t7` and `vps/t7` as two same-named synthetic rows (use native local `t7` on the wire), then remove remote connectivity. Reject and record every undeclared call; disable outbound network during the fixture run. No real VPS, desktop or agents needed.

Collect the build command/tool versions, proof result, dependency list, attempted backend operations and three snapshots: initial list, remote loss, recovered list with a replacement remote terminal. Require distinguishable rows, no old selection silently rebound, local row still usable, remote shown stale or unavailable, and no mutation, public listener or required Apple service. If necessary, first determine whether an existing daemon endpoint can expose those observations; absence is a negative result, not a reason to build an alternate daemon during the test.

**Mandatory ingress gate (planned):** enumerate every listener and client-reachable handler in the proposed release configuration, including legacy routes, aliases, generic forwarding and background jobs a handler can enqueue. Trace the three [allowlisted IPC methods](../../ARCHITECTURE.md#topology-and-contracts) through to their effects. A hidden UI button or a proxy in front of a still-reachable write endpoint is not isolation. Record the handler inventory, configuration and rejection sites. If the reachable surface cannot be accounted for, keep the narrow broker.

Against the actual client ingress in the disposable fixture, establish a valid same-user connection and exercise `List` as a positive control. Then submit (1) a well-formed request for a real mutating method identified in the upstream handler inventory, with valid fixture arguments, and (2) an unrecognized method on the same connection. Record both replies and a per-request fixture call trace, isolating unrelated watcher traffic. Each must receive an explicit method rejection **before any `hc` call**, even a read, and enqueue no deferred work. A timeout, invalid-auth rejection, fixture-side rejection, or empty trace without a rejection reply does not pass. If either probe fails or is inconclusive, retain the narrow broker regardless of build or proof results.

- **All criteria, including the ingress gate, pass with configuration or packaging only:** recommend reusing reflexd instead and keep Signalrail as the Linux shell; record what TS code is no longer needed.
- **Daemon requires application/policy changes, live user data or broad write access:** retain the narrow broker. Record the first blocking dependency/call, not a speculative port estimate.
- **Fixture cannot represent remote reachability or identity:** inconclusive on that capability for both designs. Keep the narrow default, label remote freshness unknown, and require the isolated two-core reconnect test before enabling handoff. Never infer safe continuity from the language chosen.

The engineer owning the first backend ticket records the result here before committing to the desktop integration. Revisit separately if guarded dispatch later becomes release scope.
