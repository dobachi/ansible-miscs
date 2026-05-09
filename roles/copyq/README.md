copyq
=========

CopyQ クリップボードマネージャをインストールし、GNOME 側に
`Super+V` (Windows の Win+V 相当) でクリップボード履歴ポップアップを
開くカスタムショートカットを登録します。

ショートカットは GNOME のカスタムキーバインドとして dconf system-wide
override で設定します (`org.gnome.settings-daemon.plugins.media-keys`)。
Wayland 上では CopyQ 単体ではグローバルショートカットを登録できないため、
GNOME 側からバインドして `copyq menu` を呼び出す方式を採用しています。

Requirements
------------

- Ansible 2.9 or higher
- Linux (apt 系ディストリビューション)
- GNOME (Super+V ショートカットを使う場合)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `copyq_install_shortcut` | `true` | Super+V のカスタムショートカットを登録する |
| `copyq_shortcut_name` | `CopyQ clipboard history` | キーバインド名 |
| `copyq_shortcut_command` | `copyq show` | 実行コマンド。Wayland 上で `copyq menu` はフォーカス維持できず一瞬で閉じるため、メイン画面を開く `copyq show` を既定とした (`copyq toggle` も可) |
| `copyq_shortcut_binding` | `<Super>v` | 割り当てるキー |
| `copyq_enable_autostart` | `true` | GNOME ログイン時に CopyQ デーモンを自動起動するか (`/etc/xdg/autostart/` に .desktop を配置) |
| `copyq_autostart_src` | `/usr/share/applications/com.github.hluk.copyq.desktop` | コピー元 (apt 同梱のランチャ .desktop) |
| `copyq_autostart_dest` | `/etc/xdg/autostart/com.github.hluk.copyq.desktop` | autostart .desktop の配置先 |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: servers
  become: yes
  roles:
    - copyq
```

ショートカットを登録したくない場合:

```yaml
- hosts: servers
  become: yes
  roles:
    - role: copyq
      vars:
        copyq_install_shortcut: false
```

注意点
------

- dconf override は **system-wide のデフォルト値** として効きます。
  ユーザが既に独自のカスタムキーバインドを設定している場合、
  `custom-keybindings` 配列はユーザ側が優先されるため、本ロールの
  バインドが適用されないことがあります。その場合は GNOME 設定 →
  キーボード → ショートカットから手動で `copyq menu` を Super+V に
  割り当ててください。
- GNOME Shell の組み込み `toggle-message-tray` が既定で
  `['<Super>m', '<Super>v']` を握っており、シェル組み込み binding は
  カスタム binding より優先されます。本ロールはこの override で
  `toggle-message-tray` から `<Super>v` を外します (`<Super>m` は残るので
  通知パネルは引き続き Super+M で開けます)。
- CopyQ サーバが起動していないとショートカットは無反応です。本ロールは
  `copyq_enable_autostart: true` 時に
  `/etc/xdg/autostart/com.github.hluk.copyq.desktop` を配置するため、次回 GNOME
  ログインから自動起動します。手動で先に上げたいときは GNOME セッション内の
  ターミナルで `copyq --start-server show` を一度叩いてください。
- Wayland では `copyq menu` (ポップアップ) はフォーカスを保持できず開いた瞬間
  に閉じる既知の問題があるため、ショートカットは `copyq show` (メイン画面)
  を既定としています。Win+V 風のポップアップ動作が欲しい場合は CopyQ を
  XWayland 経由 (`QT_QPA_PLATFORM=xcb`) で起動した上で
  `copyq_shortcut_command: "copyq menu"` に変更してください。

Tags
----

- `copyq`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
