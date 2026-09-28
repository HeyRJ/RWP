# Runbook: working from someone else's device, and cleaning up after

Use it only for what the phone can't do. Every device you type into is a device that may have kept a copy.

## Trust tiers

| Tier | Example | Allowed |
|---|---|---|
| 1. Your own | laptop, phone | everything in the other runbooks |
| 2. A trusted person's | a family member's laptop | browser only, private/guest window, sanitise after |
| 3. Untrusted | cyber café, hotel business centre, a client's machine | avoid. If unavoidable: one browser session for one task, assume it was keylogged, rotate afterwards |
| Never | your work laptop | not for personal infrastructure: it's monitored, and personal remote access almost certainly breaks its usage policy |

## Setup: smallest footprint that works

1. Open a **guest profile or private window**. Don't sign into the browser itself, and don't let it save passwords.
2. Go to **claude.ai** → open the pinned conversation linked to the home Mac. It stays linked for as long as the home Mac's Claude app runs, so you can work on the Mac with nothing installed. (E2 in `SETUP.md` is the test that proves this; don't rely on it untested.)
3. Need the runbooks, STATE or the last trip log? Your private RWP repo on GitHub (current as of the last depart/return with `AUTO_PUSH=1`).
4. Need a real shell? Tailscale's browser SSH console needs Tailscale SSH on the home Mac, and the Tailscale Mac apps (App Store, Standalone) can't be a Tailscale SSH server; only the open-source `tailscaled` can. Otherwise it's your assistant, your phone's SSH app, or, on a Tier 2 device only, Tailscale installed with an **ephemeral** auth key (admin console → Settings → Keys → Generate auth key → Ephemeral) plus the built-in `ssh`.
5. Never type on a Tier 2/3 device: your password-manager master password, the home Mac's password, anything marked high-value in `inventory/CREDENTIALS_MAP.md`. Approve 2FA on your phone.

## Production without the laptop

If you run production somewhere else (a cloud platform), write its break-glass path here: which dashboard rolls back a deploy, which repo, who to tell. From a borrowed device: that dashboard and your code host's web UI, nothing else, and no database writes. Rotate whatever you signed into afterwards.

## After: sanitise before you hand it back

1. **Sign out** of everything you used (claude.ai, GitHub, your email, the Tailscale admin console…): click sign out; closing the tab isn't enough.
2. **Close the private/guest window.** If you used a normal profile: clear history, cookies, cached files, saved passwords and autofill for "all time", and remove any account you added to the browser.
3. **Delete downloads** (Downloads folder, Trash/Recycle Bin).
4. **Remove anything installed.** Tailscale: `tailscale logout`, uninstall, then delete the node in the admin console (ephemeral nodes vanish on their own; check anyway).
5. **From your phone, afterwards**: review active sessions and sign that device out (your email provider's device list, GitHub → Settings → Sessions, claude.ai, Tailscale → Machines: no strangers).
6. **Rotate** every password you typed there. Tier 3: all of them, same day.
7. **Log it** in the trip log: date, device, what you accessed, what you rotated. `rwp return` reminds you.
