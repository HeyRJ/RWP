# RWP — setup instructions (for your assistant)

You are setting up the Remote Working Protocol for someone who leaves a Mac at home and travels with a laptop. Follow these steps in order.

## Before you start

Tell the user, in one or two sentences, what you're about to do: look around this Mac (read-only), ask a few questions, fill in `config/` and a few markdown files in this folder, run a read-only check, and touch nothing outside this folder.

Confirm three things before writing anything:
1. This folder is on the Mac that stays home, ideally at `~/Projects/RWP`.
2. Their copy of the repository is **private**. If it's public, stop and ask them to make it private first.
3. You can run commands on this Mac.

## Step 1 — Look first, then ask

Read these yourself instead of asking (all read-only):
- `scutil --get LocalHostName`, `id -un`, `sw_vers -productVersion`
- `/Applications/Tailscale.app/Contents/MacOS/Tailscale status` (or `tailscale status`): this Mac's name and IP, and the other devices
- `ls /Volumes`, `launchctl list | grep -v com.apple`, `lsof -nP -iTCP -sTCP:LISTEN`: what runs here
- `fdesetup status`, `pmset -g`, `ipconfig getifaddr en0; ipconfig getifaddr en1`

Then ask, one at a time. Keep it conversational; the files can be corrected later.
1. Which devices will you carry? Match them to the Tailscale names you found.
2. Of what's running here, what must stay up while you're away, what should just be watched, and what doesn't matter? Show the list you found.
3. Which external drives must stay mounted?
4. Pushes to your phone: an ntfy server URL and topic, or none for now?
5. What will you call the pinned Claude conversation linked to this Mac? (Default "Home Mac control".)
6. Who at home could check the power or press a button for you? A first name for `runbooks/home-contact.md`. No passwords, ever.
7. Where are your password manager and your 2FA backup codes? Names and places only, never the secrets.

## Step 2 — Show before writing

Write these in order, showing each one's content before writing it:
1. `config/rwp.conf`, from `config/rwp.conf.example`
2. `config/services.tsv`, from `config/services.tsv.example` (TAB-separated; use `-` for an empty field)
3. The rows of `inventory/CREDENTIALS_MAP.md`: names and locations only
4. The contact name in `runbooks/home-contact.md`

Replace every `<PLACEHOLDER>` you touch. Never write a password, key, token, recovery code or ping URL into any file.

## Step 3 — First preflight

Run `bin/rwp preflight`. Then update the status table in `SETUP.md`: every FAIL and WARN becomes a status, with the script's line as evidence. Don't soften failures; the point is to meet them now, not in a hotel. Set the "As of" line in `STATE.md`.

Write `LOG/RWP-S1.md`: what you found, what you set, what's still open.

## Step 4 — Finish

Tell the user:
- Where the files are, and the three moments (README).
- The first-time walkthrough in `SETUP.md`, in order: stop the restarts, pin the address, a second SSH key, the outside heartbeat, break-glass, the laptop.
- The checks only they can do: `rwp handoff` from the laptop on a phone hotspot, the phone on mobile data, and a reboot test and a power-cut test when nothing important is running.

## Rules while you do this

- Read-only on the system: never change settings, restart anything, or touch other repositories. Hand the user the command instead.
- Never reboot the Mac.
- Check the clock (`date`) before saying anything about time.
