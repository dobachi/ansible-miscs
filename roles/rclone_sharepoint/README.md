rclone_sharepoint
=================

`rclone mount` で SharePoint Online のドキュメントライブラリを
`/mnt/sharepoint` (既定) に systemd 経由で常設マウントするロール。

`rclone` の `onedrive` バックエンド (`drive_type=sharepoint`) を使い、
FUSE で透過マウントする。systemd system unit が boot 時に自動起動する。

**k16 単体で完結する設計**: 別マシンや ansible-vault は不要。k16 上で
`rclone config` を一度だけ手で走らせれば、以降は systemd が面倒を見る。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (apt と systemd を使う)
- 対象ホスト (k16 等) にブラウザがあること
  (OAuth 認証で http://127.0.0.1:53682/auth を開くため)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `rclone_sharepoint_enable` | `false` | このロールの本体を動かすかのスイッチ。host_vars で `true` にする |
| `rclone_sharepoint_remote_name` | `sharepoint` | rclone.conf 内の `[remote]` 名 (`rclone config` の "name" と一致させる) |
| `rclone_sharepoint_mountpoint` | `/mnt/sharepoint` | マウント先 |
| `rclone_sharepoint_user` | `dobachi` | FUSE `--uid/--gid` と rclone.conf 探索先のユーザー名 |
| `rclone_sharepoint_umask` | `022` | FUSE `--umask` |
| `rclone_sharepoint_vfs_cache_mode` | `writes` | `--vfs-cache-mode` |
| `rclone_sharepoint_dir_cache_time` | `1000h` | `--dir-cache-time` |
| `rclone_sharepoint_log_level` | `INFO` | rclone のログレベル |
| `rclone_sharepoint_config_path` | `""` | rclone.conf の場所を明示したい場合に上書き。空なら `~{user}/.config/rclone/rclone.conf` を自動算出 |
| `rclone_sharepoint_service_name` | `rclone-sharepoint.service` | systemd unit ファイル名 |

`rclone_sharepoint_enable: false` のときは本体ブロックを丸ごと skip する
ため、`ubu26_desktop.yml` 等の共用 playbook に組み込んでも、host_vars で
明示的に有効化しないホストでは何も起きない。

Dependencies
------------

なし。rclone と fuse3 はロール内で apt から導入する。

セットアップ手順 (k16 単体で完結)
------------------------------

### 1. host_vars/k16/main.yml に有効化フラグを書く

```yaml
rclone_sharepoint_enable: true
# 必要に応じて以下を上書き
# rclone_sharepoint_mountpoint: /mnt/sharepoint
# rclone_sharepoint_user: dobachi
```

### 2. ロールを 1 度デプロイ (rclone のインストールと systemd unit 設置)

```bash
ansible-playbook -i hosts playbooks/conf/linux/rclone_sharepoint.yml \
  -e server=home_desktops
```

この時点では rclone.conf がまだ無いため、`ConditionPathExists` で
systemd は service を skip する (エラーにはならない)。playbook は最後に
「次に rclone config を走らせてね」というメッセージを出して正常終了する。

### 3. k16 上で `rclone config` を 1 度実行 (OAuth)

dobachi で k16 にログインし、対話で remote を作る:

```bash
rclone config
```

対話の答え方:

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

### 4. systemd サービスを start

```bash
sudo systemctl start rclone-sharepoint.service
mount | grep /mnt/sharepoint
ls /mnt/sharepoint
```

サービスはステップ 2 で既に enable 済みなので、以降の boot では自動マウント
される (rclone.conf がそのまま残っている限り)。

### 5. (任意) playbook を再実行しても安全

ステップ 2 以降の再実行はべき等。テンプレートに差分があれば daemon-reload
+ restart まで自動で走る。

トークンの取り扱いについて
--------------------------

rclone は SharePoint へアクセスするたびに access_token を refresh_token で
更新し、新しいトークンを rclone.conf に書き戻す。
このロールは rclone.conf を **生成も配布もしない** ため、k16 上で rclone が
普通に運用していればトークンは自動でリフレッシュされ続ける。

- Microsoft の refresh_token は無操作で約 90 日有効。日常的にマウントが
  動いていれば失効しない。
- 長期間止まっていて失効した場合は、k16 上で `rclone config reconnect
  sharepoint:` (または `rclone config` から該当 remote を edit) して
  再認証する。Ansible 側は触らなくてよい。

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
