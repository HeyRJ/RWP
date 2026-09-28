# Runbook: the laptop as command deck (primary) and remote login (fallback)

## One-time setup (SETUP 6)

`~/.ssh/config` on the laptop:

```
Host home
  HostName <tailscale-name>        # MagicDNS; or the full <tailscale-name>.<tailnet>.ts.net, or its 100.x IP
  User <macos-username>
  ServerAliveInterval 30
  ServerAliveCountMax 4
```

If ssh offers the wrong key, add `IdentityFile ~/.ssh/<key>` and `IdentitiesOnly yes`.

Clone your private RWP repo to `~/Projects/RWP` on the laptop. `rwp handoff` pulls updates for you.

## Before you leave

On a phone hotspot, not home Wi-Fi: `~/Projects/RWP/bin/rwp handoff`. It checks this laptop's Tailscale, the route to the home Mac (and whether you're secretly still on the home network), SSH, Screen Sharing and this laptop's FileVault; copies the home Mac's STATE and away sheet here (`state/server_*.md`); pulls the latest RWP docs; and leaves the receipt that `depart --activate` needs.

## Working while away

- **Your assistant** is the command deck. Anything that has to touch the home Mac runs in a session linked to it; the pinned conversation already is.
- **Status**: `~/Projects/RWP/bin/rwp status`. Runs on the home Mac over SSH. If it doesn't answer, you get the last cached status and a pointer to `home-mac-unreachable.md`.
- **Shell**: `ssh home`. Long work always inside tmux: `ssh -t home tmux new -A -s rwp` (attach, or create). Detach with Ctrl-b d; the work keeps running.
- **GUI**: Screen Sharing, `open vnc://<tailscale-name>`. If it's sluggish, switch to High Performance.
- **Files**: `rsync -avP home:Projects/<project>/<path> ./` and back. Repos with a remote: push/pull as usual. Local-only repos: work on the home Mac over SSH.
- **Bad networks**: sign into the captive portal first, then Tailscale. If direct connections are blocked, Tailscale relays (DERP) on its own: slower, still works. Your phone's hotspot is the fallback network.

## When a path fails

| Symptom | Likely cause | Do |
|---|---|---|
| The assistant can't reach the home Mac, SSH works | the Claude app quit or is updating | `ssh home open -a Claude`, wait 30 s, retry |
| SSH and Screen Sharing time out; the admin console shows the Mac online | Sharing services off | look through the Claude link; changing Sharing needs your go |
| Everything fails; the admin console shows "last seen …" | the Mac or the home network is down | `runbooks/home-mac-unreachable.md` |
| This laptop's Tailscale says logged out | its key expired | log in again (needs your identity provider + 2FA) |
