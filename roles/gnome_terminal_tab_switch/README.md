gnome_terminal_tab_switch
=========

Bind `Ctrl+Tab` / `Ctrl+Shift+Tab` to next / previous tab on GUI terminals on
GNOME desktop, via a system-wide dconf override.

By default this role configures both:

- **GNOME Terminal** (`gnome-terminal`) — schema `org.gnome.Terminal.Legacy.Keybindings`
- **Ptyxis** (default terminal in Ubuntu 25.10 / 26.04) — schema `org.gnome.Ptyxis`

Override is harmless if a particular terminal is not installed.

Requirements
------------

- Ansible 2.9 or higher
- Target: Ubuntu desktop with GNOME (tested on Ubuntu 26.04 LTS / GNOME / Wayland)

Role Variables
--------------

All variables have sensible defaults — the role works without specifying any of
them.

| Variable | Default | Description |
|---|---|---|
| `gnome_terminal_tab_switch_apply_gnome_terminal` | `true` | Configure gnome-terminal keybindings |
| `gnome_terminal_tab_switch_apply_ptyxis` | `true` | Configure Ptyxis shortcuts |
| `gnome_terminal_tab_switch_next` | `<Primary>Tab` | Next-tab accelerator (GNOME format) |
| `gnome_terminal_tab_switch_prev` | `<Primary><Shift>Tab` | Previous-tab accelerator |

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: ubu26
  become: yes
  roles:
    - gnome_terminal_tab_switch
```

To use different keybindings (e.g. Alt+Tab):

```yaml
- hosts: ubu26
  become: yes
  roles:
    - role: gnome_terminal_tab_switch
      gnome_terminal_tab_switch_next: "<Alt>Tab"
      gnome_terminal_tab_switch_prev: "<Alt><Shift>Tab"
```

Tags
----

- `gnome_terminal_tab_switch`: Apply this tag to run only this role

Notes
-----

- A re-login (or terminal restart) is required for the new keybindings to take
  effect.
- Per-user `gsettings` values take precedence over the system-wide dconf default.
  If a user previously customized these keys, this override will not apply for
  them until they reset with `gsettings reset`.

License
-------

BSD-3-Clause
