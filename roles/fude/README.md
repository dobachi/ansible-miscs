fude
====

[dobachi/fude](https://github.com/dobachi/fude) (Tauri 製アプリ) の upstream
`.deb` を GitHub Releases から取得して `apt` 経由で導入するロール。

[`roles/rclone_sharepoint`](../rclone_sharepoint/) と同じ「upstream .deb +
apt インストール」路線。`fude_version: latest` (既定) なら GitHub Releases
API で `tag_name` を解決し、dpkg-query と比較してインストール要否を自動判定する
(べき等)。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (amd64)
  - 現リリースは amd64 の .deb のみ提供。arm64 等は assert で弾く
- ネット接続 (GitHub Releases にアクセス)
- root/sudo 権限 (apt 経由のインストールのため)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `fude_enable` | `false` | このロールを実行するかのスイッチ |
| `fude_repo` | `dobachi/fude` | GitHub owner/repo (fork するなら差し替え) |
| `fude_version` | `latest` | `latest` で API 自動解決 / 固定したいなら `v0.3.7` のような tag をそのまま指定 |
| `fude_arch` | `""` | 空なら `ansible_facts['architecture']` を `amd64`/`arm64` にマップして自動解決 |
| `fude_download_dir` | `/tmp` | .deb を落とす一時ディレクトリ。インストール後に削除 |

Dependencies
------------

なし。

Example Playbook
----------------

```yaml
- hosts: workstation
  become: true
  roles:
    - role: fude
      fude_enable: true
```

実行例 ([playbooks/conf/linux/fude.yml](../../playbooks/conf/linux/fude.yml)):

```bash
# 最新リリースを入れる (既にインストール済みなら no-op)
ansible-playbook -i hosts playbooks/conf/linux/fude.yml \
  -e server=localhost -e fude_enable=true

# バージョンを固定する
ansible-playbook -i hosts playbooks/conf/linux/fude.yml \
  -e server=localhost -e fude_enable=true -e fude_version=v0.3.7
```

実行後の動作確認:

```bash
dpkg -l fude | tail -1                 # ii fude 0.3.7  ...
which fude                             # /usr/bin/fude
# GUI なので fude を起動して画面が出ること
```

アンインストール:

```bash
sudo apt remove fude
```

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
