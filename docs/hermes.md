# Hermes Agent セットアップ (Docker sandboxed, OpenRouter primary)

このドキュメントは、本リポジトリの Ansible ロールを使って **Hermes Agent CLI**
を Docker container 版で立ち上げ、host FS の触れる範囲を専用ワークスペースに
限定する手順をまとめたもの。

## 隔離モデル

Hermes 公式が用意している **2つの Docker 連携** のうち、
**"Running Hermes IN Docker"** (全体 container 化) を採用する。

> 公式 docs より抜粋: "There are two distinct ways Docker intersects with
> Hermes Agent: (1) Running Hermes IN Docker… (2) Docker as a terminal
> backend… **No nesting occurs.** You choose one model."

なぜ (1) を選ぶか — `file` tool に workspace 制限フラグが無いため:

- `tools/file_tools.py:461-462` — 絶対パスは resolve してそのまま実行。
- `_path_resolution_warning` — workspace 外への書き込みも警告するだけで**阻止しない**。
- `security:` config には path 系の restrict が無い (`allow_private_urls`,
  `redact_secrets`, `tirith_*` 等はあるが path scope 用ではない)。

したがって `terminal.backend: docker` (公式 Docker 連携の (2)) では shell
コマンドは container で走るが、**`file` tool は依然 host FS を触る**。
`file` tool を含めた完全隔離には、エージェントプロセス全体を container 化
するしかない (= 公式 Docker 連携 (1))。

## アーキテクチャ

```
[Host]                                    [container: nousresearch/hermes-agent]
~/hermes-workspace  ── bind mount ──→    /workspace   (WORKDIR)
~/.hermes           ── bind mount ──→    /opt/data    (config, .env, sessions)

$ hermes                                 (~/.local/bin/hermes = launcher)
     │
     └──> docker run --rm -it \
              -v ~/hermes-workspace:/workspace \
              -w /workspace \
              -v ~/.hermes:/opt/data \
              -e HERMES_UID=$(id -u) \
              -e HERMES_GID=$(id -g) \
              nousresearch/hermes-agent:latest "$@"
```

- `file` tool が `/etc/passwd` を書こうとしても、届くのは container 内の
  `/etc/passwd` (別物、rootfs は image layer で immutable、 hermes 非 root user
  権限外)。
- `terminal` tool も同 container 内 shell なので `rm -rf ~` は container 内の
  空 `$HOME` を消すだけで host には影響しない。
- host に永続化されるのは `/workspace` (= `~/hermes-workspace`) と
  `/opt/data` (= `~/.hermes`) の書き込みだけ。それ以外の host FS は container
  から**存在しない**扱い。

## 構成ロール

| 役割 | ロール | 補足 |
|---|---|---|
| Docker Engine + docker group | [`docker`](../roles/docker/) | Compose v2 plugin、dobachi を docker group に追加 |
| Hermes Agent (container + launcher) | [`hermes_agent`](../roles/hermes_agent/) | image pull + `~/.local/bin/hermes` 配置 + workspace 初期化 |

## 前提

- Ansible 2.9 以上
- Ubuntu 24.04 以降 (Debian 系)
- OpenRouter API key を Vault 化して渡せる状態
- NOPASSWD sudo (もしくは `--ask-become-pass`)

## Vault 準備

```bash
ansible-vault encrypt_string 'sk-or-v1-...' \
  --name 'vault_hermes_openrouter_api_key' \
  >> host_vars/k16/vault.yml
```

## プレイブック実行

```bash
ansible-playbook -i hosts playbooks/conf/linux/hermes.yml \
  -e server=k16 \
  -e hermes_agent_enable=true \
  --ask-vault-pass
```

これで:
- Docker Engine が起動、`dobachi` が docker group に入る
- `nousresearch/hermes-agent:latest` が pull される
- `~/hermes-workspace/` が作成される (0700)
- `~/.hermes/config.yaml` と `~/.hermes/.env` が bootstrap される
- `~/.local/bin/hermes` にラッパースクリプトが配置される

## 実行後の手動ステップ

### 1. docker group の反映

```bash
newgrp docker              # 現セッションに反映
# または一度ログアウトしてログイン
docker run --rm hello-world # 動作確認
```

### 2. PATH と launcher の確認

```bash
command -v hermes           # ~/.local/bin/hermes を指すはず
head -1 ~/.local/bin/hermes # shebang 確認
```

### 3. 動作確認

```bash
hermes doctor               # container 内で hermes doctor が走る
hermes config show          # provider: openrouter が反映されているか
```

### 4. workspace に触らせたいファイルを配置

```bash
# 例: 既存プロジェクトをコピー
cp -r ~/some-project ~/hermes-workspace/

# 例: リポジトリを clone
git clone https://github.com/... ~/hermes-workspace/some-repo

# 例: 一時的に別ディレクトリを触らせたい場合 (自己責任)
HERMES_WORKSPACE=/path/to/other-dir hermes
```

### 5. 対話開始

```bash
hermes
```

セッション内:
```
/model openrouter/anthropic/claude-sonnet-4.5
/model openrouter/qwen/qwen3-coder
```

## 推奨モデル (OpenRouter 経由)

| 用途 | モデル slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated list 掲載 |

## トラブルシューティング

### `docker: permission denied while trying to connect to the docker daemon`

現在のシェルに docker group がまだ反映されていない。`newgrp docker` を実行するか
一度ログアウト → ログイン。

### `error: workspace directory not found: ~/hermes-workspace`

Ansible が workspace を作る前に launcher を実行した、または手動で削除した。
```bash
mkdir -p ~/hermes-workspace
chmod 700 ~/hermes-workspace
```

### `hermes doctor` で `✗ OPENROUTER_API_KEY not set`

`~/.hermes/.env` に key が書かれていない。Vault 展開失敗の可能性が高い。
`--ask-vault-pass` を付けて ansible を再実行。

### tool call が発火しない

model slug が **agentic モデルでない**可能性。Hermes 側が「NOT agentic」と
警告するモデル (`hermes-3-*`, `hermes-4-*`) は使わない。Claude / GPT / Gemini
/ DeepSeek / Qwen3 系に切り替える。

### 別 provider を追加したい

container 内で `hermes setup` を対話実行するか、`hermes config set` を叩く:

```bash
hermes setup                                        # 対話ウィザード
hermes config set ANTHROPIC_API_KEY sk-ant-...      # 自動で .env 側に振り分け
```

これらの変更は host `~/.hermes/config.yaml` / `~/.hermes/.env` に永続化される。

## 非標準構成 (ローカル LLM)

本リポジトリは `roles/llama_server_deb` / `roles/amdgpu_gtt` を残しているが、
Hermes と組み合わせる用途は**推奨しない**:

- Hermes 自身が Hermes-3 / Hermes-4 を "NOT agentic" と警告
  (`hermes_cli/model_switch.py:53-58`)
- ローカルサイズ (7-14B) の agentic モデルは Hermes curated list に載っていない

`llama_server_deb` はテキスト補完/実験用途で単独で使う想定に留める。

## 関連ドキュメント

- [Hermes 公式 docs](https://hermes-agent.nousresearch.com/docs/)
- [Hermes 公式 Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker)
- [roles/hermes_agent/README.md](../roles/hermes_agent/README.md)
- [roles/docker/README.md](../roles/docker/README.md)
