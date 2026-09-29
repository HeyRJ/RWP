# Runbook: Tailscale that runs before login (the home Mac on open-source `tailscaled`)

**Why.** The Tailscale Mac apps (Standalone and App Store) start only after someone logs in. Tailscale's own comparison: "Run before login — Standalone: no; tailscaled: yes". So after `rwp restart` (which stops at the login window) the home Mac drops off the tailnet until someone logs in. With `tailscaled` running as a system daemon it's back on the tailnet at the login window, and you log in by Screen Sharing from anywhere: `open vnc://<home-mac>`.

**What it doesn't fix.** The FileVault unlock screen. Nothing of yours runs until the disk is unlocked; that case needs a foothold (`runbooks/foothold.md`).

**What it costs.**
- Tailscale recommends `tailscaled` on macOS "only for unattended installs managed by experienced macOS system administrators": it's less tested than the apps.
- No menu-bar app and no automatic updates: `brew upgrade tailscale` by hand, at home.
- It doesn't set the Mac's DNS: the home Mac stops resolving tailnet names (`*.ts.net`) unless you add `100.100.100.100` as a DNS server. Other devices still find it by name. Check first whether anything on it uses tailnet names.
- The Mac becomes a new machine in the tailnet. Move the name and the address over (step 5) so nothing that points at the old ones breaks.
- **Do it at home only**, at the keyboard or over Screen Sharing on the LAN: you're replacing the thing you reach the Mac through. About 45 minutes, including a restart that stops every running job. You pick the time.

## Steps (you run them)

1. **Note what's there:** `tailscale status` (its name and 100.x address).
2. **Install the daemon, don't start it yet:** `brew install --formula tailscale` (the formula, not the app cask).
3. **Remove the app:** Tailscale menu → Quit; move `/Applications/Tailscale.app` to the Bin and approve removing its network extension; empty the Bin; restart. (Tailscale's advice when switching variants: delete, empty the Bin, restart, then install the other.)
4. **Start the daemon and log in:**
   ```
   sudo brew services start tailscale      # a LaunchDaemon: starts at boot, before login
   sudo tailscale up --hostname=<home-mac>
   ```
   Log in in the browser it opens.
5. **Admin console → Machines:** remove the old machine (now offline). On the new one, from its ⋯ menu: rename it if it came up as `<home-mac>-1`; change its Tailscale IPv4 address to the old one (free once the old machine is removed); **Disable key expiry**.
6. **Check:** `~/Projects/RWP/bin/rwp preflight` → H1 PASS, **W1 PASS "Tailscale runs before login"**. From the laptop: `ssh <user>@<home-mac> true` and `open vnc://<home-mac>`.
7. **Prove it (SETUP E3a):** `rwp restart`, then from the laptop on the phone hotspot, after about two minutes: `open vnc://<home-mac>` → the login window → log in → `rwp status`.

Leave Tailscale SSH (`--ssh`) off. It replaces macOS's key check with the tailnet policy, and the default policy asks for a browser login every 12 hours, which would break SSH from the phone and from scripts.

## Roll back
```
sudo brew services stop tailscale
brew uninstall tailscale
```
Then install the Standalone app from tailscale.com/download, log in, and fix the name and address in the admin console the same way.
