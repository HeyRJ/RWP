# RWP — SETUP

One-time work, in the order to do it. `INIT.md` fills the status table from your first preflight.
✅ verified · ❌ missing · ❓ needs you · ⏸ your decision

## First-time walkthrough (about 1.5 h; nothing here restarts anything)

### 1. Restarts (15 min) — S1, checks H8 / R1 / L1 / W1 / W2
What a restart does to a Mac with FileVault ON (ways back in: `runbooks/restart-recovery.md`):

| Restart | Comes back to |
|---|---|
| macOS update | **logged in, on its own**: macOS stashes your login before it reboots (seen on a Mac mini on macOS 27) |
| `rwp restart` (runs `sudo fdesetup authrestart`) | **the login window**: FileVault skipped once, but nobody is logged in, so the Tailscale app, your assistant and your LaunchAgents wait ("goes straight to the regular login window", Der Flounder; E3a confirms). From outside: only with Tailscale before login or a foothold (W1) |
| power out longer than the UPS lasts, a crash, a plain `shutdown -r` | **the FileVault unlock screen**: no Tailscale, no assistant. `rwp unlock` gets past it over SSH from home Wi-Fi (macOS 26+, Apple silicon); from outside only through a foothold (W2) |

Every kind kills the jobs you started by hand; only launchd jobs come back (S22). `rwp status` lists what died (R1).
- **Automatic installs off** (not a lockout, but they kill running work at a time you didn't choose): System Settings → General → Software Update → ⓘ next to Automatic Updates → **Install macOS updates: off**; for zero surprise restarts also **Install Security Responses and system files: off**. Keep "Download new updates" on. Terminal equivalent:
  ```
  sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates -bool false
  sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate CriticalUpdateInstall -bool false   # optional
  ```
  Then update by hand when you're home.
- **Power:** the Mac, the router **and** the modem all need backup power (UPS or house inverter). A Mac that stays up behind a dead router is just as unreachable. A small DC mini-UPS covers a router and modem cheaply. If your inverter has a UPS / Normal (Eco) switch, UPS mode changes over faster: it's the setting meant for computers. Know how long the UPS lasts, and plug its USB cable into the Mac if it has one, so macOS can shut down cleanly before the battery runs out. E4 proves it.
- **Restarts you trigger remotely:** always `rwp restart` (`--dry-run` first shows the way back in and what will stop; it needs `fdesetup supportsauthrestart` = true), never a plain restart. You run it; your assistant never does.
- Check: `bin/rwp preflight` → **H8 PASS**; W1/W2 say which restarts you could get back from while away.

### 2. Stop the address moving (10 min) — S6
macOS can use a private, rotating Wi-Fi MAC; a new MAC gets a new DHCP lease, so the Mac's LAN address wanders. Compare `networksetup -getmacaddress en1` with `ifconfig en1 | awk '/ether/{print $2}'`: different means a private address.
- On the home Mac: System Settings → Wi-Fi → Details… (home network) → **Private Wi-Fi address: Off**. Off rather than Fixed: the unlock screen joins Wi-Fi before your data volume is unlocked, and whether it uses the private MAC there is unknown (E3b checks). An Ethernet cable avoids the question entirely.
- Router admin → DHCP → reserve an address for that MAC. Put it in `SERVER_LAN_IP`. Changing the MAC drops the Mac off Wi-Fi for a moment and can hand it a new address until the reservation is in: do this at home.
- Needed for `rwp unlock` (E3b) and a foothold (`runbooks/foothold.md`).

### 3. A second key that can SSH in: your phone (10 min) — S3, check H2b
- Phone: an SSH app (Termius, Blink…). Create a key **inside the app** (ed25519), protected by Face ID if offered. Copy its **public** key.
- From the laptop: `ssh <user>@<home-mac> 'cat >> ~/.ssh/authorized_keys'`, paste the line, add a comment at the end (` phone-<date>`), Enter, Ctrl-D.
- Test from the phone with **Wi-Fi off, Tailscale on**: run `~/Projects/RWP/bin/rwp status`.
- Check: `bin/rwp preflight` → **H2b PASS**.

### 4. Someone outside the house notices when the Mac dies (10 min) — S8
If your alerts come from the home Mac, a dead Mac is silent, and silence reads as "all fine".
- healthchecks.io (free) → **Add Check**: period **5 min**, grace **10 min**. Email alerts are on by default; add a phone channel if you like. Anything hosted on the home Mac can't do this job.
- Copy the ping URL, then on the home Mac:
  ```
  ~/Projects/RWP/bin/rwp heartbeat install https://hc-ping.com/<uuid> --dry-run   # look first
  ~/Projects/RWP/bin/rwp heartbeat install https://hc-ping.com/<uuid>
  ```
  The URL stays in `state/` and the LaunchAgent, never in git. RWP has two commands that change the system, this and `rwp restart`; you run both.
- The check turns "up" within a minute; send a test notification from the integration. Check: `bin/rwp preflight` → **S8 PASS**.

### 5. Break-glass: losing the laptop and the phone together (30 min) — S18
- Whatever you log into Tailscale with (GitHub, Google, Microsoft…) is the key to your tailnet. Treat that account as the most important one here.
- For that account, your email, your Apple ID and your password manager: make sure 2FA isn't only on the phone, and get recovery codes / an emergency kit.
- Print them and seal them in an envelope at home with someone you trust. Not in the laptop bag, not in this repo.
- Worth knowing: icloud.com/find lets you lock or erase a lost Apple device with only your Apple ID password (a "Find Devices" button skips the 2FA code). A replacement SIM restores your number, but some countries (India, for one) block incoming SMS for 24 h after a SIM replacement, so SMS can't be your only way back.
- Write down *where* things are in `inventory/CREDENTIALS_MAP.md`, never the secrets.
- Later: a hardware security key on your keyring, registered on the accounts above.

### 6. Laptop ready (5 min) — S13–S15
- Clone your private repo to `~/Projects/RWP` on the laptop.
- `~/.ssh/config` alias for the home Mac (`runbooks/laptop.md`). FileVault and Find My ON (it travels).

## Checks only you can do

Do E1, E2 and E4 any time. **E3 stops every running job on the home Mac**: only when nothing is running that you care about.

- **E1 Hotspot handoff (5 min)** — S16, H9. Laptop on your phone's hotspot (home Wi-Fi off), Tailscale on: `~/Projects/RWP/bin/rwp handoff`. Expect "reached … via DERP(…)" or a public IP, SSH ok, Screen Sharing ok, "Receipt left … (ok=yes, lan=no)". Then open `vnc://<home-mac>` and log in once.
- **E2 Phone on mobile data (10 min)** — S5, S17. Create the pinned conversation from the home Mac's Claude app (so it's linked to that Mac). Then, phone Wi-Fi off, Tailscale on: SSH app → `rwp status`; ntfy (if you use it) → `rwp notify "test"` arrives; the pinned conversation → ask it to run `date` on the home Mac.
- **E3 Restart paths (10–15 min each)** — S11. For each, note whether the Mac comes back to **the desktop** (everything starts) or **the login window** (the Tailscale app, the Claude app and your launchd jobs wait for a login), and how long it takes. Then `rwp status`: R1 lists what died, L1 fails while nobody is logged in.
  - **E3a** From the laptop on home Wi-Fi: `~/Projects/RWP/bin/rwp restart --dry-run`, then `rwp restart`. Expected: the login window. Log in with `open vnc://<LAN IP>`. Then `rwp status`: R1 says "planned".
  - **E3b** On the home Mac: `sudo shutdown -r now`. From the laptop on home Wi-Fi: `~/Projects/RWP/bin/rwp unlock` (your macOS password at the prompt). While it sits at the unlock screen, look at the router's client list: which address and MAC does the Mac have? If it isn't `SERVER_LAN_IP`, the unlock screen uses another MAC: reserve an address for that one too, or use Ethernet. Note desktop or login window, then set `PREBOOT_TESTED="<date>"` in `config/rwp.conf`.
  - **E3c** (once a foothold exists) E3b again with the laptop on the phone hotspot.
- **E4 UPS check (5 min)** — S1. Note `uptime`, cut the mains for 2 minutes, restore it. `uptime` shouldn't reset, and the phone on mobile data should still reach the Mac (proving the router and modem stayed up too).
- **E5 Depart dry run (5 min).** `rwp depart --leave "test" --back <date>` without `--activate`; read `state/AWAY_SHEET.md`.

## Later (the ways back in from outside; `runbooks/restart-recovery.md` D compares them)
- **A foothold at home** (`runbooks/foothold.md`): an Apple TV, a spare Android phone or a Raspberry Pi on backup power, as a Tailscale subnet router for **only the home Mac's address** (`<LAN IP>/32`). Covers the unlock screen (`rwp unlock`) and the login window (Screen Sharing to `<LAN IP>`) from anywhere. W1 and W2 detect it on their own; nothing to set in the config.
- **Tailscale that runs before login** (`runbooks/tailscaled.md`): the open-source `tailscaled` instead of the Tailscale app. Covers the login window from anywhere without a foothold; not the unlock screen. At home only: it replaces the thing you reach the Mac through.
- **A hardware security key** (above).

## Status

| # | Item | Status |
|---|---|---|
| S1 | Survive a restart while away: power backup, automatic installs off, a way back in from outside (W1/W2: foothold or tailscaled) | ❓ |
| S2 | Power settings: sleep 0, disksleep 0, autorestart 1, womp 1 | ❓ |
| S3 | Second device that can SSH in | ❓ |
| S4 | Tailnet hygiene: no stale nodes or expired keys; carry-device key expiries in your calendar | ❓ |
| S5 | Claude link: app running; relaunches after a restart (E3); pinned conversation works from the phone (E2) | ❓ |
| S6 | Fixed Wi-Fi address + DHCP reservation | ❓ |
| S7 | Wired network | ❓ |
| S8 | Outside heartbeat | ❓ |
| S9 | Which services must stay up while away (`config/services.tsv`) | ❓ |
| S10 | Home contact walked through `runbooks/home-contact.md` | ❓ |
| S11 | Restart paths: update, `rwp restart` (E3a), plain restart + `rwp unlock` (E3b), through a foothold (E3c) | ❓ |
| S12 | `hostname` matches LocalHostName (cosmetic) | ❓ |
| S13 | Laptop FileVault + Find My | ❓ |
| S14 | Laptop `~/.ssh/config` alias | ❓ |
| S15 | RWP cloned on the laptop | ❓ |
| S16 | First hotspot handoff (E1) | ❓ |
| S17 | Phone kit: Tailscale, Claude app, SSH key, 2FA, provider consoles, ntfy (optional) | ❓ |
| S18 | Break-glass | ❓ |
| S19 | A work laptop is never a fallback for personal infrastructure | rule |
| S20 | This repo is private | ❓ |
| S21 | One line in your assistant's global preferences: *"If ~/Projects/RWP/STATE.md says AWAY, follow the AWAY rules in ~/Projects/RWP/CLAUDE_RULES.md."* | ❓ |
| S22 | Jobs that must survive a restart run under launchd (a LaunchAgent with RunAtLoad/KeepAlive and a log file). Claude Code sessions resume with `claude --continue` in their project folder | ❓ |
| S23 | Tailscale before login (`runbooks/tailscaled.md`), optional if a foothold exists | ❓ |
