# Release one: two hosts, one attention list

A successful release lets one operator identify a blocked remote Pi among identically named sessions, observe it, deliberately open the live TUI, and continue using the local list when SSH drops. Requires a configured terminal emulator, local Qt and an hc version whose schema we support on both hosts; host setup and SSH trust are operator prerequisites. No network-facing dashboard, remote Signalrail service, writes through the broker, history indexing or app-managed restart. No GDP/Lean dependency in the read path.

## First implementation ticket R1: can a remote row be trusted after reconnect?

Do **not** build Qt, a broker or a real agent integration for this ticket. When separately authorized, use two disposable Linux test users/containers, fresh SSH credentials restricted to them, isolated hc state on each, and one inert long-lived test program per host with the same label. No production SSH alias, user sockets or agent transcripts. Register only the disposable peer. Record versions, before/during/after machine and terminal snapshots, ordered arrival times of events and connection transitions, and whether a record can be distinguished as live vs cached. Redact paths and identities before checking in fixtures.

1. Confirm initial host-qualified rows are distinct. Interrupt only the disposable SSH link. Does the local core expose the loss and continue to answer local reads?
2. While disconnected, replace the peer program with a same-label program; restore SSH. Record address, generation and incarnation evidence for both. If address reuse isn't observed, mark that case untested and replay a same-address replacement fixture separately; don't claim the fixture proves live behavior.
3. **Pass:** loss and replacement are distinguishable; implement invalidation and explicit reselection against recorded evidence. **Partial:** reachability known but incarnation ambiguous; invalidate all remote selections on discontinuity, never restore them by address. **Fail:** no reliable fresh host view; disable remote handoff, keep unknown rows diagnostic-only and block the two-host release pending an authoritative backend probe or contract. A buildable local panel is not a passed VPS milestone.

**Deliverable:** a short observation table, redacted fixtures and an explicit pass/partial/fail recommendation. No application code required to close R1. [B1](decisions/001-release-broker.md) is an independent broker-choice experiment; passing it does not substitute for R1.

## Next gates and accountable checks

- **R2, adapter contract:** fake snapshot plus dropped, duplicated, reordered and overflowing invalidations. Index must converge after resync; no stale row inherits focus. Capture backend call names and fail if a mutating method is reachable. This contract test, not a proof of future dispatch, protects v1.
- **R3, desktop:** operator names their terminal emulator and preferred shortcut. Test explicit argument-vector launch, observer unable to type or resize, accessible labels, keyboard-only use and close/reopen without killing the program. If launcher syntax is unsupported, require a configured supported launcher; no shell-string fallback.
- **R4, two-machine acceptance:** operator identifies VPS architecture, hc version, SSH alias and whether survival after logout is required. With separate approval, repeat R1 with disposable real Pi sessions; disconnect while observing, close the panel, reconnect and reselect the correct host. Test logout independently of network interruption. If provider status is missing show Not reported. If logout persistence is absent report the limitation, do not change host policy.

### G1: Check the shipped handoff rule

After R1 defines the freshness inputs and before enabling the R3 launcher, pin gdp-wired in the development toolchain. Check the production decision against the [handoff eligibility laws](../ARCHITECTURE.md#formal-assurance-and-tests). Require checked, assumption-free proofs; for extraction require exhaustive translation. Keep negative controls: removing the epoch check must admit a stale selection, and treating a refresh as consent must produce an unsolicited interactive request. CI must reject both mutants with witnesses. A missing, sampled or unsupported translation is **unproven**, not a passing gate. R3b remains necessary: the prover does not control the backend between validation and launch.

### R3b: Swap during handoff

In an isolated launcher fixture, select a labeled disposable program A and pause between final row validation and emulator launch. Replace A with B at the same address; resume the launch. Record the selected reference, actual attach target, mode, and whether any input from panel activation reached B. Use a real disposable backend when it can reproduce reuse; label a fixture-only result as such. Ordinary Enter must open details, never an interactive PTY. Observe must send no input or size changes. If a documented atomic incarnation guard exists, require stale-target rejection. Otherwise keep the current-address warning and explicit non-default interactive action; never advertise observation as binding a later attach. If freshness is unknown, suppress both launch actions. If any activation keystroke reaches B, disable interactive handoff and block two-host sign-off pending a fix. Do not run this test against user sessions.

### R4a: Attention transition

Mandatory part of R4, to run only with separate approval. Use two identically named disposable Pi sessions, one on each host. Record the Pi and hc versions and identify the supported status-reporting integration, its version/configuration, and the exact benign input request used. Do not synthesize status events or infer blocking from terminal text. For each checkpoint save the owning host and terminal address, ordered raw `hc` status records (redact private text only), and the corresponding rail rows:

1. **Baseline:** both sessions are reachable and neither is requesting input. Identify both rows by host, not name.
2. **Blocked:** trigger the input request on the remote Pi only. Require a fresh agent-reported `blocked` status from that terminal and verify that only the VPS row enters **Needs input**. Inspecting the row must not clear it.
3. **Resumed:** answer in the remote Pi’s live terminal. Require a fresh non-blocked report from that same terminal and verify the VPS row leaves **Needs input**. The local row must remain outside the group throughout.

Missing reports must render as **Not reported**. This is a correct fallback, **not a pass**: if the supported integration cannot demonstrate both transitions, record the missing evidence and leave the blocked-agent release criterion unmet. Neither a connection change nor opening the terminal counts as a resume signal.

### Later writes

- **Later write gate only:** no claim/answer buttons until atomic receiver-side guards exist. Use reducer proofs for confirmation-to-payload binding and no second claim, a transaction concurrency test for the unique attempt, and real-process fault injection for loss between send and receipt. A solved reducer law cannot make an unsafe receiver safe.
