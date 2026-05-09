headscale_server
=========

Install and configure [Headscale](https://github.com/juanfont/headscale), the OSS Tailscale-compatible coordination server, on a Debian/Ubuntu host. The role installs the upstream `.deb`, deploys `/etc/headscale/config.yaml` from a template, seeds an initial ACL policy, runs `headscale configtest`, enables the systemd unit, and (optionally) creates initial users.

This role does **not** terminate TLS — pair it with the companion `caddy_for_headscale` role (or any HTTPS reverse proxy you prefer). Headscale must be reached over TLS for Tailscale clients to connect.

Requirements
------------

- Ansible 2.16 or higher (Ansible 2.10's `module_utils.six.moves` is incompatible with Python 3.12 targets)
- Debian-based target: Ubuntu 24.04 / 26.04 (amd64 or arm64) or Raspberry Pi OS (64-bit)
- A real DNS A/AAAA record pointing at the host (`headscale_server_url`'s host)

Role Variables
--------------

Required:

| Variable | Example | Description |
|---|---|---|
| `headscale_server_url` | `https://hs.example.com` | Public URL clients connect to (must terminate TLS, i.e. https) |
| `headscale_base_domain` | `hs-net.example.com` | MagicDNS base. **Must differ from the `server_url` domain** |

Optional:

| Variable | Default | Description |
|---|---|---|
| `headscale_version` | `0.28.0` | Upstream version (no leading `v`) |
| `headscale_arch` | auto (`amd64` / `arm64`) | Override only for cross-arch builds |
| `headscale_listen_addr` | `127.0.0.1:8080` | Bind address; loopback assumes Caddy in front |
| `headscale_v4_prefix` / `headscale_v6_prefix` | CGNAT defaults | Tailnet IP allocation |
| `headscale_global_nameservers` | `[1.1.1.1, 9.9.9.9]` | DNS offered to clients via MagicDNS |
| `headscale_users` | `[]` | Initial users to create (idempotent) |
| `headscale_acl_enable_ssh` | `true` | Seed ACL with a Tailscale SSH allow rule |
| `headscale_acl_ssh_users` | `[root, autogroup:nonroot]` | Local users that Tailscale SSH may impersonate |
| `headscale_derp_server_enabled` | `false` | Run an embedded DERP server on this host |

Dependencies
------------

None. Pair with `caddy_for_headscale` for TLS termination on the same host.

Example Playbook
----------------

See `playbooks/conf/linux/headscale_server.yml`. Minimal form:

```yaml
- hosts: headscale
  become: true
  vars:
    headscale_server_url: "https://hs.example.com"
    headscale_base_domain: "hs-net.example.com"
    caddy_admin_email: "you@example.com"
    headscale_users:
      - alice
  roles:
    - headscale_server
    - caddy_for_headscale
```

Day-2 operations
----------------

- Logs: `journalctl -u headscale -f`
- Backups: `/etc/headscale/`, `/var/lib/headscale/db.sqlite`, `/var/lib/headscale/noise_private.key` (loss = re-register every client)
- Pre-auth keys: `headscale preauthkeys create -u <user> --reusable --expiration 24h`
- ACL test after edits: `headscale policy check --file /etc/headscale/acl.hujson`

License
-------

BSD-3-Clause
