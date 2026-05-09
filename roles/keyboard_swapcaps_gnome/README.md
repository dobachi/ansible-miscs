keyboard_swapcaps_gnome
=========

Swap Caps Lock and Ctrl on Ubuntu desktop (GNOME / Wayland) and console.

This role configures the swap at two layers:

1. **dconf system-wide override** (`/etc/dconf/db/local.d/00-keyboard-swapcaps`) — applies to GNOME sessions on next login. Works on Wayland because GNOME's `xkb-options` setting is honored by Mutter.
2. **`/etc/default/keyboard`** — applies to TTY/console and the GDM login screen via `setupcon`.

This is the GNOME/Wayland-friendly counterpart to `keyboard_nocaps`, which directly edits `/usr/share/X11/xkb/symbols/{jp,us}` and is intended for Raspberry Pi OS (X11 era) usage.

Requirements
------------

- Ansible 2.9 or higher
- Target: Ubuntu desktop with GNOME (tested on Ubuntu 26.04 LTS / GNOME 50 / Wayland)

Role Variables
--------------

| Variable | Default | Description |
|---|---|---|
| `keyboard_swapcaps_xkb_option` | `ctrl:swapcaps` | XKB option string. Other useful values: `ctrl:nocaps`, `caps:ctrl_modifier` |
| `keyboard_swapcaps_apply_dconf` | `true` | Apply system-wide GNOME default via dconf |
| `keyboard_swapcaps_apply_console` | `true` | Apply via `/etc/default/keyboard` (console + login screen) |

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: ubu26
  become: yes
  roles:
    - keyboard_swapcaps_gnome
```

To use a different XKB option:

```yaml
- hosts: ubu26
  become: yes
  roles:
    - role: keyboard_swapcaps_gnome
      keyboard_swapcaps_xkb_option: "caps:ctrl_modifier"
```

Tags
----

- `keyboard_swapcaps_gnome`: Apply this tag to run only this role

Notes
-----

- A re-login is required for the GNOME session to pick up the new `xkb-options`.
- Per-user override via `gsettings set org.gnome.desktop.input-sources xkb-options "['ctrl:swapcaps']"` will take precedence over the dconf system default for that user.

License
-------

BSD-3-Clause
