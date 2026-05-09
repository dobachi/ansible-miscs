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
| `copyq_shortcut_command` | `copyq menu` | 実行コマンド (`copyq toggle` などにも変更可) |
| `copyq_shortcut_binding` | `<Super>v` | 割り当てるキー |

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
- `copyq menu` は CopyQ デーモンが未起動なら自動起動します。
  起動時に常駐させたい場合は `~/.config/autostart/com.github.hluk.copyq.desktop`
  を別途配置してください (本ロールの対象外)。

Tags
----

- `copyq`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
