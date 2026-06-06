docker
======

Docker CE と Compose v2 plugin を **Ubuntu (24.04 noble / 26.04 resolute)** 向けに
公式手順 (docs.docker.com/engine/install/ubuntu/) で導入するロール。

旧 [`roles/docker_legacy`](../docker_legacy/) との違い:

- `apt_key` (Ubuntu 24+ で deprecated) を使わず、`/etc/apt/keyrings/docker.asc`
  に keyring を配置して `signed-by=` で参照する。
- リポジトリの codename をハードコードせず `docker_codename` 変数で制御。
- `docker-compose` v1 (Python, 2023 年 EOL) ではなく `docker-compose-plugin`
  (v2, Go 実装) を入れる。
- 対話ユーザー (`docker_users`) を docker グループに追加するオプション。

Debian / CentOS / Rocky では `roles/docker_legacy` を使ってください
(本ロールは assert で Ubuntu 以外を弾きます)。

Requirements
------------

- Ansible 2.9 以上
- Ubuntu 24.04 (noble) 以降
- root/sudo 権限
- ネット接続 (download.docker.com にアクセス)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `docker_enable` | `false` | このロールを実行するかのスイッチ |
| `docker_codename` | `noble` | Docker apt リポの codename。Ubuntu 26.04 (resolute) は 2026-06 時点で公式 dists/ に未提供のため、安定して動く LTS の noble を既定値にしてある。公式が resolute を公開したら `{{ ansible_distribution_release }}` に切替可能 |
| `docker_packages` | `[docker-ce, docker-ce-cli, containerd.io, docker-buildx-plugin, docker-compose-plugin]` | 入れる apt パッケージ |
| `docker_users` | `[]` | docker グループに追加するユーザー名のリスト (sudo 不要で docker CLI を叩けるようにする) |
| `docker_service_enable` | `true` | `docker.service` を enable + start するか |
| `docker_keyring_path` | `/etc/apt/keyrings/docker.asc` | keyring 配置先 |
| `docker_keyring_url` | `https://download.docker.com/linux/ubuntu/gpg` | keyring 取得元 |

Dependencies
------------

なし。

Example Playbook
----------------

```yaml
- hosts: k16
  become: true
  roles:
    - role: docker
      docker_enable: true
      docker_users: [dobachi]
```

実行後の動作確認:

```bash
docker version                           # Client + Server (Engine) 両方が見える
docker compose version                   # Compose v2.x が見える
sudo systemctl status docker             # active (running)
groups dobachi | grep -o docker          # docker  ← グループに入った
# グループ反映には再ログイン or `newgrp docker` が必要
docker run --rm hello-world              # 動けば一連の手順 OK
```

`docker_codename` を上書きしたいとき (Docker 公式が resolute を出した後):

```yaml
- role: docker
  docker_enable: true
  docker_codename: "{{ ansible_distribution_release }}"
```

Tags
----

- `docker`: このタグでこのロールだけ走らせられる

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
