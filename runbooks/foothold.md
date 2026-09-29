# Runbook: a foothold at home (the unlock screen and the login window, from anywhere)

**What it is.** A small device that stays on at home, on the same network as the home Mac and on backup power, running Tailscale as a subnet router for **one address only: the home Mac's (`<LAN IP>/32`)**. Through it your laptop and phone reach that address from anywhere, even while the home Mac's own Tailscale isn't running.

**What it gives you.**
- `rwp unlock` from anywhere: power out longer than the UPS lasts, a crash, a plain restart (W2 turns PASS).
- Screen Sharing to the login window from anywhere (`open vnc://<LAN IP>`), so `rwp restart` has a way back (W1 turns PASS).
- A tell for "Mac down" vs "house offline": the foothold online in the admin console while the Mac isn't means it's the Mac.

**Before you start.** The home Mac's address must not move: SETUP 2 (Private Wi-Fi address off + DHCP reservation, or Ethernet). The foothold goes on backup power with the router and the modem; a foothold that dies with the power can't help after a power cut.

## Pick a device

| Device | Notes |
|---|---|
| Apple TV | Tailscale app → **Subnet Router** → **Advertise New Route** → `<LAN IP>/32`. A tvOS bug dropped advertised routes in Tailscale 1.90.4; fixed in 1.90.9 (Nov 2025). Test that it still routes while the TV is "asleep". |
| Spare Android phone | Tailscale app, subnet routes in its settings. Keep it plugged in, battery optimisation off for Tailscale. |
| Raspberry Pi (a Pi 4 with 2 GB is plenty) | The most predictable; Ethernet into the router. Steps below. |
| Router with Tailscale (GL.iNet, OpenWrt) | Fewest boxes; check your model first. |

Tailscale's guide for each platform: https://tailscale.com/kb/1019/subnets

## Set it up (Raspberry Pi; about 30 min)

On the Pi (Raspberry Pi OS Lite, Ethernet to the router):
```
curl -fsSL https://tailscale.com/install.sh | sh
echo 'net.ipv4.ip_forward = 1' | sudo tee -a /etc/sysctl.d/99-tailscale.conf
sudo sysctl -p /etc/sysctl.d/99-tailscale.conf
sudo tailscale up --hostname=home-foothold --advertise-routes=<LAN IP>/32
```
Log in with your tailnet's login. Then in the Tailscale admin console → Machines → `home-foothold`:
- **Edit route settings** → approve `<LAN IP>/32`;
- **Disable key expiry** (a foothold whose key expires is a foothold that's gone the day you need it).

Only the one address: the rest of the house's network stays off the tailnet.

## On the home Mac and the laptop
- **Home Mac:** Tailscale → **Use Tailscale subnets: off** (or `tailscale set --accept-routes=false`). It doesn't need a route to its own address through the foothold. A precaution; you run it.
- **Laptop and phone:** nothing. macOS, iOS and Android pick up approved subnet routes by default.
- At home the laptop may reach `<LAN IP>` through the foothold rather than directly (the /32 is more specific than the home network's route). Harmless; if the foothold misbehaves while you're home, switch "Use Tailscale subnets" off on the laptop for a while.

## Prove it (one evening)
1. On the home Mac: `rwp preflight` → **W1 PASS** and **W2 PASS** name the foothold.
2. Laptop on the phone hotspot (home Wi-Fi off): `nc -z -G 5 <LAN IP> 22 && echo reachable`, then `open vnc://<LAN IP>`.
3. When you can spare every running job: SETUP E3b through the foothold (plain restart at home, laptop on the hotspot, `rwp unlock`). Set `PREBOOT_TESTED` in `config/rwp.conf`.

## If it's lost or stolen
Admin console → Machines → `home-foothold` → Remove. It only ever routed to the home Mac, and the Mac still wants your password or key.
