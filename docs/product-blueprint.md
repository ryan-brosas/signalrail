# Signalrail from first launch to retirement

Release-one product blueprint. Start at [ARCHITECTURE.md](../ARCHITECTURE.md). Open the [screen board](screen-board.html) locally to review the desktop surfaces. All examples are proposed behavior, not observations. The [operator flow](operator-flow.md) owns shortcuts and feature scope; the [release plan](first-release.md) owns evidence gates.

## Product promise

An operator comes back to two Pi sessions in identically named folders. Within a single window they can identify which one needs input, inspect it without typing and deliberately return to its terminal. First value is identifying the correct remote session, not completing a setup wizard. The primary outcome is a correct handoff, not more activity in Signalrail.

## Operator lifecycle and service ownership

| Stage | Operator experience | Owner and recovery |
|---|---|---|
| Install | Install the workstation package; launch Signalrail. No account creation. | Package supplies desktop entry, Qt client, broker, CLI and user unit. No automatic Hyprland edit, VPS installer or hc restart. Operator starts or approves activation of the user service. |
| First launch | See “Checking local core…” immediately, then local sessions even if no VPS exists. | Broker negotiates capabilities. Missing core, incompatible version and unavailable launcher are distinct diagnostics. Launcher failure never hides the list. |
| Add a VPS | Diagnostics says “No remote machine registered.” Return here after verifying SSH trust and registering the peer in hc. | hc owns registration; SSH owns credentials. No password entry, host-key acceptance or setup shell in Signalrail. Wait for a fresh remote snapshot before exposing actions. |
| Daily use | Launch, filter, identify the host, inspect, observe, explicitly open an interactive terminal if needed. | Broker owns freshness and projection. Qt owns selection. The emulator owns human input. Neither viewing nor closing a card clears a blocked report. |
| Suspend or reconnect | Cached rows say Last seen; previously selected remote targets are not automatically reopened. | Treat wake as a continuity break. Reconcile using R1 evidence. After a broker restart show Checking rather than zero sessions. |
| Upgrade/rollback | Client and broker are upgraded together. Old client detecting an incompatible broker shows a version diagnostic. | Restart Signalrail only. Roll back the pair; there is no release-one database migration or cross-host rollback. Never upgrade hc on behalf of the user. |
| Exit/remove | Close hides the window. To retire Signalrail, stop/disable its user unit and remove the package. | Leave remote sessions, hc registrations, SSH keys and desktop bindings alone. User-owned XDG preferences can be removed separately. No transcripts to export. |

## Screen contracts

**Rail:** one filter, then attention groups. Host always appears before project; keep the host visible on narrow windows by wrapping agent/status beneath the project. Selection is an outline, not the attention color. Preserve selection by reference, never sorted position. Healthy but empty: “No sessions on Local.” Filter returning no matches: “No sessions match this filter” with Clear filter. Unknown host state is not an empty successful query.

**Handoff details:** show full folder, host, address, agent report and last observation. The default launch is Observe. Interactive launch is a separate, non-default action with the text “Opens the terminal currently at VPS/t7. Verify the program before typing.” Refresh the selected reference first. If it changed, cancel; if freshness cannot be established, disable both launches. A successful preflight cannot bind a later attach to the same incarnation. Observation reduces accidental input; it does not eliminate that race. Never forward the UI activation keystroke to the emulator.

**Diagnostics:** show broker, local core, peer, reporting integration and launcher in that order. Each failing row names what is unavailable, what still works and the operator's next step. Do not attribute a generic disconnect to a bad key without evidence. Default diagnostic output contains capability, version and error codes; exclude environment variables, hostnames, folder paths, command lines, screen contents and prompt text. No automatic upload or repair.

**Preferences:** use system theme and launcher by default. Only workstation preferences belong in XDG config. Select a supported terminal adapter by identifier rather than shell text. Don't persist selections or permission prompts. No global shortcut capture or desktop notification requirement for the first release.

## Visual and accessibility contract

Nominal width 480 logical px; 16 px outer inset, 8 px spacing unit, minimum 44 px action rows and a 2 px focus indicator. Use system sans for labels, monospace for addresses. Respect system scaling. When text grows, stack columns and scroll instead of dropping the machine name. Accent denotes focus, amber plus text denotes Needs input, neutral plus timestamp denotes Last seen. No continuously spinning work indicator. Target 4.5:1 text contrast and 3:1 focus contrast against the actual desktop theme; fall back to legible system colors. Letters used as shortcuts must still type normally in the filter. Expose host, folder, status and disabled reason to screen readers; announce a disconnect once, not on every poll. Escape dismisses only the top surface.

## Data and client behavior

A row contains an opaque key scoped to broker epoch, host registration reference and display name, backend address, optional folder/agent label, host freshness, reported state and kind if known, observation time, eligible actions and disabled reasons. It is not a durable conversation ID. Stale or unreachable host state overrides reported activity. For live hosts, blocked goes under Needs input, working under Working, and missing or unrecognized state under Other with a factual label. Process termination is not inferred from disconnection.

Qt and CLI share the same projection. Start with List, then Watch from its revision. An epoch change, missed revision or overflow needs a new List, not merging stale data. CLI JSON carries freshness and schema version: a healthy local view plus stale VPS is a successful **partial observation**; inability to obtain a broker snapshot is an error, never a successful empty array. Qt never forwards arbitrary backend RPC. A terminal adapter builds fixed argv from a supported launcher, validated address and explicit mode. Reject malformed or option-like addresses. Avoid collecting screen text for the index or doctor.

## Design acceptance beyond the backend gates

- **Lifecycle check:** in a disposable account install, close the rail, stop the broker, remove the package. Record whether the owned test terminal survives each step and whether SSH or compositor configuration changed. Destructive coupling blocks release.
- **Usability check:** give a tester the two same-named sessions on the board. Ask them to identify the blocked machine, inspect without typing, recognize stale information, and explain where their answer would go. Repeat keyboard-only at large font scale on the real client before shipping. A wrong-host selection or hidden disabled reason requires redesign.
- **Privacy check:** use a synthetic marker and fake path in test status data; inspect default diagnostics and logs. Leakage blocks release. Remove collection, rather than asking the operator to redact logs later.
- **Handoff check:** use R3b in the release plan. If freshness or deliberate input isolation cannot be validated, disable interactive launch. If only incarnation binding is absent, explicitly offer a current-address handoff, not a guarantee about the prior agent.

No live system, usability test, deployment or proof run is part of this blueprint. Broker choice and first implementation ticket remain in the linked ADR and release plan.
