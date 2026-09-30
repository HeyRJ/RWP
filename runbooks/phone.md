# Runbook: phone only

Good for: checking, reading a log, stopping a runaway spend, relaunching something. Not for: editing code or long sessions.

## What you have (after SETUP 3 and E2)

- **Claude app** → the pinned conversation linked to the home Mac: one started from the home Mac's Claude app. It stays linked wherever you open it; a fresh chat on the phone is not linked and can only hand you the SSH line. Ask for `rwp status`, a log tail, a service relaunch. The AWAY rules still apply: nothing under A2 without your explicit go.
- **ntfy** (optional, Tailscale on): the topic in `NTFY_TOPIC` for RWP pushes.
- **SSH app** with its own key: `ssh <user>@<tailscale-name>`. Useful lines:
  - `~/Projects/RWP/bin/rwp status`
  - `tmux ls`, then `tmux attach -t <name>` (detach: Ctrl-b d)
  - `tail -n 30 <logfile>`
- **Provider consoles** for anything you rent, logged in, so you can stop it from the phone.

## Rules

- Tailscale on only while you need it; it's how the phone reaches the home Mac.
- Nothing under A2 from a phone unless it's the only way to stop real damage (a runaway spend, a disk about to fill).
