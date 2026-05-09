fcitx5_mozc
=========

Install fcitx5 + fcitx5-mozc on Ubuntu desktop and pin **Ctrl + Space** as the trigger key for IME on/off.

Targeted at Ubuntu 26.04 LTS (GNOME 50, Wayland-only). The existing `japanese` role in this repo targets WSL with the older fcitx (fcitx4); this role is the desktop / fcitx5 counterpart.

Pre-flight safety
-----------------

The role first inspects the target host with `im-config -m`, `dpkg -l`, and `pgrep ibus-daemon` and **refuses to switch** if another IM framework is already configured (e.g. `ibus`). To proceed anyway, set `fcitx5_mozc_force_switch=true`. This is intentional — silently overwriting `GTK_IM_MODULE` etc. would orphan the user's existing dictionaries and key customizations on the previous framework.

Requirements
------------

- Ansible 2.9 or higher
- Ubuntu 24.04 or later (tested on 26.04 LTS)
- A desktop user account on the target

Role Variables
--------------

| Variable | Default | Description |
|---|---|---|
| `fcitx5_mozc_force_switch` | `false` | Override the safety check that halts when another IM is configured |
| `fcitx5_mozc_purge_ibus_mozc` | `false` | Also purge `ibus-mozc` / `ibus-anthy` to avoid duplicate Mozc backends |
| `fcitx5_mozc_packages` | fcitx5 + mozc + gtk3/4 + qt5/6 frontends + GUI tools + CJK fonts | Override to slim down |
| `fcitx5_mozc_generate_locale` | `true` | Generate `ja_JP.UTF-8` |
| `fcitx5_mozc_set_system_env` | `true` | Write `GTK_IM_MODULE` etc. to `/etc/environment` |
| `fcitx5_mozc_run_im_config` | `true` | Run `im-config -n fcitx5` for the user |
| `fcitx5_mozc_trigger_key` | `Control+space` | fcitx5 trigger key in `~/.config/fcitx5/config` |
| `fcitx5_mozc_keyboard_layout` | `us` | Keyboard layout for the "direct input" side. `us` or `jp` |
| `fcitx5_mozc_install_autostart` | `true` | Install `~/.config/autostart/fcitx5.desktop` (Ubuntu+GNOME does NOT autostart fcitx5 by default after im-config) |
| `fcitx5_mozc_mask_ibus_autostart` | `true` | Stub out `/etc/xdg/autostart/ibus-*.desktop` per-user, so GDM does not relaunch ibus-daemon every login |
| `fcitx5_mozc_configure_gnome_dconf` | `true` | System-wide dconf override: pin `input-sources` to a single layout, clear `switch-input-source` WM keybindings (prevents GNOME from stealing Ctrl+Space) |
| `fcitx5_mozc_user` | (required) | Desktop user to apply per-user settings to |
| `fcitx5_mozc_user_home` | (required) | Home directory of that user |

Dependencies
------------

None.

Example Playbook
----------------

```yaml
- hosts: ubu26
  become: yes
  vars:
    fcitx5_mozc_user: "{{ lookup('env', 'USER') }}"
    fcitx5_mozc_user_home: "{{ lookup('env', 'HOME') }}"
  roles:
    - fcitx5_mozc
```

If you know you want to drop an existing IBus+Mozc setup:

```yaml
- hosts: ubu26
  become: yes
  vars:
    fcitx5_mozc_user: dobachi
    fcitx5_mozc_user_home: /home/dobachi
    fcitx5_mozc_force_switch: true
    fcitx5_mozc_purge_ibus_mozc: true
  roles:
    - fcitx5_mozc
```

Tags
----

- `fcitx5_mozc`: Run the full role
- `fcitx5_mozc_check`: Run only the pre-flight detection (handy to dry-inspect a host)

Post-install
------------

- Re-login (or reboot) so the desktop session picks up the new `/etc/environment` and the new `im-config` selection.
- Verify with `fcitx5-diagnose`.
- Test: open a text editor and press **Ctrl + Space**; the IME indicator should toggle between direct input and Mozc.

License
-------

BSD-3-Clause
