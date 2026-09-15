Ubuntu notes for a self-hosted PDS

Use the official installer from https://github.com/bluesky-social/pds
This package only adds pds-host helpers.

Keep TCP 80 and 443 on the public internet. Relays, AppView, and Let's Encrypt
need them. Do not put a browser challenge (for example Anubis) in front of
/xrpc or WebSockets.

Do not point the PDS hostname at a Tailscale 100.x address.

Change PDS_HOSTNAME in /pds/pds.env and the matching names in
/pds/caddy/etc/caddy/Caddyfile together (pds-host apply-hostname). A shell
variable is not enough. After a rename, update account handles and PLC
serviceEndpoint before deleting old DNS.

Some VPS providers block outbound SMTP on 25/465/587. Use a transactional
mail API on an allowed port. Test sending to your own address first.

Back up /pds off-box. It holds repository data, blobs, and rotation keys.
