vnc_server
==========

Ubuntu 26.04 (resolute) で **TigerVNC standalone server + Xfce4** をセットアップ
する Ansible ロール。Wayland 専用の GNOME とは独立した **Xvnc セッション**
を立てるため、物理ディスプレイの状態 (接続/未接続) に左右されない遠隔操作
環境を提供する。

なぜ GNOME 共有ではなく独立 Xfce か
----------------------------------

Ubuntu 26 の GNOME は Wayland のみで、従来の X11 ベース VNC では GNOME
セッションを共有できない。`gnome-remote-desktop` も 46 以降は VNC を
drop し RDP のみサポート。よって VNC を使う場合は **Xvnc 上に独立した DE
を立てる** のがベスプラ。

物理ディスプレイ依存の問題:
- 物理 HDMI 未接続だと GNOME headless RDP は仮想 EDID (kernel cmdline) を
  要求し、その仮想 EDID は物理モニタ再接続時に縦横比を破壊する副作用がある
- Xvnc は DRM connector を一切使わないため、これらの問題と無縁

含まれるもの
------------

1. apt 導入: `tigervnc-standalone-server`, `tigervnc-common`, `tigervnc-tools`,
   `xfce4`, `xfce4-goodies`, `dbus-x11`
2. `~/.vnc/{passwd,config,xstartup}` を template (root が user 権限で作成)
3. `/etc/tigervnc/vncserver.users` に display:user マッピング追加
4. `ufw` で VNC port (デフォルト 5901) を Tailscale subnet のみ許可
5. `tigervncserver@:N.service` (system service テンプレート) を enable + start

使い方
------

### 必須変数 (Vault 強推奨)

```yaml
# host_vars/<host>/main.yml
vnc_server_enable: true
vnc_server_user: "dobachi"
vnc_server_password: "{{ vault_vnc_password }}"
```

```bash
# host_vars/<host>/vault.yml に Vault 化したパスワードを格納
ansible-vault encrypt_string 'YourStrongPassword' \
  --name 'vault_vnc_password' >> host_vars/<host>/vault.yml
```

### 任意変数 (defaults/main.yml 参照)

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `vnc_server_display` | `:1` | Xvnc display 番号 |
| `vnc_server_port` | `5901` | listen ポート (= 5900 + display N) |
| `vnc_server_geometry` | `1920x1080` | 解像度 |
| `vnc_server_depth` | `24` | 色深度 (bit) |
| `vnc_server_localhost_only` | `false` | true で 127.0.0.1 のみ |
| `vnc_server_xstartup_cmd` | `startxfce4` | Xvnc 起動後に exec するコマンド |
| `vnc_server_unit_enabled` | `true` | boot 時 enable するか |
| `vnc_server_unit_state` | `started` | started / stopped |
| `vnc_server_force_password_rewrite` | `false` | パスワード再生成 |

### 実行

```bash
ansible-playbook -i hosts playbooks/conf/linux/vnc_server.yml \
  -e server=k16 --ask-vault-pass
```

### 接続

```
ホスト: k16.<tailnet>.ts.net (Tailscale MagicDNS)
ポート: 5901
パスワード: vault_vnc_password に設定したもの
```

クライアント:
- Linux:   `remmina`, `vinagre`, `tigervnc-viewer`
- macOS:   標準の Screen Sharing (`vnc://k16.<tailnet>.ts.net:5901`)
- Windows: TigerVNC Viewer, TightVNC Viewer, RealVNC Viewer

`localhost_only: true` のときはクライアント側で SSH トンネルが必要:

```bash
ssh -L 5901:127.0.0.1:5901 k16.<tailnet>.ts.net
vncviewer localhost:5901
```

セキュリティ
------------

- VNC の生プロトコルは平文だが、Tailscale (WireGuard) で transport を
  暗号化するため Tailscale 内通信に閉じる前提で生 VNC でも実用上 OK。
- `vnc_server_open_firewall: true` (既定) で ufw は Tailscale subnet
  (100.64.0.0/10, fd7a:115c:a1e0::/48) のみ allow。それ以外は drop。
- より厳密にしたい場合は `vnc_server_localhost_only: true` + SSH トンネル。

トラブルシューティング
----------------------

| 症状 | 確認 |
| --- | --- |
| 接続するも黒画面 | `~/.vnc/xstartup` が exec 失敗。`~/.vnc/<host>:N.log` |
| service が起動しない | `journalctl -u tigervncserver@:1.service` |
| パスワード変更したい | `-e vnc_server_force_password_rewrite=true` で再実行 |

同梱: vim 行
-----------

```
# vim: set et ts=2 sw=2:
```
