# Runbook: the home Mac restarted. Ways back in, by kind of restart

Start on the laptop: `~/Projects/RWP/bin/rwp status`.
- **It answers** → R1 says when it restarted and which long-running jobs are gone; L1 fails if nobody is logged in (login window). Log in if needed (B below), then bring back what you need yourself.
- **"did not answer over SSH" + "<LAN IP>:22 answers"** → it's at the unlock screen or the login window: `rwp unlock` (C below) works out which.
- **Nothing answers** → `runbooks/home-mac-unreachable.md` section 2.

## A. Which restart, where it stops, what's stopping us (FileVault ON)

| What happened | Where the Mac stops | Way back in | From where, out of the box | What's stopping "from anywhere" |
|---|---|---|---|---|
| macOS update | logged in, by itself | nothing to do | anywhere | nothing (D15) |
| `rwp restart` (planned; `fdesetup authrestart`) | **the login window**: nobody logged in, so the Tailscale app, your assistant's app and your LaunchAgents wait | Screen Sharing to the login window, log in | home Wi-Fi only | the Tailscale Mac apps don't run before login → `runbooks/tailscaled.md` or `runbooks/foothold.md` |
| power out longer than the UPS lasts, a crash, a plain restart, a hard power-off | **the FileVault unlock screen** | `rwp unlock`: SSH at the unlock screen with your macOS password (macOS 26 or later, Apple silicon) | home Wi-Fi only | nothing on the tailnet reaches the house's network → `runbooks/foothold.md`; the LAN address must be fixed (SETUP 2) |
| hung (no restart) | wherever it hung | a power cycle, then row 3 | someone at home, at the power button | no remote power switch (D below) |

`rwp preflight` shows the "from anywhere" columns live: **W1** (planned restart) and **W2** (unlock screen). WARN = only from home.

## B. A planned restart: `rwp restart`

From the laptop:
```
~/Projects/RWP/bin/rwp restart --dry-run    # the way back in, and the jobs that will stop
~/Projects/RWP/bin/rwp restart              # sudo password, then fdesetup asks for your user name and password
```
It asks you to type `yes`, or `RESTART` when there's no way back in from outside. It skips the unlock screen once and stops at the login window ("goes straight to the regular login window", Der Flounder; SETUP E3a checks it on your Mac). Then:
- at home: `open vnc://<LAN IP>` (or the keyboard) and log in;
- with a foothold: the same, from anywhere;
- with Tailscale before login: `open vnc://<home-mac>` from anywhere.

Then `rwp status`: R1 marks the restart "planned", L1 passes once you're logged in.

`rwp restart --arm` restarts nothing: it makes the next restart triggered some other way (the Apple menu over Screen Sharing, an installer) skip the unlock screen once. FileVault protection is reduced until that restart; don't count on it for a crash or a power cut.

## C. At the unlock screen: `rwp unlock`

Needs all of: Apple silicon and macOS 26 or later, Remote Login on, the Mac on the network before unlock (Wi-Fi works from 26.5 per Jeff Geerling; Ethernet is steadier), its LAN address known and fixed (SETUP 2), the laptop on home Wi-Fi or a foothold. Apple's `man apple_ssh_and_filevault` describes it: password authentication only; SSH disconnects while the data volume mounts.

From the laptop: `~/Projects/RWP/bin/rwp unlock`. It:
1. stops if the Mac is already up on the tailnet;
2. checks `<LAN IP>:22` answers (no answer: away without a foothold, or the Mac is off);
3. tries your key over the LAN: the normal SSH server takes it, the unlock screen doesn't. Key accepted = already past the unlock screen, and it tells you whether that's the login window or Tailscale being down;
4. otherwise opens SSH with password only: log in with your macOS password. The connection closes when FileVault unlocks. The unlock screen's host key is kept separately in `state/known_hosts_preboot`, trusted on first use (do the first one at home: E3b);
5. waits up to 4 minutes for the tailnet, then runs `rwp status`; if the Mac lands at the login window instead, it says so: `open vnc://<LAN IP>` and log in.

By hand, the same thing: `ssh -o PubkeyAuthentication=no <user>@<LAN IP>`.

E3b answers what the docs don't: whether the unlock screen joins Wi-Fi with the same MAC (and so the same address) as the logged-in Mac, whether it lands on the desktop or the login window, and how long it takes.

## D. What each fix costs

| Fix | Covers | Cost | Where it's written up |
|---|---|---|---|
| Run E3a and E3b at home | turns "documented" into seen | 30 min; stops every running job on the Mac: you pick the time | SETUP E3 |
| Fixed LAN address (Private Wi-Fi address off + DHCP reservation, or Ethernet) | `rwp unlock` hits the right address | 10 min | SETUP 2 |
| A foothold at home on backup power: Apple TV, spare Android phone or Raspberry Pi as a one-address subnet router | the unlock screen **and** the login window, from anywhere | a device you may already own, or a small purchase | `runbooks/foothold.md` |
| Tailscale before login (open-source `tailscaled` instead of the app) | the login window from anywhere, without a foothold; not the unlock screen | 45 min at home, includes a restart; Tailscale calls it "for experienced macOS system administrators" | `runbooks/tailscaled.md` |
| A smart plug on the Mac, on backup power | a hung Mac: off 30 s, on; `autorestart 1` boots it, then C | a purchase; a hard power-off can corrupt whatever was being written, so last resort only | — |
| FileVault off + automatic login | every restart comes back by itself | a stolen Mac's disk is readable; D9 recommends keeping it ON | D9, your call |

Never: a stored password to automate the unlock. A FileVault password in a file is FileVault switched off with extra steps (D17).
