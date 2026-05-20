tailscale_client
=========

Install the [Tailscale](https://tailscale.com/) client from the upstream apt repo, enable `tailscaled`, and (optionally) register the node against a coordination server using a pre-auth key. Designed primarily for self-hosted Headscale, but works unchanged against the Tailscale SaaS controlplane.

Requirements
------------

- Ansible 2.16 or higher
- Debian-based target (Ubuntu 22.04 / 24.04 / 26.04, Debian 12 / 13, Raspberry Pi OS 64-bit)
- For automatic registration: a pre-auth key from the coordination server, e.g.
  - `headscale preauthkeys create -u <user> --reusable --expiration 24h`

Role Variables
--------------

| Variable | Default | Description |
|---|---|---|
| `tailscale_use_upstream_repo` | `true` | Use pkgs.tailscale.com apt repo (recommended; distro packages lag) |
| `tailscale_distro_id` / `tailscale_distro_codename` | auto | Override only on derivative distros |
| `tailscale_login_server` | `""` | Coordination server URL. Empty -> Tailscale SaaS controlplane |
| `tailscale_auth_key` | `""` | Pre-auth key. Empty -> install only, log in by hand later |
| `tailscale_accept_dns` | `true` | `--accept-dns` |
| `tailscale_accept_routes` | `true` | `--accept-routes` |
| `tailscale_ssh` | `true` | `--ssh` (Tailscale SSH; pair with `headscale_acl_enable_ssh`) |
| `tailscale_advertise_exit_node` | `false` | `--advertise-exit-node` |
| `tailscale_advertise_routes` | `[]` | List of CIDRs for `--advertise-routes` |
| `tailscale_hostname` | `""` | `--hostname=<value>` (default: machine hostname) |
| `tailscale_extra_up_args` | `[]` | Raw flags appended verbatim |

Dependencies
------------

None.

Example Playbook
----------------

Minimal install (manual login later):

```yaml
- hosts: tailscale_clients
  become: true
  roles:
    - tailscale_client
```

Auto-register against self-hosted Headscale, with Tailscale SSH enabled:

```yaml
- hosts: tailscale_clients
  become: true
  vars:
    tailscale_login_server: "https://hs.example.com"
    tailscale_auth_key: "{{ vault_tailscale_auth_key }}"
    tailscale_ssh: true
  roles:
    - tailscale_client
```

Subnet router (advertise the home LAN to the tailnet):

```yaml
- hosts: home_router
  become: true
  vars:
    tailscale_login_server: "https://hs.example.com"
    tailscale_auth_key: "{{ vault_tailscale_auth_key }}"
    tailscale_advertise_routes:
      - 192.168.10.0/24
  roles:
    - tailscale_client
```

After provisioning, accept the routes on the headscale side:

```bash
headscale routes list
headscale routes enable -r <route-id>
```

Tags
----

- `tailscale_client`: full role
- `tailscale_up`: only the registration step (handy for re-keying)
- `tailscale_set`: only the flag-reconcile step (push changes to `tailscale_ssh`, `tailscale_accept_dns`, `tailscale_accept_routes`, `tailscale_advertise_exit_node`, `tailscale_advertise_routes`, `tailscale_hostname` on an already-up node, without needing an auth key)

Reconcile vs. register
----------------------

`tailscale up` runs **only** when the node is not yet Running (or `tailscale_up_force=true`), because re-running it with a single-use auth key would fail. To still push flag changes (e.g. flipping `tailscale_ssh` on) onto a node that was already up before the change, the role also runs `tailscale set` on every play when the backend is Running. `tailscale set` is idempotent and does not require an auth key.

Security note
-------------

`tailscale_auth_key` should be supplied via Ansible Vault or `-e @vault.yml`, **not** committed in plain text. The `tailscale up` task uses `no_log: true` to keep the key out of stdout/log files.

License
-------

BSD-3-Clause
