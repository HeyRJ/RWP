# RWP — INDEX

- `README.md` — what RWP is, the three moments, quick start.
- `INIT.md` — setup instructions for your assistant (first time only).
- `CLAUDE.md` — entry point; boot order.
- `CLAUDE_RULES.md` — hard rules + the AWAY rules (A1–A7) every session follows while you're away.
- `STATE.md` — HOME / AWAY and the current trip (top block written by the script), known problems.
- `PROTOCOL.md` — depart (declare → preflight → blocking → activate), away, return; what ACTIVE does and doesn't cover.
- `SETUP.md` — first-time walkthrough (1–6), checks only you can do (E1–E5), later purchases, status table.
- `DECISIONS.md` — why it's built this way, with what was rejected.
- `bin/rwp` — the script: `preflight | depart | handoff | status | notify | return | heartbeat | restart | unlock`. Checks are read-only; `heartbeat install/remove` and `restart` change the system, and you run them.
- `config/rwp.conf.example` → `config/rwp.conf` — everything personal: names, addresses, labels, thresholds. No secrets.
- `config/services.tsv.example` → `config/services.tsv` — what must stay up while you're away (required / watch / info).
- `runbooks/laptop.md` — the laptop as command deck; SSH / tmux / Screen Sharing fallback.
- `runbooks/phone.md` — phone only.
- `runbooks/borrowed-device.md` — trust tiers, minimum-footprint setup, sanitising afterwards.
- `runbooks/lost-device.md` — laptop / phone / both lost: contain, revoke, continue.
- `runbooks/home-mac-unreachable.md` — which path is down, why, what to stop while it's down.
- `runbooks/restart-recovery.md` — after a restart: where each kind stops, the way back in (`rwp restart`, `rwp unlock`), what's stopping "from anywhere", what each fix costs.
- `runbooks/foothold.md` — a one-address subnet router at home: the unlock screen and the login window from anywhere.
- `runbooks/tailscaled.md` — the home Mac on open-source `tailscaled`, so it's on the tailnet at the login window. At home only.
- `runbooks/home-contact.md` — printable page for the person at home. No passwords.
- `inventory/CREDENTIALS_MAP.md` — what credential lives where and how to revoke it. Names only.
- `LOG/` — `RWP-S<N>.md` setup sessions, `RWP-T<N>.md` trips (opened by `depart --activate`).
- `state/` — not in git: check records, handoff receipts, `AWAY_SHEET.md`, caches.

Keep this current. An index that lies is worse than none.
