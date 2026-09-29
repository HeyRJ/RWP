# RWP — PROTOCOL

HOME → **depart** → AWAY → **return** → HOME. The mode lives in `STATE.md`. A trip's detail lives in `state/AWAY_SHEET.md` (on the home Mac, not in git) and `LOG/RWP-T<N>.md`.

## 1. DEPART — the night before, or at least a few hours before

Order matters. Everything that needs your hands happens first. Background jobs may keep running after you leave, but only once they're detached, logged and listed.

### 1a. Declare (your assistant asks if you didn't say)
- Leave when; back when (a date); which devices you carry.
- Per project: what must keep running while you're away, and what can stop.
- Any rented compute live or planned.

### 1b. Preflight: `bin/rwp preflight --back <date>` (read-only)

| ID | Check | Blocks ACTIVE? | Fix |
|---|---|---|---|
| R1 | Did the home Mac restart since the last check, and which long-running jobs from then are gone? | no | bring back what you need; launchd for what must survive (SETUP S22) |
| H1 | Tailscale running on the home Mac, online, its key expiry disabled | yes | `open -a Tailscale`; admin console → disable key expiry |
| T1 | Your carry devices' Tailscale keys valid past your return date | yes | re-authenticate that device now |
| T2 | No stale nodes or expired keys in the tailnet | no | remove them in the admin console |
| H2 | SSH (Remote Login) answers on the tailnet IP | yes | Sharing → Remote Login |
| H2b | More than one device holds a key that can SSH in | no | `SETUP.md` 3 (phone key) |
| H3 | Screen Sharing answers on :5900 | yes | Sharing → Screen Sharing |
| H4 | Never sleeps, disks never sleep, restarts after a power cut | yes | `sudo pmset -a sleep 0 disksleep 0 autorestart 1 womp 1` |
| H5 | Claude desktop app running (the Claude link to the home Mac) | yes | `open -a Claude` |
| H6 | SSD free space above the floor; required volumes mounted | yes | move finished outputs to an external drive |
| H7 | Registered services up (`config/services.tsv`) | only `required=yes` | the restart hint in the registry (you run it) |
| H8 | Automatic macOS updates OFF (an update restart logs back in by itself, but kills every job started by hand) | no | Software Update → Automatic Updates → Install macOS updates OFF |
| H9 | Handoff receipt: from the laptop, off the home network, < 24 h old | yes, at activation | on the laptop, on a phone hotspot: `bin/rwp handoff` |
| S1 | FileVault ON has a pre-boot way in | no, but see §4 | `SETUP.md` 1 and Later |
| S2/S3 | Wired network; LAN IP matches config | no | `SETUP.md` 2 |
| S5 | Repos with unpushed or uncommitted work you'll want away | no | you commit/push; RWP never does |
| S6 | Jobs running longer than 30 min (listed, not stopped) | no | detached + logged + monitor line |
| S7 | Money meters: SSH/rsync to public IPs, public IPs in job command lines | no | stop it, or list it with its hourly cost and how to stop it |
| S8 | Outside heartbeat loaded, last ping succeeded | no | `SETUP.md` 4 |

### 1c. Blocking: done before hands-off
1. **Physical.** Power backup covers the Mac, the router and the modem; cables seated; external drives powered; Ethernet in if you can.
2. **Settings that need sudo** (your assistant hands you the command, you run it): automatic updates off (H8), anything in H4.
3. **Code you'll touch from the laptop**: committed and pushed, or you'll work on the home Mac over SSH.
4. **Jobs that should run while you're away**: start them now, not at the door. First progress line verified; detached; monitor line in remote form; listed in the away sheet.
5. **Rented boxes**: stop them, or each goes in the away sheet with its hourly cost, expected end, the most you'll let it spend, and how to stop it from your phone (provider console logged in on the phone).
6. **Handoff from the laptop, off home Wi-Fi** (phone hotspot): `~/Projects/RWP/bin/rwp handoff`. Writes the receipt H9 needs, caches STATE + the away sheet on the laptop, pulls the latest RWP docs.
7. **Phone, on mobile data**: Tailscale on → open the pinned Claude conversation → ask for `rwp status`; then `rwp notify "test"` and see the push arrive.

### 1d. Activate
`bin/rwp depart --leave "<when>" --back <date> --devices "<list>" --activate [--accept "H6v: reason"]…`

- No unaccepted hard failure → **ACTIVE**, or **ACTIVE with accepted risks** (each accepted ID and its reason goes into STATE, the away sheet and the trip log).
- Any unaccepted hard failure → **NOT ACTIVE**. Nothing changes; fix and rerun.
- On ACTIVE: STATE → AWAY, `state/AWAY_SHEET.md` written, `LOG/RWP-T<N>.md` opened (and pushed to your private repo with `AUTO_PUSH=1`), a push to your phone if ntfy is configured.
- Background work keeps running. It is listed, not stopped.

## 2. AWAY
- The AWAY rules A1–A7 (`CLAUDE_RULES.md`) apply to every assistant session on the home Mac, in every project.
- Check in with `bin/rwp status` from the laptop. It caches the answer, so you keep the last known state if the Mac goes dark.
- Every remote session gets a line in the trip log: time, device, what changed.
- Return date moves: rerun `bin/rwp depart --back <new date> … --activate`. It re-checks key expiries against the new date and keeps the same trip.

## 3. RETURN: `bin/rwp return`
1. Checks again: what broke while you were away.
2. Reconcile code between the laptop and the home Mac, each repo by its own rules.
3. Restore anything you switched off for the trip.
4. Anything used on a device that isn't yours: rotate what you typed, remove any Tailscale node you added, sign out stray sessions (`runbooks/borrowed-device.md`).
5. Close the trip log: what happened, what the protocol missed → `DECISIONS.md` / `SETUP.md`.
6. STATE → HOME (the script does this).

## 4. What ACTIVE does not cover (until SETUP closes them)
- **A restart that isn't an update, while FileVault is ON**: power out longer than the UPS lasts, a crash, a plain restart. The Mac waits at the unlock screen; Tailscale, Claude, SSH and every service stay down until someone unlocks it (at the keyboard, or over SSH from the home network; SETUP Later for from anywhere). Update restarts log back in by themselves. Restarts you trigger remotely: `sudo fdesetup authrestart`.
- **Jobs started by hand.** Any restart kills them; only launchd jobs come back. `rwp status` lists what died (R1).
- **A dead Mac nobody notices**, until the outside heartbeat exists (SETUP 4).
- **Anything that doesn't live on the home Mac**, like production on a cloud platform: give it its own break-glass line in `runbooks/borrowed-device.md`.
