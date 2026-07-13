hermes_agent
============

Nous Research の [Hermes Agent](https://hermes-agent.nousresearch.com/) CLI を
**Docker container 版** (`nousresearch/hermes-agent`) として per-user 導入し、
ラッパースクリプトを `~/.local/bin/` に配置するロール。

## 隔離モデル

エージェント本体 (venv, Node, Playwright, `file` / `terminal` / `browser` 等の
tool 実装) は container の内側で走る。host からは以下2ディレクトリだけを
bind mount で見せる:

```
[Host]                       [container]
~/hermes-workspace  ────→   /workspace   (WORKDIR)
~/.hermes           ────→   /opt/data    (config, API key, sessions)
```

`file` tool も `terminal` tool も上記2つの外を書き換えられない (container
rootfs は image layer で immutable、それ以外の host FS は container から見えない)。
`$HOME/.ssh`, `/etc/passwd`, `~/Sources/*` などの host ファイルは container
からは存在しない扱い。

**触らせたいファイルは `~/hermes-workspace/` にコピー / clone してから
`hermes` を起動する運用**が原則。`$PWD` 依存にしていないので、うっかり
`$HOME` から `hermes` を叩いて `~/.ssh` を露出させる事故が起きない。

## Requirements

- Ansible 2.9 以上
- Debian/Ubuntu (24.04 / 26.04)
- Docker Engine 動作中 (このロールを実行する前に `roles/docker` を通す)
- 対象ユーザー (`hermes_agent_user`) が `docker` グループに所属
- OpenRouter API key (`https://openrouter.ai/keys`) を Vault 暗号化して渡す

## Role Variables

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_enable` | `false` | このロールを実行するかのスイッチ |
| `hermes_agent_user` | 実行ユーザー | Hermes を入れる対象ユーザー |
| `hermes_agent_image` | `nousresearch/hermes-agent:latest` | 引く公式 image (tag pinning したいなら上書き) |
| `hermes_agent_workspace` | `~/hermes-workspace` | container の `/workspace` に bind mount する host ディレクトリ |
| `hermes_agent_workspace_subdirs` | `[]` | 初期化時に workspace 配下に掘るサブディレクトリ list |
| `hermes_agent_launcher_dir` | `~/.local/bin` | ラッパースクリプト設置先 |
| `hermes_agent_launcher_name` | `hermes` | ラッパースクリプト名 (別名にしたい場合 `hermes-sb` 等) |
| `hermes_agent_extra_docker_args` | `[]` | `docker run` 追加引数 (例: `["--network=host"]`) |
| `hermes_agent_openrouter_model` | `anthropic/claude-sonnet-4.5` | OpenRouter で primary として使うモデル slug |
| `hermes_agent_openrouter_api_key` | `""` | OPENROUTER_API_KEY (Vault 化した値を渡す) |
| `hermes_agent_reset_config` | `false` | true にすると `~/.hermes/config.yaml` を強制上書き |

設定ファイルは既定で初回 bootstrap (`force: false`) のみで配置する。
container 内で `hermes setup` / `hermes model` / `hermes config set` 経由で
加えた修正は再ロール実行で消えない (host `~/.hermes/config.yaml` = container
`/opt/data/config.yaml`)。

## Dependencies

`roles/docker` を playbook 内で先に流すこと (docker group への追加が必要)。

## Example Playbook

```yaml
- hosts: k16
  become: true
  vars_files:
    - "{{ inventory_dir }}/host_vars/k16/vault.yml"
  roles:
    - role: docker
      vars:
        docker_enable: true
        docker_users: ["dobachi"]
    - role: hermes_agent
      vars:
        hermes_agent_enable: true
        hermes_agent_openrouter_api_key: "{{ vault_hermes_openrouter_api_key }}"
```

Vault に API key を登録:

```bash
ansible-vault encrypt_string 'sk-or-v1-...' \
  --name 'vault_hermes_openrouter_api_key' \
  >> host_vars/k16/vault.yml
```

## 使い方 (実行後)

1. **docker group 反映** (初回のみ):
   ```bash
   newgrp docker      # 現セッションに反映 (or ログアウト → ログイン)
   docker run --rm hello-world
   ```

2. **PATH 確認**:
   ```bash
   command -v hermes         # /home/<user>/.local/bin/hermes を指す
   ```

3. **動作確認**:
   ```bash
   hermes doctor
   hermes config show        # provider: openrouter が反映
   ```

4. **触らせたいファイルを workspace へ**:
   ```bash
   cp -r ~/some-project ~/hermes-workspace/
   # or: git clone <url> ~/hermes-workspace/some-repo
   ```

5. **対話開始**:
   ```bash
   hermes
   # プロンプト内でモデル切替:
   /model openrouter/anthropic/claude-sonnet-4.5
   /model openrouter/qwen/qwen3-coder
   ```

## 環境変数による override

ラッパースクリプトが読む環境変数:

| 変数 | 用途 |
| --- | --- |
| `HERMES_WORKSPACE` | 一時的に workspace を別ディレクトリに切替 (`HERMES_WORKSPACE=/tmp/work hermes`) |
| `HERMES_IMAGE` | image tag 差替 |
| `HERMES_EXTRA_DOCKER_ARGS` | `docker run` に追加引数 (word split, 単純用途のみ) |

## 推奨モデル (OpenRouter 経由)

| 用途 | モデル slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated list 掲載 |
| コンパクション (aux) | `google/gemini-flash-*` | 安価・高速 |

## License

BSD-3-Clause

## Author

dobachi
