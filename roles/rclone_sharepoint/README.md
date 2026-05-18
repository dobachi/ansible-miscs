rclone_sharepoint
=================

`rclone mount` で SharePoint Online のドキュメントライブラリを
`/mnt/sharepoint` (既定) に systemd 経由で常設マウントするロール。

`rclone` の `onedrive` バックエンド (`drive_type=sharepoint`) を使い、
FUSE で透過マウントする。systemd system unit が boot 時に自動起動する。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (apt と systemd を使う)
- 別マシン (ブラウザがある PC) で先に `rclone config` を一度走らせ
  rclone.conf を生成しておくこと
- ansible-vault で rclone.conf を暗号化し host_vars に格納できる環境

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `rclone_sharepoint_enable` | `false` | このロールの本体を動かすかのスイッチ。host_vars で `true` にする |
| `rclone_sharepoint_remote_name` | `sharepoint` | rclone.conf 内の `[remote]` 名 |
| `rclone_sharepoint_mountpoint` | `/mnt/sharepoint` | マウント先 |
| `rclone_sharepoint_user` | `dobachi` | FUSE `--uid/--gid` を引く対象ユーザー名 |
| `rclone_sharepoint_umask` | `022` | FUSE `--umask` |
| `rclone_sharepoint_vfs_cache_mode` | `writes` | `--vfs-cache-mode` |
| `rclone_sharepoint_dir_cache_time` | `1000h` | `--dir-cache-time` |
| `rclone_sharepoint_log_level` | `INFO` | rclone のログレベル |
| `rclone_sharepoint_config_path` | `/root/.config/rclone/rclone.conf` | rclone.conf 配置先 |
| `rclone_sharepoint_conf_content` | `""` | rclone.conf の中身。host_vars から vault 変数で渡す |
| `rclone_sharepoint_service_name` | `rclone-sharepoint.service` | systemd unit ファイル名 |

`rclone_sharepoint_enable: false` のときは本体ブロックを丸ごと skip する
ため、`ubu26_desktop.yml` 等の共用 playbook に組み込んでも、host_vars で
明示的に有効化しないホストでは何も起きない。

Dependencies
------------

なし。rclone と fuse3 はロール内で apt から導入する。

事前準備 (一度だけ手動でやる作業)
-------------------------------

### 1. 別マシンで rclone.conf を作る

ブラウザが使える PC (k16 でも可) で次を実行する:

```bash
rclone config
```

対話で以下を答える:

| 質問 | 回答 |
| --- | --- |
| `n)` New remote | `n` |
| name | `sharepoint`  (= `rclone_sharepoint_remote_name`) |
| Storage | `onedrive` |
| client_id / client_secret | 空のままで OK (rclone 内蔵のを使う) |
| Edit advanced config? | `n` |
| Use auto config? | `y` (ブラウザで Microsoft 認証する) |
| Type of connection | `Sharepoint site` (`drive_type=sharepoint`) |
| SharePoint site URL | 組織の `https://<tenant>.sharepoint.com/sites/<name>` |
| ドライブ選択 | 対象ドキュメントライブラリの番号 |
| Yes this is OK | `y` |

完了すると `~/.config/rclone/rclone.conf` に `[sharepoint]` セクションが
書き込まれる。

### 2. rclone.conf を ansible-vault に投入

```bash
cd <ansible-miscs>
ansible-vault encrypt_string "$(cat ~/.config/rclone/rclone.conf)" \
  --name 'vault_rclone_sharepoint_conf' \
  >> host_vars/k16/vault.yml
```

### 3. host_vars/k16/main.yml にスイッチを書く

```yaml
rclone_sharepoint_enable: true
rclone_sharepoint_conf_content: "{{ vault_rclone_sharepoint_conf }}"
# 必要に応じて以下も上書き
# rclone_sharepoint_mountpoint: /mnt/sharepoint
# rclone_sharepoint_user: dobachi
```

### 4. デプロイ

```bash
ansible-playbook -i hosts playbooks/conf/linux/rclone_sharepoint.yml \
  -e server=home_desktops --ask-vault-pass
```

もしくは `ubu26_desktop.yml` 経由で一括適用してもよい。
NOPASSWD sudo 環境なら `-K` は不要。vault を読むので
`--ask-vault-pass` (もしくは `.vault-pass.txt`) が必要。

### 5. 動作確認

```bash
systemctl status rclone-sharepoint.service
mount | grep /mnt/sharepoint
ls -la /mnt/sharepoint
touch /mnt/sharepoint/_ansible_smoketest && rm /mnt/sharepoint/_ansible_smoketest
```

トークンの取り扱いについて (重要)
------------------------------

rclone は SharePoint へアクセスするたびに access_token を refresh_token で
更新し、その新しいトークンを rclone.conf に書き戻す。
このロールは `copy: content=...` で rclone.conf を毎回上書きするため、
**Ansible デプロイの直後は rclone が運用中に更新した refresh_token が
失われ、vault に入っているトークンに巻き戻る** ことになる。

- Microsoft の refresh_token は無操作で約 90 日有効。日常的にマウントが
  動いていれば失効しない。
- 念のため、定期的に (例: 数か月に一度) 別マシンで `rclone config reconnect`
  でトークンを再生成 → vault に再投入する運用を推奨する。
- refresh_token が失効した場合はマウントが認証エラーになる。
  再度 `rclone config` でリフレッシュして vault を入れ替える。

Example Playbook
----------------

```yaml
- hosts: k16
  become: yes
  roles:
    - rclone_sharepoint
```

Tags
----

- `rclone_sharepoint` / `sharepoint`: このロールの全タスクに付与。

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
