# Hermes Agent セットアップ (Docker gateway mode, OpenRouter primary)

このドキュメントは、本リポジトリの Ansible ロールを使って **Hermes Agent CLI**
を Docker **gateway モード** で常駐させ、curator (skill 保守) と memory 更新を
24/7 で稼働させる手順をまとめたもの。

## 隔離モデル

エージェント本体は container 内で走り、host からは以下 2 ディレクトリだけを
bind mount で見せる:

```
[Host / systemd unit hermes-gateway.service]
    └─ docker run --name hermes nousresearch/hermes-agent gateway run
           ├─ curator (24/7 skill 保守)
           ├─ dashboard (opt-in, 127.0.0.1:9119)
           └─ hermes CLI (docker exec で attach)

[Host bind mounts]
  ~/.hermes            → /opt/data    (HERMES_HOME + workspace サブディレクトリ)
                         WORKDIR=/opt/data/workspace (触らせるファイル置き場)
  ~/.hermes           → /opt/data    (config, .env, skills, memories, sessions)
```

- `file` tool は container 内の `/opt/data` (= host `~/.hermes`) 配下しか触れない
  (rootfs は image layer で immutable、他 host FS は container から不可視)。
- `terminal` tool も同 container 内 shell で走るので `rm -rf ~` の類が host に
  波及しない。
- `$HOME/.ssh` などは container 側では**存在しない**扱い。触らせたいファイルは
  `~/.hermes/workspace/` にコピー / clone してから使う。

## Self-improvement (Hermes の売り) との整合

Hermes が売りにする「自己成長」— skills 生成、memory 更新、curator による
背景保守 — は gateway モードで完全に機能する:

| 要素 | 保存先 | 動作 |
| --- | --- | --- |
| Skills | `~/.hermes/skills/` | bind mount で永続化、curator が 24/7 更新 |
| Memory | `~/.hermes/memories/` | クロスセッションで更新 |
| Sessions / state.db | `~/.hermes/sessions/` | 永続化 |
| SOUL.md (persona) | `~/.hermes/SOUL.md` | 保持 |
| **Curator (background)** | container 内 s6-supervised | **常駐 → 24/7 稼働** |

短時間 `--rm` の ephemeral モードでは curator が session 終了で死ぬため、
本リポジトリは gateway モードを標準構成としている。

## 構成ロール

| 役割 | ロール |
|---|---|
| Docker Engine + docker group | [`docker`](../roles/docker/) |
| Hermes Agent (gateway container + wrappers) | [`hermes_agent`](../roles/hermes_agent/) |

## 前提

- Ansible 2.9 以上、`community.docker` collection
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
- `nousresearch/hermes-agent:latest` を pull
- `~/.hermes/workspace/` を 0700 で作成
- `~/.hermes/config.yaml` と `~/.hermes/.env` を bootstrap
- `/etc/systemd/system/hermes-gateway.service` を配置 → enable + start
- `~/.local/bin/hermes` (docker exec CLI ラッパー) 配置
- `~/.local/bin/hermes-shell` (docker exec bash helper) 配置

## 実行後の手動ステップ

### 1. docker group の反映 (初回のみ)

```bash
newgrp docker              # 現セッションに反映 (または一度ログアウト)
docker ps --filter name=hermes    # 常駐 container が動いているか
```

### 2. サービス状態確認

```bash
systemctl status hermes-gateway
journalctl -u hermes-gateway -f    # container ログを追う
```

### 3. 動作確認

```bash
hermes doctor                # container 内で hermes doctor
hermes config show           # provider: openrouter が反映されているか
```

### 4. workspace にファイルを置いて対話

```bash
git clone https://github.com/... ~/.hermes/workspace/example-repo

hermes                       # 対話開始
# セッション内:
#   /model openrouter/anthropic/claude-sonnet-4.5
#   /model openrouter/qwen/qwen3-coder
```

### 5. container 内 shell に潜る

```bash
hermes-shell                       # container 内 bash (workspace WORKDIR)
hermes-shell -c "ls /opt/data/skills"  # 1 コマンド実行
```

curator の稼働状態を眺めたい場合:
```bash
docker exec hermes ps aux | grep curator
```

## 手動制御

```bash
sudo systemctl stop hermes-gateway         # 停止
sudo systemctl start hermes-gateway        # 起動
sudo systemctl restart hermes-gateway      # 再起動
sudo systemctl disable hermes-gateway      # boot 時 auto-start 無効化
```

## 推奨モデル (OpenRouter 経由)

| 用途 | slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated 掲載 |

## トラブルシューティング

### `docker exec: no such container: hermes`

gateway container が起動していない。
```bash
sudo systemctl status hermes-gateway
sudo systemctl start hermes-gateway
```

### `error: gateway container 'hermes' is not running`

ラッパースクリプトのメッセージ。上記と同じ状況。

### `hermes doctor` で `✗ OPENROUTER_API_KEY not set`

Vault 展開失敗の可能性。`--ask-vault-pass` を付けて ansible 再実行。

### tool call が発火しない

Hermes-3 / Hermes-4 系モデル (`hermes-3-*`, `hermes-4-*`) は NOT agentic と
Hermes 自身が警告 (`hermes_cli/model_switch.py:53-58`)。Claude / GPT / Gemini
/ DeepSeek / Qwen3 系に切り替える。

### 別 provider を追加したい (Nous Portal / Anthropic 直)

container 内で:
```bash
hermes setup                                        # 対話ウィザード
hermes config set ANTHROPIC_API_KEY sk-ant-...      # 自動で .env に振り分け
```

これらの変更は host `~/.hermes/config.yaml` / `~/.hermes/.env` に永続化される。

### dashboard をブラウザから利用したい

Hermes 内蔵の dashboard は、sessions / tools / config / skills / memories を
Web UI で扱える。以下の 3 通りの公開方法があるが、いずれも **2026-06 hardening
以降、非 loopback bind は Basic auth (or OAuth) が必須** (`HERMES_DASHBOARD_INSECURE`
は無効化済み)。

#### A. localhost only + SSH tunnel

```yaml
# host_vars/<host>/main.yml
hermes_agent_dashboard_enable: true
# hermes_agent_dashboard_bind は default "127.0.0.1" のまま
```

```bash
# 手元マシンから:
ssh -L 9119:127.0.0.1:9119 dobachi@<host>
# ブラウザで:
http://localhost:9119
```

追加露出ゼロだが毎回 tunnel を張る必要あり。

#### B. Tailscale IP に bind (推奨、常時アクセス)

k16 の場合:
```yaml
# host_vars/k16/main.yml
hermes_agent_dashboard_enable: true
hermes_agent_dashboard_bind: "100.64.0.1"     # `tailscale ip -4` で確認
hermes_agent_dashboard_basic_auth_username: "dobachi"
hermes_agent_dashboard_basic_auth_password: "{{ vault_hermes_dashboard_password }}"
```

Vault 登録:
```bash
ansible-vault encrypt_string 'STRONG_PASSWORD_HERE' \
  --name 'vault_hermes_dashboard_password' \
  >> host_vars/k16/vault.yml
```

ansible 再実行後、Tailscale mesh 内のデバイスから:
```
http://100.64.0.1:9119
```
Basic auth プロンプトに `dobachi / <password>` を入力。

**Tailscale ACL** (Tailnet の管理コンソール) で `tag:owner` に限定するなど
アクセス制御をかけると更に堅牢。単一ユーザーの Tailnet なら追加設定不要。

#### C. 0.0.0.0 (非推奨)

全 interface に露出。Basic auth があっても brute force / ネット越し露出の
リスクが高い。ansible 実行時に警告を出すが、選ぶメリットは薄い。

### dashboard を無効化

```yaml
hermes_agent_dashboard_enable: false
```
ansible 再実行 → systemd unit 再生成 → container restart → dashboard 停止。

## 非標準構成 (ローカル LLM)

`roles/llama_server_deb` / `roles/amdgpu_gtt` は残しているが、Hermes と
組み合わせる用途は推奨しない (Nous 自身が Hermes-3/4 を NOT agentic と警告
しているため)。ローカル LLM は単独用途のみ。

## 関連ドキュメント

- [Hermes 公式 docs](https://hermes-agent.nousresearch.com/docs/)
- [Hermes 公式 Docker guide](https://hermes-agent.nousresearch.com/docs/user-guide/docker)
- [roles/hermes_agent/README.md](../roles/hermes_agent/README.md)
- [roles/docker/README.md](../roles/docker/README.md)
