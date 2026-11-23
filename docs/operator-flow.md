# From an attention signal to the right terminal

Design sketches, not screenshots. The workstation panel groups attention across hosts; the host column never disappears during filtering. This example deliberately uses the same agent, name and folder on two machines.

```
Signalrail                         Local: live    VPS: live
Filter: [                         ]        2 sessions
Needs input
 > VPS     api/tests    Pi    Permission requested
Working
   Local   api/tests    Pi    Working

Enter: details     O: observe     /: filter     Esc: close
```

Invoke from launcher or a user-configured bind. Up/Down moves selection, Tab moves among controls and Esc first clears a filter, then closes. Keep the selected row stable by reference, never by ordinal. If it vanishes, clear selection; the next item does not inherit Enter. `O` opens a native observe-only terminal. `Enter` opens handoff details with Observe as the default action. Open interactive terminal is a separate, non-default action: show the current machine/address, revalidate the selection, and warn that the address may now refer to another process. No panel keystrokes are forwarded. Detach using the backend's documented Ctrl+backslash shortcut. Don't automatically raise or focus a terminal because its status changed.

```
VPS unavailable                         Last seen 14:32
   VPS     api/tests    Pi    Last seen: permission requested
   Observe unavailable          Open unavailable
Local remains connected.                [Show diagnostics]
```

On reconnect, rebuild the remote view; never reopen the old terminal automatically. If there is no authoritative current view, leave actions disabled. If the historical item can't be matched safely, leave it unselected and show a fresh row. A human can select the new row; this still isn't an atomic incarnation-bound attach.

## Three failure traces

| Event | What the system knows | What the operator sees and can do |
|---|---|---|
| SSH breaks after a manual send or before a future broker receipt | V1 has no send ledger and cannot tell whether a human's Enter reached Pi. For later broker sends a durable claim proves only that an attempt was reserved. | V1: disconnected/last seen; reconnect and inspect the transcript. Future broker: “May have been sent”; no automatic retry or suggested duplicate. |
| The address `vps/t7` is reused while the list is open | Alias/address equality proves routing, not process continuity. A differing signature or lost observation invalidates selection. | Drop old handoff, refresh and explicitly select the current row. If hc cannot establish freshness, inspect via the operator's own terminal instead of offering a stale shortcut. |
| Permission prompt A changes to B while its card is open | A blocked report can say why the agent stopped but doesn't bind keyboard input to A. A newer report supersedes the display, not the operator's consent. | Update or withdraw the card; no numbered buttons and no inherited answer. Observe or open the live TUI, read B, and answer there. |

## Reflex feature decisions

Based on [upstream feature inventory](https://github.com/monotykamary/reflex/blob/e17584d834599c4fc4187a23086ad1bb28b972bc/README.md). Decisions are for release one, not promises of permanent exclusion.

| Upstream experience | Decision and reason |
|---|---|
| Edge tab, rail, menu-bar count | Adapt to a normal keyboard-accessible Qt window. Optional edge positioning later; no compositor-specific hover requirement. |
| Ask bar, scoped search, agent/folder selection and launching | Keep list filtering; defer composing, starting and routing. Launching creates state on a host. |
| Routing memory, override learning, uncertain-route confirmation | Defer memory and models; manual host-qualified selection is the default. A future suggestion is never authorization. |
| Questions and inbox | Adapt to an attention list and open-to-answer. Defer rich cards, transcript-derived updates and persistent unread history until versioned identity and privacy controls exist. |
| Resume, forks and stopped-agent recovery | Defer. A lost PTY and a resumable conversation are different objects. |
| Side questions (`btw`) | Defer. Correct behavior varies by agent and may create a fork. |
| Full terminal, sharing, resizing and keyboard behavior | Keep native attach/observe; omit an embedded emulator in v1 to avoid competing input and resize rules. |
| Other machines and scoped targets | Keep hc's existing SSH machine registrations. Adapt to Linux host badges, reachability and recovery; omit automatic provisioning. |
| Phone pairing | Defer. A mobile client creates a new authentication and network boundary. |
| Sandboxes, lanes, pause/renew and inter-machine migration | Defer. Billing, file copying, secret handling and resuming sessions need separate authorization and recovery designs. |
| Images, remote pasting and Open In integrations | Defer. Plain terminal handoff needs no implicit file transfer. |
| Keep awake | Omit. Desktop power policy belongs to the operator; disabling suspend does not establish VPS durability. |
| Guided setup and updates | Adapt to read-only doctor. Do not install integrations or restart hc for the user. |
