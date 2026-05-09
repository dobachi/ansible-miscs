caddy_for_headscale
=========

Install [Caddy](https://caddyserver.com/) from the upstream Cloudsmith apt repository and configure it as the TLS reverse proxy in front of a local Headscale instance. TLS certificates are obtained via Let's Encrypt automatically (HTTP-01).

Companion role to `headscale_server`. Run them together on the same host.

Why a dedicated role
--------------------

Headscale's protocol uses WebSocket POST, which means **Cloudflare and Cloudflare Tunnel cannot be used in front of it**. A direct DNS A record + Caddy is the simplest setup that just works, and Caddy's automatic TLS removes the cert renewal cron. This role bakes in those constraints.

Requirements
------------

- Ansible 2.16 or higher
- Ports 80/tcp and 443/tcp reachable from the public internet (for the ACME HTTP-01 challenge)
- A DNS A/AAAA record for the host portion of `headscale_server_url`

Role Variables
--------------

| Variable | Default | Description |
|---|---|---|
| `caddy_admin_email` | (required) | E-mail used by Caddy for Let's Encrypt registration / expiry notifications |
| `headscale_server_url` | (required) | Read from the headscale_server role's vars; provides the hostname to terminate TLS for |
| `caddy_for_headscale_upstream` | `{{ headscale_listen_addr | default('127.0.0.1:8080') }}` | Where Caddy forwards to. Defaults to whatever Headscale binds to |

Dependencies
------------

None. In practice you'll list this role after `headscale_server` so the variables are already in scope.

Example Playbook
----------------

```yaml
- hosts: headscale
  become: true
  vars:
    headscale_server_url: "https://hs.example.com"
    headscale_base_domain: "hs-net.example.com"
    caddy_admin_email: "you@example.com"
  roles:
    - headscale_server
    - caddy_for_headscale
```

License
-------

BSD-3-Clause
