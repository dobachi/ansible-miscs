hermes_agent
============

Nous Research の [Hermes Agent](https://hermes-agent.nousresearch.com/) を
**Docker gateway mode** で常駐させ、per-user のラッパー/helper を
`~/.local/bin/` に配置するロール。

## アーキテクチャ

```
[Host / systemd unit hermes-gateway.service]
    └─ docker run --name hermes nousresearch/hermes-agent gateway run
           ├─ curator (24/7 skill 保守)
           ├─ dashboard (opt-in, 127.0.0.1:9119)
           ├─ messaging gateway (opt-in)
           └─ hermes CLI (docker exec で attach)

[Host bind mounts]
  ~/.hermes            → /opt/data    (HERMES_HOME、workspace サブディレクトリを含む)
  ~/.hermes           → /opt/data    (config, .env, skills, memories, sessions)

[Host wrappers ~/.local/bin/]
  hermes         → docker exec で container 内の hermes CLI
  hermes-shell   → docker exec で container 内 bash
```

## 隔離モデル

- container 内の `file` / `terminal` / `browser` tool は上記 2 bind mount の
  外を触れない (rootfs は image layer で immutable、他 host FS は container
  から不可視)。
- `$HOME/.ssh` や `/etc/passwd` などは container 側から**存在しない**扱い。
- 触らせたいファイルは `~/.hermes/workspace/` にコピー / clone してから使う。

## Self-improvement との関係

Hermes の売りである「自己成長」(skills 生成、memory 更新、curator による
背景保守) は gateway mode で完全に機能する:

- **Skills** (`~/.hermes/skills/`) — bind mount で永続化。curator が 24/7 更新
- **Memory** (`~/.hermes/memories/`) — bind mount 経由でクロスセッション更新
- **Sessions / state.db** — bind mount 経由で永続化
- **SOUL.md** (persona) — bind mount 経由で保持
- **Curator** — 常駐 container 内で s6-supervised、常時稼働

短時間 `--rm` の ephemeral モードでは curator の 24/7 保守が働かないため、
このロールは gateway mode を既定とする。

## Requirements

- Ansible 2.9 以上、`community.docker` collection
- Debian/Ubuntu (24.04 / 26.04)
- Docker Engine 動作中 (`roles/docker` を先に流すこと)
- 対象ユーザーが docker group に所属 (usermod 直後は `newgrp docker` or 再ログイン)
- OpenRouter API key を Vault 化して渡す

## Role Variables

### 基本

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_enable` | `false` | このロールを実行するかのスイッチ |
| `hermes_agent_user` | 実行ユーザー | 対象ユーザー (~/.hermes / workspace 所有者) |
| `hermes_agent_image` | `nousresearch/hermes-agent:latest` | 公式 image |
| `hermes_agent_container_name` | `hermes` | 常駐 container の名前 |
| `hermes_agent_workspace` | `~/.hermes/workspace` | container 内 WORKDIR (`/opt/data/workspace`) にマップされる host ディレクトリ。HERMES_HOME 配下なら追加 bind mount 不要 |
| `hermes_agent_workspace_subdirs` | `[]` | workspace 配下に掘るサブディレクトリ list |

### wrappers / systemd

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_launcher_dir` | `~/.local/bin` | ラッパー設置先 |
| `hermes_agent_launcher_name` | `hermes` | CLI ラッパー名 (`docker exec` 経由で container 内 hermes) |
| `hermes_agent_shell_helper_name` | `hermes-shell` | bash helper 名 (`docker exec bash`) |
| `hermes_agent_systemd_unit_name` | `hermes-gateway.service` | systemd unit ファイル名 |
| `hermes_agent_systemd_unit_enabled` | `true` | boot 時 auto-start |
| `hermes_agent_systemd_unit_state` | `started` | ansible 実行時最終状態 |

### 公開ポート (opt-in、いずれも 127.0.0.1 bind)

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_dashboard_enable` | `false` | 内蔵 dashboard を有効化 |
| `hermes_agent_dashboard_port` | `9119` | dashboard port |
| `hermes_agent_dashboard_bind` | `127.0.0.1` | host 側 bind IP (Tailscale IP 可、非 loopback は Basic auth 必須) |
| `hermes_agent_dashboard_basic_auth_username` | `""` | Basic auth ユーザ名 (非 loopback bind で必須) |
| `hermes_agent_dashboard_basic_auth_password` | `""` | Basic auth パスワード (Vault 化推奨) |
| `hermes_agent_api_server_enable` | `false` | OpenAI 互換 API server を 127.0.0.1:8642 で公開 |
| `hermes_agent_api_server_port` | `8642` | API server port |
| `hermes_agent_api_server_key` | `""` | API server 認証キー (`_enable=true` なら必須) |

### provider

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_openrouter_model` | `anthropic/claude-sonnet-4.5` | OpenRouter モデル slug |
| `hermes_agent_openrouter_api_key` | `""` | OPENROUTER_API_KEY (Vault 化した値を渡す) |
| `hermes_agent_reset_config` | `false` | true で `~/.hermes/config.yaml` を強制上書き |
| `hermes_agent_extra_docker_args` | `[]` | `docker run` 追加引数 |

## Dependencies

`roles/docker` を playbook 内で先に流すこと。

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

Vault 登録:
```bash
ansible-vault encrypt_string 'sk-or-v1-...' \
  --name 'vault_hermes_openrouter_api_key' \
  >> host_vars/k16/vault.yml
```

## 使い方 (実行後)

```bash
# サービス状態
systemctl status hermes-gateway
docker ps --filter name=hermes

# 対話 CLI (container 内 hermes に attach)
hermes                        # 対話開始
hermes doctor                 # 健全性チェック
hermes config show            # 設定確認

# container 内 bash に落ちる (デバッグ / 検査)
hermes-shell
hermes-shell -c "ls /opt/data/workspace"

# 触らせたいファイル配置
cp -r ~/some-project ~/.hermes/workspace/
git clone <url> ~/.hermes/workspace/example
```

### 手動制御

```bash
sudo systemctl stop hermes-gateway         # 一時停止
sudo systemctl start hermes-gateway        # 起動
sudo systemctl restart hermes-gateway      # 再起動
sudo systemctl disable hermes-gateway      # boot 時 auto-start 無効化
```

## 環境変数による override

wrapper が読む環境変数:

| 変数 | 用途 |
| --- | --- |
| `HERMES_CONTAINER` | 別名 container に接続 (複数 profile を並走させる場合) |

## 推奨モデル (OpenRouter 経由)

| 用途 | slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated 掲載 |

## License

BSD-3-Clause

## Author

dobachi
