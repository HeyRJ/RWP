# RWP — DECISIONS

Why it's built this way. Add your own below, numbered, with the reason and what you rejected.

- **D1** RWP is its own folder and repo, global across every project on the home Mac. It *reads* other projects (`ps`, `lsof`, `launchctl print`, `git status`) and never changes them; each project keeps its own rules.
- **D2** Roles while away: the laptop is the command deck; remote login to the home Mac (SSH / tmux / Screen Sharing) is the fallback; the phone is the last device you own; anyone else's device is browser only, then sanitised. Rejected: the phone as primary (fine for checks, not for work).
- **D3** ACTIVE needs proof from outside the house: a handoff receipt from the laptop, off the home network, under 24 h old. A test on home Wi-Fi proves nothing, because the local path hides a broken remote path. Rejected: "the home Mac's own checks are enough"; it can't see what the laptop sees.
- **D4** Checks are read-only. The script prints the fix (usually a sudo command or a System Settings path) and you run it. Rejected: auto-fixing power, update or Sharing settings; a wrong remote change to networking or power can't be undone from a hotel.
- **D5** No daemons. RWP is a script you run, not a service. Rejected: an always-on watcher on the home Mac; it dies with the Mac, which is the one case it would exist for. The one periodic job is the outside heartbeat (D8), and the thing that watches it lives outside the house.
- **D6** Your copy is private, always. It describes how to get into your machine. This template holds no one's details.
- **D7** How every session learns you're away: one line in your assistant's global preferences (SETUP S21). Rejected: a global session-start hook, if anything on the home Mac runs the assistant headless (cron jobs, pipelines): they'd get RWP text injected into their prompts.
- **D8** `rwp heartbeat install | remove` is the only command that changes the system (a LaunchAgent that pings an outside monitor every 5 minutes). You run it.
- **D9** Keep FileVault on. The restarts that lock you out are power out longer than the UPS lasts, crashes and plain restarts; cover them with backup power, `fdesetup authrestart` for restarts you trigger, and a pre-boot way in if you need one. Turning FileVault off swaps a travel inconvenience for a theft exposure that never expires.
- **D10** While AWAY, every command your assistant hands you is in remote form: runnable from the laptop, short enough for a phone.
- **D11** Credentials appear here by name and location only, never values.
- **D12** `state/` stays out of git (check records, receipts, the away sheet, the heartbeat URL). `STATE.md` carries the summary.
- **D13** Optional `AUTO_PUSH=1`: depart and return commit `STATE.md` + the trip log to your private remote, so the current trip is readable from a browser when the Mac is dark. Only those two files; a failed push is reported, never fatal.
- **D14** One script, no personal strings in it. Everything personal lives in `config/rwp.conf`.
- **D15** An update restart is not a lockout: macOS stashes your login before it reboots and logs you back in after (seen on a Mac mini on macOS 27, and documented by Eclectic Light for updates in general). It does kill every job started by hand, so automatic installs stay a warning (H8), not a blocker.
- **D16** Check R1: every run snapshots the long-running jobs and compares them with the previous snapshot; a boot time newer than the last check means RESTARTED, and each job that's gone is listed. So "did we lose anything?" is one `rwp status` from the phone. It only knows jobs that were running at the last check and older than 30 minutes.
