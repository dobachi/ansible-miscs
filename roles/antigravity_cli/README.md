antigravity_cli
===============

Antigravity CLI（`agy` / Google のターミナル用エージェント）を**公式 GitHub リリースの
tar.gz** からインストールするロール。`godot` ロールと同じ流儀で
`~/Applications/antigravity-cli/<version>/` に展開し、`alternatives` で
`~/.local/bin/agy` を選択中バージョンのバイナリに向ける（PATH から `agy` 起動可）。

リリース: <https://github.com/google-antigravity/antigravity-cli/releases>

Requirements
------------

- 対象は Linux x86_64 / arm64（WSL2 を含む）。
- **`sudo ansible-playbook ...` で実行すること**（プロセス全体を sudo 起動）。タスクは
  `ansible_facts['env']['SUDO_USER']` を参照して sudo 元ユーザーのホーム配下に入れる。
  `godot` / `goland` など他のアプリ系ロールと同じ前提。
- tar.gz 内のバイナリ実体名は `antigravity` だが、`alternatives` で `agy` を張るため、
  起動・バージョン確認は公式ドキュメント通り `agy --version` で行う。

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `antigravity_cli_version` | `1.0.13` | リリースタグ。[releases](https://github.com/google-antigravity/antigravity-cli/releases) 参照 |
| `antigravity_cli_arch` | `linux_x64` | アセットのアーキ種別。arm64 は `linux_arm64` |
| `antigravity_cli_home` | `~/Applications/antigravity-cli` | 展開先ベース（SUDO_USER のホーム配下） |
| `antigravity_cli_command` | `agy` | PATH に張るコマンド名 |
| `antigravity_cli_mcp_enable` | `true` | Claude Code 等から agy を MCP 経由で使えるようにするか |
| `antigravity_cli_mcp_package` | `antigravity-cli-mcp` | 導入する npm パッケージ (Bun 必須) |
| `antigravity_cli_mcp_server_name` | `agy` | `~/.claude.json` の mcpServers に登録する名前 |
| `antigravity_cli_mcp_register_claude` | `true` | `~/.claude.json` への登録を行うか |
| `antigravity_cli_mcp_claude_config` | `~/.claude.json` | 登録先 (SUDO_USER のユーザースコープ) |
| `antigravity_cli_mcp_agy_bin` | `~/.local/bin/agy` | MCP サーバが叩く agy バイナリ (`AGY_PATH`) |
| `antigravity_cli_mcp_timeout_ms` | `300000` | ask-agy 1 回のタイムアウト (ms) |

別バージョンを入れる場合は `-e antigravity_cli_version=1.0.12` のように上書きする。
`alternatives` により `~/.local/bin/agy` のリンク先が切り替わり、複数バージョンを共存できる。

MCP サーバ (Claude Code から agy を使う)
----------------------------------------

Antigravity CLI 自体には「自分を MCP サーバとして公開する」機能が無い (agy は MCP
*クライアント* 側)。そのため本ロールは `antigravity_cli_mcp_enable: true` (既定) の
とき、サードパーティの [`antigravity-cli-mcp`](https://www.npmjs.com/package/antigravity-cli-mcp)
を **Bun** (`roles/bun` を `include_role`) で `bun add -g` 導入し、Claude Code の
ユーザースコープ設定 `~/.claude.json` の `mcpServers` に `agy` として登録する。

- 公開ツール: `ping` / `ask-agy` / `search-web` / `write-file`
- パッケージは `Bun.spawn` / `Bun.write` を使うため node では動かず Bun が要る
  (既存の gemini-cli / codex は npx なのと違い、ここだけ Bun ランタイムを足す)。
- 登録は既存の `mcpServers` をマージで保存するので、gemini-cli / codex 等の
  既存エントリは消えない。
- MCP を入れたくない場合は `-e antigravity_cli_mcp_enable=false`。

反映後、**Claude Code を再起動**すると `agy` MCP サーバが現れ、`/mcp` で確認できる。
agy の認証 (`~/.gemini/antigravity-cli` の OAuth トークン) は事前に `agy` を一度
起動してログインしておくこと。

Example Playbook
----------------

    - hosts: "{{ server | default('wsl') }}"
      roles:
        # register_home / commons は meta 依存として自動実行される。
        - antigravity_cli

実行例（プロセス全体を sudo で起動する点に注意）:

    # この環境 (localhost / wsl) に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/antigravity_cli.yml

    # k16 に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/antigravity_cli.yml -e server=k16

インストール後は `agy --version` で確認し、`agy` でエージェントを起動する。

License
-------

MIT

Author Information
------------------

dobachi
