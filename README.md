# RWP — Remote Working Protocol

You have a Mac at home that does real work: a Mac mini running models, long jobs, a server or two. You travel with a laptop. RWP is what you run the night before you leave, so everything you need is reachable from wherever you are, plus a written plan for the day it isn't.

It's a handful of markdown files and one read-only bash script, made to be run with Claude (any assistant that can run commands on your Mac will do). Same shape as [CCM](https://github.com/HeyRJ/CCM): plain files and one habit.

## The problem

Remote access that works on your home Wi-Fi tells you nothing about the hotel. What strands people is boring and predictable:

- **A restart while you're away.** With FileVault on, a power cut longer than your UPS lasts, a crash or a plain restart leaves the Mac at the unlock screen, and even a careful remote restart (`fdesetup authrestart`) stops at the login window: either way the Tailscale app isn't running, so from outside the house nothing reaches it. (An update you approve at the keyboard logs back in by itself; one installed with nobody there may not. Both kill everything you started by hand.) RWP has a command for each way back in and tells you which ones work from where you'll be.
- **Testing from home.** On your own network the local path hides a broken remote path. It works right up until you leave.
- **One way in.** Only the laptop's SSH key can log in. Lose the laptop and you're locked out of your own machine.
- **Silence.** Your alerts come from the machine that just died, so a dead Mac looks exactly like "all quiet".
- **Money meters.** A rented GPU box you forgot about, billing by the hour.
- **Other people's devices.** You borrow a laptop, log in, and leave your sessions behind.
- **Losing the laptop and the phone together**, with every 2FA code on the phone.

## What's here

```
README.md, INIT.md             this page; setup instructions for your assistant
CLAUDE.md, CLAUDE_RULES.md     entry point; hard rules + the AWAY rules every session follows while you're away
STATE.md                       HOME or AWAY and the current trip (the script writes the top block)
PROTOCOL.md                    depart → away → return, and what "ACTIVE" means
SETUP.md                       one-time setup in order, and the checks that prove it
DECISIONS.md                   why it's built this way, and what was rejected
bin/rwp                        preflight · depart · handoff · status · notify · return · heartbeat · restart · unlock
config/*.example               your names, addresses and services
runbooks/                      laptop · phone · borrowed device · lost device · home Mac unreachable ·
                               restart recovery · foothold · tailscaled · home contact
inventory/CREDENTIALS_MAP.md   what credential lives where, and how to revoke it (names only)
LOG/                           one file per setup session and per trip
```

## The three moments

| When | Say to your assistant | Or run on the home Mac |
|---|---|---|
| The night before you leave | "RWP depart: leaving <when>, back <date>, carrying <devices>" | `bin/rwp preflight --back <date>`, fix what fails, then `bin/rwp depart … --activate` |
| While away | "RWP status" | `bin/rwp status` (from the laptop it runs over SSH) |
| Back home | "RWP return" | `bin/rwp return` |

**ACTIVE** means the home Mac passed its hard checks *and* your laptop proved it can get in **from outside the house**: `bin/rwp handoff`, run on the laptop while it's on your phone's hotspot, within the last 24 hours. Anything less is NOT ACTIVE, or ACTIVE with the risks you accepted, each one named.

## Quick start

1. **Use this template → create a *private* repository.** Your copy will describe how to get into your machine. Never make it public.
2. Clone it on the home Mac, at `~/Projects/RWP`.
3. Give your assistant access to that Mac (Claude Code in the folder, or a Claude conversation linked to the Mac) and say: **`Read INIT.md and set this up for me.`**
4. It looks around, asks a few questions, fills `config/`, runs the first preflight and turns every failure into a line in `SETUP.md`.
5. Work through `SETUP.md` once. Clone your repo on the laptop too.

## What this touches

- **Checks are read-only**: `pmset`, `fdesetup`, `tailscale status`, `launchctl print`, `lsof`, `ps`, `git status`, `defaults read`. Nothing is restarted, reconfigured or killed; fixes are printed for you to run.
- It writes only inside its own folder (`state/`, `LOG/`, `STATE.md`).
- Two commands change your system, and you run both yourself: `rwp heartbeat install` (a LaunchAgent that pings an outside monitor every 5 minutes) and `rwp restart` (a planned restart that skips the FileVault unlock screen once, after you confirm).
- Optional `AUTO_PUSH=1` commits `STATE.md` and the trip log to your own private repo on depart and return, so the current trip is readable from any browser.

## Requirements

- The home Mac on macOS (written and tested on macOS 27, Apple Silicon). `rwp handoff` expects the laptop to be a Mac too; the runbooks work for any device.
- Tailscale on every device; Remote Login and Screen Sharing on the home Mac (the preflight checks both).
- `python3` (Homebrew or the Xcode Command Line Tools) and the stock bash.
- Optional: an ntfy server for phone pushes, a free healthchecks.io account for the outside heartbeat, the Claude desktop app on the home Mac.
