# Runbook: the home Mac doesn't answer, or it restarted

## 0. It restarted and came back
Run `rwp status`. Check R1 says when it restarted and lists every long-running job from the last check that is now gone. Nothing on disk is lost; what's lost is whatever was running (and anything under /private/tmp, which macOS clears at boot).
- Bring back what you need, over SSH, in tmux, with each project's own start commands.
- Claude Code sessions that were running in tmux resume with `cd ~/Projects/<project> && claude --continue` (the transcripts survive in `~/.claude/projects/`).
- Log it in the trip log: when, why (`softwareupdate --history` shows an update), what you restarted.

Which restarts come back on their own (FileVault ON); the ways back in are in `runbooks/restart-recovery.md`:

| Restart | Comes back to |
|---|---|
| macOS update | logged in, by itself |
| `rwp restart` (the way to restart it remotely; `fdesetup authrestart`) | the login window: log in by Screen Sharing (L1 fails until someone does) |
| power out longer than the UPS lasts, a crash, a plain restart | the unlock screen → `rwp unlock` (section 2, step 4) |

## 1. Which paths are down?

| Claude link | SSH | Screen Sharing | Tailscale admin console shows the Mac | Most likely |
|---|---|---|---|---|
| down | up | up | online | the Claude app quit or is updating → `ssh home open -a Claude` |
| up | down | down | online | Sharing services off → look through the Claude link; changing Sharing needs your go |
| down | down | down | online | the Mac is up but hung → Screen Sharing to `vnc://<LAN IP>` through a foothold if you have one; else the home contact (hold the power button 10 s, then press once) |
| down | down | down | "last seen …" | **the Mac or the home network is offline** → section 2 |

Also look at when your last push arrived (pushes stop when the Mac does) and at the outside heartbeat's history (SETUP 4).

## 2. Offline: find out why, cheapest check first

1. **Power cut?** Ask the home contact whether there's power and whether the UPS/inverter is on (`runbooks/home-contact.md`).
2. **Internet down?** Ask whether the router's internet light is on. If not: router and modem off at the wall, 30 s, on, wait 3 minutes.
3. **Mac off?** Its light is off → press the power button once (on a 2024-or-later Mac mini it's underneath, at the back).
4. **Mac restarted and sitting at the FileVault unlock screen, or at the login window?** The unlock screen is the default outcome of any restart that isn't an update or `rwp restart`.
   - From the laptop, at home or through a foothold (`runbooks/foothold.md`): `~/Projects/RWP/bin/rwp unlock`. It tells the two apart, takes your password at the unlock screen, waits for the tailnet, and tells you when the login window needs Screen Sharing (`runbooks/restart-recovery.md` C).
   - Away without a foothold: it stays offline until someone unlocks it at the keyboard. Whether anyone at home gets that password is your call; the protocol doesn't assume it.
5. **Tailscale logged out, or its key expired?** Only fixable through a foothold (Screen Sharing to `vnc://<LAN IP>`) or at home. H1 checks the home Mac's key expiry is disabled, so this should not happen.

## 3. While it's down: stop the damage

- **Rented boxes keep billing** when the job that fed them or pulled from them is dead. Check the away sheet (the laptop caches it in `state/server_AWAY_SHEET.md`) and decide box by box in the provider console.
- Jobs that were running are gone if the Mac restarted, unless launchd brings them back. Note which ones in the trip log.

## 4. After it's back

`rwp status`, then check each project's own state before restarting anything. Log what happened and how long it was down.
