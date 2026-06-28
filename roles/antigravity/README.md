antigravity
===========

Google Antigravity（エージェント型 IDE）を**公式 APT リポジトリ**から導入するロール。
`docker` ロールと同じ流儀で、`/etc/apt/keyrings/` に署名鍵を配置し `signed-by` で
公式リポジトリを追加して `apt install antigravity` する。以後は通常の
`apt upgrade` で本体が自動更新される。

公式手順: <https://antigravity.google/download/linux>

Requirements
------------

- 対象は **Debian/Ubuntu (apt) 専用**。`ansible_os_family == "Debian"` を assert する。
- Antigravity は **x86_64 (amd64)** のみ提供。
- root 権限が必要（`become: true` もしくは `sudo ansible-playbook ...`）。
- glibc >= 2.28 / glibcxx >= 3.4.25（Ubuntu 22.04 以降 / Debian 12 以降なら充足）。

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `antigravity_package` | `antigravity` | 導入する apt パッケージ名 |
| `antigravity_state` | `present` | パッケージ状態。最新追従は `latest` |
| `antigravity_keyring_url` | `https://us-central1-apt.pkg.dev/doc/repo-signing-key.gpg` | 署名鍵（ASCII armored）の取得元 |
| `antigravity_keyring_path` | `/etc/apt/keyrings/antigravity-repo-key.gpg` | dearmor 後の鍵の配置先 |
| `antigravity_repo_url` | `https://us-central1-apt.pkg.dev/projects/antigravity-auto-updater-dev/` | apt リポジトリ URL |
| `antigravity_repo_suite` | `antigravity-debian` | suite |
| `antigravity_repo_component` | `main` | component |
| `antigravity_repo_arch` | `amd64` | リポジトリの arch 指定 |

Example Playbook
----------------

    - hosts: "{{ server | default('wsl') }}"
      become: true
      roles:
        - antigravity

実行例:

    # この環境 (localhost / wsl) に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/antigravity.yml

    # k16 に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/antigravity.yml -e server=k16

インストール後は `antigravity` コマンド、またはアプリ一覧の "Antigravity" から起動する。

License
-------

MIT

Author Information
------------------

dobachi
