rclone_sharepoint
=================

`rclone mount` で SharePoint Online のドキュメントライブラリを
`/mnt/sharepoint` (既定) に systemd 経由で常設マウントするロール。

`rclone` の `onedrive` バックエンド (`drive_type=sharepoint`) を使い、
FUSE で透過マウントする。systemd system unit が boot 時に自動起動する。

**k16 単体で完結する設計**: 別マシンや ansible-vault は不要。k16 上で
`rclone config` を一度だけ手で走らせれば、以降は systemd が面倒を見る。

`rclone` は upstream GitHub Releases の `.deb` を入れる
(Ubuntu apt 同梱版は古く、SharePoint の list 周りで empty が返る
既知の問題があるため)。

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
| `rclone_version` | `1.74.1` | 導入する rclone のバージョン (GitHub Releases のタグから `v` を除いた値) |
| `rclone_release_base_url` | `https://github.com/rclone/rclone/releases/download` | upstream .deb の取得元 |
| `rclone_deb_arch` | 自動判定 (`amd64` / `arm64`) | rclone の .deb アーキ表記 |
| `rclone_deb_download_dir` | `/tmp` | .deb の一時保存先 |
| `rclone_sharepoint_remote_name` | `sharepoint` | rclone.conf 内の `[remote]` 名 (`rclone config` の "name" と一致させる) |
| `rclone_sharepoint_mountpoint` | `/mnt/sharepoint` | マウント先 |
| `rclone_sharepoint_user` | `dobachi` | systemd unit の `User=` と rclone.conf 探索先のユーザー名 |
| `rclone_sharepoint_umask` | `022` | FUSE `--umask` |
| `rclone_sharepoint_vfs_cache_mode` | `writes` | `--vfs-cache-mode` |
| `rclone_sharepoint_dir_cache_time` | `1000h` | `--dir-cache-time` |
| `rclone_sharepoint_log_level` | `INFO` | rclone のログレベル |
| `rclone_sharepoint_config_path` | `""` | rclone.conf の場所を明示したい場合に上書き。空なら `~{user}/.config/rclone/rclone.conf` を自動算出 |
| `rclone_sharepoint_service_name` | `rclone-sharepoint.service` | systemd unit ファイル名 |

`rclone_sharepoint_enable: false` のときは本体ブロックを丸ごと skip する
ため、`ubu26_desktop.yml` 等の共用 playbook に組み込んでも、host_vars で
明示的に有効化しないホストでは何も起きない。

systemd unit は `User={{ rclone_sharepoint_user }}` で実行されるため:
- rclone のトークンリフレッシュ書き戻しで rclone.conf の所有権が root に
  飛ぶ問題は発生しない。
- `--uid/--gid` を明示する必要がない (プロセス所有者がそのまま反映される)。
- マウントポイントの所有者もロールがこのユーザーに合わせる。

Dependencies
------------

なし。rclone は upstream .deb、fuse3 は apt から導入する。

セットアップ手順 (k16 単体で完結)
------------------------------

### 1. host_vars/<host>/main.yml に有効化フラグを書く

```yaml
rclone_sharepoint_enable: true
rclone_sharepoint_remote_name: "<remote name>"     # 任意。既定は sharepoint
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

### 3. 対象ホスト上で `rclone config` を 1 度実行 (OAuth)

`rclone_sharepoint_user` (既定 `dobachi`) としてログインし、対話で remote
を作る:

```bash
rclone config
```

対話の答え方:

| 質問 | 回答 |
| --- | --- |
| `n)` New remote | `n` |
| name | `rclone_sharepoint_remote_name` と一致させる |
| Storage | `onedrive` |
| client_id / client_secret | 空のままで OK (rclone 内蔵のを使う) |
| region | `1` (Microsoft Cloud Global) |
| tenant | 空のまま Enter |
| Edit advanced config? | `n` |
| Use auto config? | `y` (ブラウザで Microsoft 認証) |
| ブラウザの「アカウントを選択」 | **ブラウザで対象 SharePoint サイトを開く時と同じアカウントを選ぶ** |
| Type of connection | **項目名 `Sharepoint site name or URL` (`(url)` と表示される方)** を選ぶ ※下記注意 |
| Site URL | `https://<tenant>.sharepoint.com/sites/<site名>` |
| ドライブ選択 | 目的のライブラリ (普通は `Documents` / `Shared Documents`) |
| Yes this is OK | `y` |
| | `q` |

> **重要**: `Type of connection` で `Root Sharepoint site (sharepoint)` を
> 選ぶと、テナント直下の集約サイトの (実質空の) drive を掴んでしまい、
> `rclone lsf` が常に空になります。必ず `Sharepoint site name or URL`
> (`(url)` と表示される選択肢) を選んでサイト URL を入れること。
> rclone のバージョンによって選択肢の番号は変わるので、項目名で確認。

完了すると `~/.config/rclone/rclone.conf` に該当セクションが書き込まれる。

### 4. rclone 単体で動作確認 (systemd に渡す前)

```bash
rclone lsf <remote name>:
rclone about <remote name>:
```

`lsf` で実ファイル/フォルダが見えれば OK。空のままだったら drive 選択違い
(`rclone config` の `e) Edit existing remote` で library を選び直す)、
または OAuth のアカウントが対象サイトに権限を持っていない可能性。
`rclone about` の `Total` が 25 TiB / `Used` が 1.5 MiB ちょうどなら
「テナント直下の空 drive を掴んでいる」典型サイン。

### 5. systemd サービスを start

```bash
sudo systemctl start rclone-sharepoint.service
mount | grep /mnt/sharepoint
ls /mnt/sharepoint
```

サービスはステップ 2 で既に enable 済みなので、以降の boot では自動マウント
される (rclone.conf がそのまま残っている限り)。

### 6. (任意) playbook を再実行しても安全

ステップ 2 以降の再実行はべき等。テンプレートに差分があれば daemon-reload
+ restart まで自動で走る。rclone のバージョンが `rclone_version` の値と
一致していれば .deb のダウンロード/再導入もスキップ。

トークンの取り扱いについて
--------------------------

rclone は SharePoint へアクセスするたびに access_token を refresh_token で
更新し、新しいトークンを rclone.conf に書き戻す。
このロールは rclone.conf を **生成も配布もしない** うえ、systemd unit を
`User=<user>` で走らせているため、トークンリフレッシュ書き戻しが起きても
所有権・パーミッションは保たれる。

- Microsoft の refresh_token は無操作で約 90 日有効。日常的にマウントが
  動いていれば失効しない。
- 長期間止まっていて失効した場合は、対象ホスト上で
  `rclone config reconnect <remote>:` (または `rclone config` から
  該当 remote を edit) して再認証する。Ansible 側は触らなくてよい。

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
