gnome_screenshot_shortcut
=========================

GNOME Shell 組み込みのスクリーンショット UI を、既定の Print キーに加えて
Windows 風の `Super+Shift+S` でも起動できるようにする system-wide dconf
override ロール。

PrintScreen キーが無いノート PC でも、`Super+Shift+S` で

- 矩形選択
- ウィンドウ単位
- 全画面
- 画面録画

の UI が立ち上がる。撮ったスクリーンショットはクリップボードコピー +
`~/Pictures/Screenshots/` への保存が同時に行われる (GNOME 既定動作)。

Requirements
------------

- Ansible 2.9 以上
- GNOME Shell 42+ (推奨 46+)
- root 権限 (`/etc/dconf/...` を編集するため)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `gnome_screenshot_shortcut_keys` | `"['Print', '<Super><Shift>s']"` | `show-screenshot-ui` に割り当てるキー配列 (dconf GVariant 形式の文字列) |

Print キーを残したくない場合や別キーに変えたい場合は上書き:

```yaml
- role: gnome_screenshot_shortcut
  vars:
    gnome_screenshot_shortcut_keys: "['<Super><Shift>s']"
```

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - gnome_screenshot_shortcut
```

注意点
------

- dconf override は **system-wide のデフォルト値** として効きます。ユーザが
  既に `show-screenshot-ui` をカスタム設定している場合はユーザ側の値が優先
  されます (`gsettings reset org.gnome.shell.keybindings show-screenshot-ui`
  で既定 = 本ロールの値に戻せます)。
- 反映には GNOME セッションの再ログインが必要なことがあります。
  即時反映したい場合は手動で:
  `gsettings set org.gnome.shell.keybindings show-screenshot-ui "['Print', '<Super><Shift>s']"`

Tags
----

- `gnome_screenshot_shortcut`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
