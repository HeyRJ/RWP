# Credentials map: names and locations only, NEVER values

What lives where, and how to cut it off. `runbooks/lost-device.md` walks this table. Replace the example rows with yours.

| Credential | Lives on | Used for | Revoke / rotate at |
|---|---|---|---|
| SSH key `<laptop-key-comment>` | laptop `~/.ssh` | SSH into the home Mac | home Mac `~/.ssh/authorized_keys` (delete the line) |
| SSH key `<phone-key-comment>` | phone SSH app | SSH into the home Mac | home Mac `~/.ssh/authorized_keys` |
| SSH keys for rented servers | home Mac `~/.ssh` | rented GPU boxes | provider console → SSH keys |
| Code host login + CLI token | home Mac, laptop | git, CLI | code host → settings → sessions, tokens, SSH keys |
| Tailscale identity (GitHub / Google / Microsoft…) | every device | the tailnet | the identity provider's account; admin console → Machines |
| Home Mac login password | your head; saved in the laptop keychain for Screen Sharing? | login, FileVault unlock, Screen Sharing | change it on the home Mac |
| Assistant account (Claude) | every device | sessions, the Claude link | its settings |
| Email account | … | recovery for everything else | provider → security → devices |
| Apple ID | every Apple device | Find My, iCloud | appleid.apple.com |
| Project API keys (`.env` files) | which machine? | … | each provider's console; production first |
| Password manager (which?) | … | everything else | … |
| 2FA: authenticator app, backup codes, hardware key | phone; backup codes where? | every account above | per account |

**The break-glass question (SETUP 5):** with the laptop *and* the phone gone, which row gets you back in first? Write the answer here, not the secret.
