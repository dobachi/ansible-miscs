bun
===

[Bun](https://bun.sh/) ランタイムを **per-user** で導入するロール。`pyenv` ロールと
同じ流儀で、`become_user` で対象ユーザーになり公式インストーラ
(`curl -fsSL https://bun.sh/install | bash`) を実行し、`~/.bun` に展開する。
`~/.bun/bin` を `.bashrc` の PATH に通すため、ログインし直せば `bun` / `bunx` が
使える。

Bun 必須の npm パッケージ (例: `antigravity-cli-mcp`) を動かすための土台として、
`antigravity_cli` ロールが MCP 有効時に `include_role` で呼ぶ。

Requirements
------------

- Debian/Ubuntu (インストーラの依存 `curl` / `unzip` を apt で入れる)。
- **`sudo ansible-playbook ...` で実行すること**。`bun_user` 既定は
  `ansible_facts['env']['SUDO_USER']`。

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `bun_user` | `{{ SUDO_USER }}` | Bun を入れる対象ユーザー |
| `bun_install_url` | `https://bun.sh/install` | 公式インストーラ URL |
| `bun_configure_bashrc` | `true` | `~/.bashrc` に `BUN_INSTALL` / `PATH` を追記するか |

冪等性: `~/.bun/bin/bun` が既にあればインストーラをスキップする (`creates`)。

Example Playbook
----------------

    - hosts: "{{ server | default('wsl') }}"
      roles:
        - bun

実行例:

    sudo ansible-playbook -i hosts playbooks/conf/linux/bun.yml

インストール後、`source ~/.bashrc`（または再ログイン）して `bun --version` で確認する。

License
-------

MIT

Author Information
------------------

dobachi
