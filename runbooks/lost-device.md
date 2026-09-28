# Runbook: a device is lost or stolen

Order: **contain → revoke → continue.** Same day. Use whatever you still have: phone, laptop, or a borrowed browser (then sanitise it).

## Laptop lost or stolen

1. **Contain.** Find My at icloud.com/find, from any browser → Mark As Lost; Erase if it isn't coming back. It needs only your Apple ID password: at the 2FA step, the "Find Devices" button skips the code. FileVault is what protects the disk in the meantime.
2. **Cut its network path.** Tailscale admin console → Machines → the laptop → Remove. You log in to Tailscale with your identity provider (GitHub, Google, Microsoft…), so this needs that account and a 2FA method that isn't on the lost device (SETUP 5).
3. **Cut its SSH path.** On the home Mac (Claude link or phone SSH): `cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.bak-$(date +%F)`, then delete the laptop's key line.
4. **Revoke what lived on it.** Walk `inventory/CREDENTIALS_MAP.md`, laptop rows first: code-host keys and tokens, API keys in project `.env` files (production first), assistant and email sessions. If the home Mac's password was saved in the laptop's keychain for Screen Sharing, change that password.
5. **Continue** on the phone for checks and a Tier 2 device for real work (`borrowed-device.md`).
6. **Log it** in the trip log.

## Phone lost or stolen

1. Find My → Lost Mode / Erase. Ask your operator to block the SIM and issue a replacement with the same number. In some countries (India, for one) incoming SMS, OTPs included, stay blocked for 24 h after a SIM replacement, so plan on app or backup-code 2FA for that first day.
2. Tailscale admin → remove the phone. Remove its key from the home Mac's `authorized_keys`.
3. 2FA: switch to backup codes or a hardware key; re-enrol the authenticator on the replacement phone.
4. Sign the phone out of your email, code host, assistant and provider consoles.

## Both lost

Everything above, from a borrowed device. That only works if you can reach your password manager and get past 2FA without either device (SETUP 5). If that path doesn't exist, there is no remote plan, which is why SETUP 5 comes before a long trip.
