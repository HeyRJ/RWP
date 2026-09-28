# RWP — CLAUDE_RULES

**Boot order:** `CLAUDE_RULES.md` → `STATE.md` → `PROTOCOL.md` → latest `LOG/RWP-*.md`

## Hard rules

1. RWP lives in `~/Projects/RWP` on the home Mac (and a clone at the same path on the laptop). Nothing in `~`.
2. RWP reads other projects; it never stops, restarts or reconfigures their processes, and never touches their git state. Each project's own rules still apply.
3. No secret values in this repo, ever: credentials appear by name and location only (`inventory/CREDENTIALS_MAP.md`).
4. Never change power/sleep, FileVault, Tailscale, SSH, Screen Sharing, firewall, network or update settings. Hand the user the command or the System Settings path; they run it (most need sudo anyway).
5. Never reboot, shut down or log out the home Mac as part of RWP. The reboot test (`SETUP.md` S11) is the user's to schedule, when nothing is running.
6. Run `date` before any statement about time ("tonight", "before you leave", ETAs).
7. Git: only this repo, and only to a private remote. Commit ledger files by explicit filename.
8. Report with evidence: ACTIVE / NOT ACTIVE comes from the script's output, quoted. Say what was verified and what was assumed.

## AWAY mode — applies when `STATE.md` says AWAY, to every assistant session on the home Mac, in every project

- **A1 Remote form.** Every command handed to the user must run from the laptop: `ssh <user>@<home-mac> '<cmd>'`, or inside tmux (`ssh -t <user>@<home-mac> tmux new -A -s <name>`). Monitor lines short enough to type on a phone.
- **A2 Nothing that needs hands to undo.** No reboot/shutdown/logout; no network, Tailscale, SSH, Screen Sharing or firewall changes; no sleep settings; don't quit the Claude app or Tailscale; don't eject drives; no macOS or app updates. Only if the user says so, knowing they're away.
- **A3 Restart only with a way back.** Before restarting any service, say how it comes back if the restart fails (launchd KeepAlive? a manual command?). No answer, no restart.
- **A4 Jobs started while away** are detached (launchd / tmux / nohup + log file), their first progress line is verified, a monitor line in remote form is given, and they're added to the away sheet.
- **A5 Money.** Every rented box stays in `state/AWAY_SHEET.md` with its hourly cost and how to stop it. No new spend without asking. Only the user stops them.
- **A6 Disk.** Check free space before any large write; a full disk while away takes services down with it.
- **A7 Log it.** One line per remote session in the trip log `LOG/RWP-T<N>.md`: time, device, what changed.

## Working with the user

- Surface gaps and risks without being asked; honest critique over agreement.
- Decisions that are theirs (spend, FileVault, which services must stay up, anything irreversible): lay out the options and stop.
- Keep the ledger current: `STATE`, `DECISIONS`, `SETUP`, and a `LOG` entry per session and per trip.
