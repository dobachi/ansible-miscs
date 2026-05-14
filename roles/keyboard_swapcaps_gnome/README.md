keyboard_swapcaps_gnome
=========

Caps Lock キーを Ctrl 化 (デフォルトでは `ctrl:nocaps` = Caps Lock 機能ごと
廃止) する。`ctrl:swapcaps` (双方向入れ替え) などにも切替可能。

> **注意**: 以前は `ctrl:swapcaps` をデフォルトとしていたが、Windows 側でも
> 同じ swap を入れているユーザが RDP で Linux に接続すると、scancode 側と
> xkb 側で「二重入れ替え」が発生して物理 Caps 押下が再度 Caps_Lock として
> 届く / Ctrl が効かない、というハマりがある。`ctrl:nocaps` だと Caps_Lock
> キーシムを生成しないので RDP 越しでも素直に Ctrl が利く。

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
| `keyboard_swapcaps_xkb_option` | `ctrl:nocaps` | XKB option string. Other useful values: `ctrl:swapcaps`, `caps:ctrl_modifier` |
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
- Per-user override via `gsettings set org.gnome.desktop.input-sources xkb-options "['ctrl:nocaps']"` will take precedence over the dconf system default for that user.
- **RDP からの利用について**: Windows 側でも Caps↔Ctrl の入れ替え (レジストリ
  Scancode Map 等) を設定している環境では、本ロールが `ctrl:swapcaps` だと
  二重入れ替えで詰む (詳細は冒頭の注意)。`ctrl:nocaps` (デフォルト) なら
  ローカル / RDP / Windows ローカル全ての環境で「物理 Caps の位置 = Ctrl」
  が成立する。

License
-------

BSD-3-Clause
