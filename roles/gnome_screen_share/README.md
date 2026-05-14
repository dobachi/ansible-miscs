gnome_screen_share
==================

GNOME Remote Desktop の **"Desktop Sharing"** (=ログイン済セッションに RDP
で横入りして同じ画面を共有する) を **Tailscale 経由限定** で構成するロール。

GNOME 46+ で標準採用された `gnome-remote-desktop` を `grdctl` 経由で設定し、
user systemd サービスを起動する。SSH や GNOME Remote Login (新規 RDP セッ
ションを立ち上げる headless モード) とは独立に共存可能。

含まれるもの
------------

| 内容 | 詳細 |
| --- | --- |
| パッケージ導入 | `gnome-remote-desktop` |
| linger 有効化 | `loginctl enable-linger <user>` (任意、デフォルト ON) |
| ufw ルール | 3389/tcp を Tailscale CGNAT (`100.64.0.0/10`) 限定で許可 |
| grdctl 設定 | RDP 認証情報、view-only モード、有効化 |
| systemd | `gnome-remote-desktop.service` の user 起動 |

Requirements
------------

- Ansible 2.10 以上 + `community.general` collection (ufw タスクで必要)
- Ubuntu 24.04+ / Debian 13+ (gnome-remote-desktop が新版である必要)
- GNOME 46+ で動作中のデスクトップ
- 既にログイン中のデスクトップセッション (per-user タスクが DBus 経由で
  動くため、playbook はそのユーザのシェル — 例えば gnome-terminal — から
  `-K` 付きで実行すること)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `gnome_screen_share_install` | `true` | apt 導入を行うか |
| `gnome_screen_share_user` | `SUDO_USER` 自動検出 | 配備対象のデスクトップユーザ |
| `gnome_screen_share_user_uid` | `SUDO_UID` 自動検出 | 同 UID (DBus パス算出用) |
| `gnome_screen_share_rdp_username` | `""` (**必須**) | RDP クライアントから入力する username |
| `gnome_screen_share_rdp_password` | `""` (**必須・Vault 推奨**) | 同 password |
| `gnome_screen_share_view_only` | `false` | true でリモート側からの操作を禁止 (閲覧のみ) |
| `gnome_screen_share_enable_linger` | `true` | ログアウト後も user services を維持 |
| `gnome_screen_share_open_firewall` | `true` | ufw に許可ルールを追加するか |
| `gnome_screen_share_firewall_sources_v4` | `['100.64.0.0/10']` | 許可元 IPv4 サブネット |
| `gnome_screen_share_firewall_sources_v6` | `['fd7a:115c:a1e0::/48']` | 許可元 IPv6 サブネット |

Vault によるパスワード保護
--------------------------

平文で書かないこと。host_vars または group_vars にこんな形で:

```bash
# 1) パスワードを暗号化して host_vars に書き出し
mkdir -p host_vars/k16
ansible-vault encrypt_string 'YourStrongPassword' \
  --name 'vault_gnome_rdp_password' >> host_vars/k16/vault.yml

# 2) host_vars/k16/main.yml に参照を追加 (平文で OK)
cat >> host_vars/k16/main.yml <<'EOF'
gnome_screen_share_rdp_username: "remote_user"
gnome_screen_share_rdp_password: "{{ vault_gnome_rdp_password }}"
EOF
```

Example Playbook
----------------

専用 playbook `playbooks/conf/linux/gnome_screen_share.yml` を用意。
デスクトップ機の **gnome-terminal** から (= dobachi の DBus セッションが
利きる場所から) 実行:

```bash
ansible-playbook playbooks/conf/linux/gnome_screen_share.yml \
  -e server=localhost --connection=local -K --ask-vault-pass
```

接続側 (RDP クライアント)
--------------------------

- Windows: 標準の「リモートデスクトップ接続」(`mstsc`)
- macOS: Microsoft Remote Desktop (App Store)
- Linux: Remmina (`apt install remmina`)

Tailscale 経由で接続:

```
host: k16.<tailnet>.ts.net   (または tailscale-assigned IP)
port: 3389
user: <gnome_screen_share_rdp_username>
pass: <gnome_screen_share_rdp_password>
```

接続後、dobachi が今見ている画面そのものに乗り入れる (= 画面共有モード)。
view-only=false なら相互に操作可能、true なら閲覧のみ。

注意点
------

- 既にログイン中の dobachi セッションが**ある**ときは画面共有モードに
  なる。**無い**ときに RDP で繋ぐと "Remote Login" モードとして新規
  セッションが立ち上がる挙動が発生し得る (gnome-remote-desktop の挙動による)。
  完全に画面共有のみにしたい場合は GNOME 設定 → System → Remote Desktop で
  "Remote Login" を OFF にしておく。
- `grdctl` と `systemctl --user` は対象ユーザの DBus を必要とする。playbook
  実行シェルが tmux など環境変数を引き継いでいない場合は `XDG_RUNTIME_DIR`
  と `DBUS_SESSION_BUS_ADDRESS` をロール側で明示しているのでそのままでも
  動くはずだが、できればデスクトップ内のターミナルから実行するのが堅実。

Tags
----

- `gnome_screen_share`: Apply this tag to run only this role

License
-------

BSD

Author Information
------------------

This role was created for the ansible-miscs project.
