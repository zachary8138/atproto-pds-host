# atproto-pds-host

Ubuntu helpers around the official AT Protocol Personal Data Server.

This package does not ship Bluesky's PDS container. It installs pds-host
for preflight checks, SSH/UFW hardening, and hostname file edits after you
have run the upstream installer.

Upstream:
- https://github.com/bluesky-social/pds
- https://atproto.com/guides/self-hosting
- https://atproto.com/guides/account-migration

Install the PDS first on a dedicated Ubuntu 24.04 host with ports 80 and 443 free:

  curl -fsSL https://raw.githubusercontent.com/bluesky-social/pds/main/installer.sh -o installer.sh
  sudo bash installer.sh

Use a hostname you control (for example pds.example.com) and a wildcard
*.pds.example.com. Do not point DNS at a Tailscale address. Keep 80/443
public so relays and Let's Encrypt can reach the host.

This package:

  sudo add-apt-repository ppa:rist138/atproto-pds-host
  sudo apt update
  sudo apt install atproto-pds-host
  sudo pds-host preflight
  sudo pds-host harden-ssh
  sudo pds-host ufw-ssh --from 100.64.0.0/10

Use a single Tailscale IP instead of 100.64.0.0/10 to allow only one
admin host. Pass --enable only after a second SSH session still works.

Hostname rename (same machine, after DNS for the new name exists):

  sudo pds-host apply-hostname pds.example.com pds2.example.com --restart

Then update each account handle and the PLC service endpoint to match
https://pds2.example.com. See the upstream migration guide. Keep the old
DNS records until that succeeds.

This is not a hosted PDS, not a public demo, and not a high-traffic
front end. It does not run the official installer for you.
