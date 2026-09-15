# atproto-pds-host

Ubuntu helpers around the [official AT Protocol Personal Data Server](https://github.com/bluesky-social/pds).

[![Ubuntu 24.04](https://img.shields.io/badge/Ubuntu-24.04%20LTS-E95420)](https://ubuntu.com/)
[![PPA](https://img.shields.io/badge/PPA-rist138%2Fatproto--pds--host-orange)](https://launchpad.net/~rist138/+archive/ubuntu/atproto-pds-host)
[![License: AGPL-3.0-or-later](https://img.shields.io/badge/License-AGPL--3.0--or--later-blue.svg)](LICENSE)

This package does **not** ship Bluesky’s PDS container and does **not** run the upstream installer. After you install the official stack, `pds-host` covers preflight checks, SSH/UFW hardening, health, and hostname file edits.

A working PDS is mostly work you do **outside** this package: a dedicated host, public DNS, the official installer, mail, and accounts. This README lists that work so the helpers are not mistaken for a full setup.

## What this is not

- Not a hosted PDS or a public demo
- Not a high-traffic front end or an AppView/relay
- Not a replacement for [self-hosting](https://atproto.com/guides/self-hosting) or [account migration](https://atproto.com/guides/account-migration)

See also [docs/ubuntu-pds.md](docs/ubuntu-pds.md).

## What you must provide yourself

Nothing below is installed or configured by `atproto-pds-host`. Have it ready before you expect federation, TLS, or Bluesky login to work.

### Host

- A **dedicated** Ubuntu 24.04 (Noble) machine. Do not share ports 80/443 with an existing website.
- Public **IPv4**. Optional **IPv6** only if the host actually answers on it (a dangling AAAA record breaks some clients).
- **TCP 80 and 443** free on the host and allowed from the public internet (relays, AppView, Let’s Encrypt).
- Outbound HTTPS. Many VPS providers also **block outbound SMTP** on 25/465/587.

### DNS (before the installer)

Use a hostname you control, for example `pds.example.com`, plus a wildcard for handles:

| Record | Name | Target |
| --- | --- | --- |
| `A` | `pds.example.com` | the VPS public IPv4 |
| `A` | `*.pds.example.com` | the same IPv4 |
| `AAAA` (optional) | both names | the VPS public IPv6 |

Do **not** point these names at a Tailscale `100.x` address. Tailscale is fine for **SSH**; it is not the PDS hostname.

Wait until `dig +short pds.example.com` (from another network) returns the public IP before running the installer, or certificate issuance will fail.

### Mail

The PDS needs working mail for invites, confirmations, and PLC/2FA-style flows.

- Use a real mailbox you control.
- If the VPS cannot send on 25/465/587, configure a transactional mail API on a port the provider allows, in `/pds/pds.env`, **after** the installer. Test a message to yourself before you migrate an account.

### Accounts and identity

- **New users:** create invite codes on the PDS and sign up with handles under `*.pds.example.com`.
- **Moving an existing Bluesky account:** follow the [upstream migration guide](https://atproto.com/guides/account-migration). That is `goat` / PLC work, not this package. Use the account password (not an app password). Tokens in email expire quickly.
- After any hostname change, update **each** account handle and the PLC `serviceEndpoint`. Keep the **old** DNS records until that succeeds.

### What not to put in front of the PDS

Do not put a browser challenge (for example Anubis) in front of `/xrpc` or WebSockets. Federation is not a browser.

### Backups

Copy `/pds` off-box. It holds repositories, blobs, and rotation keys. Losing it loses the PDS.

## Suggested order

1. Create the VPS. Confirm nothing is bound to 80/443.
2. Publish DNS `A` (and `AAAA` only if IPv6 works) for the hostname and wildcard.
3. Run the **official** installer (next section). Confirm `https://pds.example.com/xrpc/_health`.
4. Configure and test mail if the installer did not get a working SMTP path.
5. Create a user **or** migrate an account. Confirm login in the Bluesky app with that handle.
6. Optionally install this package and harden SSH/UFW.
7. Only then lock SSH to Tailscale (keep a second session open; pass `--enable` after it still works).

Skip step 6 and the PDS can still be fully functional. Skip DNS, 80/443, or mail, and it will not.

## Install the PDS

```bash
curl -fsSL https://raw.githubusercontent.com/bluesky-social/pds/main/installer.sh -o installer.sh
sudo bash installer.sh
