gnome_screen_share
==================

GNOME Remote Desktop を **Tailscale 経由限定** で構成するロール。GNOME 46+
で標準採用された `gnome-remote-desktop` の 2 つのモードを並走させ、ログイン
時/ログアウト時の双方で RDP 接続できるようにする。

| ポート | モード | 動作 |
| --- | --- | --- |
| `3389` | **Desktop Sharing** (user mode) | dobachi が既にログイン中の GNOME セッションに乗り入れ (画面共有) |
| `3390` | **Remote Login** (system mode, headless) | ログアウト中でも新規 GNOME セッションを起動 |

クライアントは目的に応じて接続先ポートを選ぶ:
- 「作業継続のため画面共有したい」 → `:3389`
- 「ログアウト中だけど入りたい」     → `:3390`

SSH (22) や他のサービスとは独立に共存可能。

含まれるもの
------------

| 内容 | 詳細 |
| --- | --- |
| パッケージ導入 | `gnome-remote-desktop` |
| linger 有効化 | `loginctl enable-linger <user>` (任意、デフォルト ON) |
| ufw ルール | 3389/tcp と 3390/tcp を Tailscale CGNAT (`100.64.0.0/10`) 限定で許可 |
| Desktop Sharing | `grdctl rdp set-credentials / disable-view-only / enable` + user systemd |
| Remote Login | `grdctl --headless rdp set-credentials / set-port / enable` + system systemd |

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
| `gnome_screen_share_enable_remote_login` | `true` | ログアウト中でも RDP で繋げる system mode を有効化 |
| `gnome_screen_share_remote_login_port` | `3390` | Remote Login (headless) 用ポート |
| `gnome_screen_share_open_firewall` | `true` | ufw に許可ルールを追加するか |
| `gnome_screen_share_firewall_sources_v4` | `['100.64.0.0/10']` | 許可元 IPv4 サブネット |
| `gnome_screen_share_firewall_sources_v6` | `['fd7a:115c:a1e0::/48']` | 許可元 IPv6 サブネット |
| `gnome_screen_share_force_virtual_display` | `false` | kernel cmdline で仮想 EDID を強制 (物理モニタ未接続でも headless RDP 可能に)。**反映に reboot 必須** |
| `gnome_screen_share_virtual_display_connector` | `"HDMI-A-1"` | 仮想化対象の DRM connector 名 (`/sys/class/drm/card*/status` で確認) |
| `gnome_screen_share_virtual_display_mode` | `"1920x1080@60D"` | 仮想 EDID の解像度/モード |

### 仮想ディスプレイ (force_virtual_display) の挙動と副作用

GNOME Remote Desktop の Remote Login (headless) は本来仮想 framebuffer で動く設計ですが、Wayland + 物理モニタ未接続の条件で GDM/Mutter が compositor を起動できず即死することがあります。`gnome_screen_share_force_virtual_display: true` にすると `/etc/default/grub` の `GRUB_CMDLINE_LINUX_DEFAULT` に `video=<connector>:<mode>` を追加し、kernel に常時仮想 EDID を信じ込ませることでこの問題を回避します。

- **物理モニタ接続時**: 実 EDID が優先されるので挙動は変わらない
- **物理モニタ未接続時**: 仮想 1920x1080 のディスプレイが OS に見える
- 副作用:
  - GNOME 設定 → ディスプレイにファントム表示
  - 「ディスプレイ無し」を理由とする自動サスペンドが無効化される
  - GPU メモリ ~10MB 常時確保
- 反映には reboot が必要 (kernel cmdline 変更のため、`update-grub` は handler で自動実行)

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

Tailscale 経由で接続。目的でポート切替:

```
# 既存セッションに乗り入れ (画面共有)
host: k16.<tailnet>.ts.net   (または tailscale-assigned IP)
port: 3389
user: <gnome_screen_share_rdp_username>
pass: <gnome_screen_share_rdp_password>

# ログアウト中でも新規セッション (Remote Login)
host: k16.<tailnet>.ts.net
port: 3390
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
