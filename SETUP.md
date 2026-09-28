# RWP — SETUP

One-time work, in the order to do it. `INIT.md` fills the status table from your first preflight.
✅ verified · ❌ missing · ❓ needs you · ⏸ your decision

## First-time walkthrough (about 1.5 h; nothing here restarts anything)

### 1. Stop the restarts (15 min) — S1, check H8
The usual chain: FileVault ON + automatic macOS updates ON + a power cut. Any restart parks the Mac at the unlock screen; no Tailscale, Claude or SSH until someone types the password at the keyboard.
- **Automatic installs off, permanently** (a server shouldn't restart itself): System Settings → General → Software Update → ⓘ next to Automatic Updates → **Install macOS updates: off**; for zero surprise restarts also **Install Security Responses and system files: off**. Keep "Download new updates" on. Terminal equivalent:
  ```
  sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate AutomaticallyInstallMacOSUpdates -bool false
  sudo defaults write /Library/Preferences/com.apple.SoftwareUpdate CriticalUpdateInstall -bool false   # optional
  ```
  Then update by hand when you're home, when you can unlock it at the keyboard.
- **Power:** the Mac, the router **and** the modem all need backup power (UPS or house inverter). A Mac that stays up behind a dead router is just as unreachable. A small DC mini-UPS covers a router and modem cheaply. If your inverter has a UPS / Normal (Eco) switch, UPS mode changes over faster: it's the setting meant for computers. E4 proves it.
- Check: `bin/rwp preflight` → **H8 PASS**.

### 2. Stop the address moving (10 min) — S6
macOS can use a private, rotating Wi-Fi MAC; a new MAC gets a new DHCP lease, so the Mac's LAN address wanders. Compare `networksetup -getmacaddress en1` with `ifconfig en1 | awk '/ether/{print $2}'`: different means a private address.
- On the home Mac: System Settings → Wi-Fi → Details… (home network) → **Private Wi-Fi address: Fixed** (or Off).
- Router admin → DHCP → reserve an address for that MAC. Put it in `SERVER_LAN_IP`.
- You'll need a fixed address for the pre-boot unlock (E3) and a LAN foothold (Later).

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
  The URL stays in `state/` and the LaunchAgent, never in git. This is the one RWP command that changes the system; you run it.
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

Do E1 and E2 any time. **E3 and E4 stop every running job on the home Mac**: only when nothing is running that you care about.

- **E1 Hotspot handoff (5 min)** — S16, H9. Laptop on your phone's hotspot (home Wi-Fi off), Tailscale on: `~/Projects/RWP/bin/rwp handoff`. Expect "reached … via DERP(…)" or a public IP, SSH ok, Screen Sharing ok, "Receipt left … (ok=yes, lan=no)". Then open `vnc://<home-mac>` and log in once.
- **E2 Phone on mobile data (10 min)** — S5, S17. Create the pinned conversation from the home Mac's Claude app (so it's linked to that Mac). Then, phone Wi-Fi off, Tailscale on: SSH app → `rwp status`; ntfy (if you use it) → `rwp notify "test"` arrives; the pinned conversation → ask it to run `date` on the home Mac.
- **E3 Reboot test (10–15 min)** — S11. `sudo shutdown -r now`. At the FileVault screen, on macOS 26 or later with Apple Silicon, unlock **over SSH from the laptop on home Wi-Fi**: `ssh <user>@<home-mac LAN IP>`, your password; the connection drops while it finishes, wait a minute. Then confirm what came back on its own (Tailscale, the Claude app, your services, the heartbeat) and note anything started by hand that didn't. Set `PREBOOT_TESTED`.
- **E4 Power-cut test (5 min)** — S1. Note `uptime`, cut the mains to your UPS/inverter for 2 minutes, restore it. `uptime` shouldn't reset, the router should stay online, and the phone on mobile data should still reach the Mac.
- **E5 Depart dry run (5 min).** `rwp depart --leave "test" --back <date>` without `--activate`; read `state/AWAY_SHEET.md`.

## Later (needs a purchase)
- **LAN foothold for pre-boot unlock from anywhere:** a Raspberry Pi (or any always-on Linux box) on backup power, running Tailscale as a subnet router for **only the home Mac's address** (`tailscale up --advertise-routes=<LAN IP>/32`, IP forwarding on, route approved in the admin console, "Use Tailscale subnets" on the laptop and phone). Then `ssh <user>@<LAN IP>` unlocks FileVault from the hotel. Set `LAN_FOOTHOLD`.
- **A hardware security key** (above).

## Status

| # | Item | Status |
|---|---|---|
| S1 | Survive a restart while away: power backup, automatic installs off, pre-boot way in | ❓ |
| S2 | Power settings: sleep 0, disksleep 0, autorestart 1, womp 1 | ❓ |
| S3 | Second device that can SSH in | ❓ |
| S4 | Tailnet hygiene: no stale nodes or expired keys; carry-device key expiries in your calendar | ❓ |
| S5 | Claude link: app running; relaunches after a restart (E3); pinned conversation works from the phone (E2) | ❓ |
| S6 | Fixed Wi-Fi address + DHCP reservation | ❓ |
| S7 | Wired network | ❓ |
| S8 | Outside heartbeat | ❓ |
| S9 | Which services must stay up while away (`config/services.tsv`) | ❓ |
| S10 | Home contact walked through `runbooks/home-contact.md` | ❓ |
| S11 | Reboot test (E3) | ❓ |
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
