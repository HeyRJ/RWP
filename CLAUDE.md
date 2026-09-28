# RWP — Remote Working Protocol

First time here? Read `INIT.md` and set it up with the user.

Boot order for every session, before doing anything else: read `CLAUDE_RULES.md`, then `STATE.md`, then `PROTOCOL.md`, then the latest `LOG/RWP-*.md`.

The entry point is `bin/rwp` (`preflight | depart | handoff | status | notify | return | heartbeat`). "RWP depart / status / return" from the user means: run that part of `PROTOCOL.md`, in order, and report ACTIVE / NOT ACTIVE with the evidence (the script's PASS/FAIL lines), never from memory.

Follow `CLAUDE_RULES.md`. Keep the ledger current: `STATE.md`, `DECISIONS.md`, `SETUP.md`, and a `LOG` entry per setup session and per trip.
