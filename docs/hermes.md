# Hermes Agent セットアップ (OpenRouter primary)

このドキュメントは、本リポジトリの Ansible ロールを使って **Hermes Agent CLI**
を **OpenRouter primary** で立ち上げる手順をまとめたもの。

## 方針: なぜ OpenRouter primary か

Hermes Agent 公式は「使いたい provider を選んでね」というスタンスだが、
ソースと docs を読むと以下が浮かぶ:

- Hermes Agent は **Hermes-3 / Hermes-4 を "NOT agentic" と警告している**
  (`hermes_cli/model_switch.py:53-58`)。Nous 自身の Hermes チャットモデルは
  tool-calling 訓練されておらず、Hermes Agent の想定外モデル。
- 公式 docs の "Easiest path" は `hermes setup --portal` (Nous Portal OAuth)。
  ただし Portal はサブスクリプション前提。
- OpenRouter は Hermes の **first-class provider**
  (`hermes_cli/providers.py:47` に `HermesOverlay` 登録済み) で、200+ モデルを
  単一 API key で切替えられる。Anthropic / Google / DeepSeek / Qwen3 / Kimi など
  agentic モデルが揃う。

以上から、コスト・柔軟性・公式サポート度のバランスが取れた OpenRouter primary
を標準構成とし、ローカル LLM (llama-server + AMD iGPU) は非標準の実験用途として
分離している。

## アーキテクチャ

```
[Hermes Agent CLI] ── HTTPS ──> [OpenRouter API]  (200+ agentic models)
   per-user                       provider: openrouter (Hermes overlay)
                                  API key: ~/.hermes/.env

[Hermes Agent CLI] ── terminal tool ──> [Docker]
   sandbox 隔離 (TERMINAL_ENV=docker)
```

## 構成ロール

| 役割 | ロール | 補足 |
|---|---|---|
| Hermes Agent CLI (per-user) | [`hermes_agent`](../roles/hermes_agent/) | `~/.hermes/config.yaml` + `~/.hermes/.env` 配置 |
| サンドボックス | [`docker`](../roles/docker/) | Hermes の `terminal` ツール用 (Compose v2 plugin) |

## 前提

- Ansible 2.9 以上
- Ubuntu 24.04 以降 (Debian 系)
- OpenRouter API key (`https://openrouter.ai/keys` で発行 → Vault 化)
- NOPASSWD sudo (もしくは `--ask-become-pass`)

## Vault 準備

```bash
# OpenRouter API key を Vault に登録
ansible-vault encrypt_string 'sk-or-v1-...' \
  --name 'vault_hermes_openrouter_api_key' \
  >> host_vars/k16/vault.yml
```

## プレイブック実行

[`playbooks/conf/linux/hermes.yml`](../playbooks/conf/linux/hermes.yml) を実行:

```bash
# k16 (host_vars 済み) — model slug と API key は host_vars で永続化済み
ansible-playbook -i hosts playbooks/conf/linux/hermes.yml \
  -e server=k16 \
  -e hermes_agent_enable=true \
  --ask-vault-pass
```

テンプレを変えた後に config.yaml へ反映したい場合は
`-e hermes_agent_reset_config=true` を追加。

## Docker (Hermes の terminal ツール用)

Hermes の `terminal` ツールを Docker sandbox で動かしたい場合は別プレイブック:

```bash
ansible-playbook -i hosts playbooks/conf/linux/docker.yml \
  -e server=localhost \
  -e docker_enable=true \
  -e docker_users='[dobachi]'
```

## 動作確認

```bash
hermes doctor                  # Hermes 健全性チェック (問題があれば ✗)
hermes config show             # provider: openrouter が反映されているか
hermes                         # 起動して会話 → tool call が実行されるか
```

セッション内でモデル切替:

```
/model openrouter/anthropic/claude-sonnet-4.5
/model openrouter/qwen/qwen3-coder
/model openrouter/deepseek/deepseek-v3.2
```

推奨モデル (OpenRouter 経由):

| 用途 | モデル slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated list 掲載 |

## トラブルシューティング

### `hermes doctor` で `✗ OPENROUTER_API_KEY not set`

`~/.hermes/.env` に key が書かれていない。Vault が展開されていないケースが多い。
`--ask-vault-pass` または `--vault-password-file` を付けて再実行。

### `HTTP 401 Unauthorized`

- API key が無効 or 使い切り → OpenRouter dashboard で credit 確認
- `.env` に BOM や余計な空白が入っていないか: `cat -A ~/.hermes/.env`

### tool call が発火しない

model slug が **agentic モデルでない**可能性。以下の regex にマッチするモデル名は
Hermes 自身が「NOT agentic」と警告する (`hermes_cli/model_switch.py:69-72`):

- `hermes-3-*`
- `hermes-4-*`
- `hermes3` / `hermes 3` / `hermes 4`

Claude / GPT / Gemini / DeepSeek / Qwen3 系に切り替える。

### 別 provider を追加したい (Nous Portal / Anthropic 直 / ローカル)

本ロールは `model:` ブロック + `OPENROUTER_API_KEY` のみ管理する。
追加の provider は `hermes setup` か `hermes model` で対話追加:

```bash
hermes setup          # OAuth / API key の対話ウィザード
hermes model          # provider + default model の picker
```

`hermes config set` で dotted-key を直接編集する経路もある:

```bash
hermes config set model.default openrouter/qwen/qwen3-coder
hermes config set ANTHROPIC_API_KEY sk-ant-...   # 自動で .env に振り分け
```

## 非標準構成 (ローカル LLM)

本ドキュメントは扱わない。AMD iGPU + llama-server で自前推論したい場合は
`roles/llama_server_deb` / `roles/amdgpu_gtt` を opt-in で追加し、
`hermes setup` から custom endpoint を登録する。ただし Hermes-3 系モデルは
`is_nous_hermes_non_agentic` regex で検出されて警告される点に注意。

## 関連ドキュメント

- [docs/local_ai.md](./local_ai.md) — AMD ローカル AI 環境セットアップ (参考)
- [roles/hermes_agent/README.md](../roles/hermes_agent/README.md)
- [roles/docker/README.md](../roles/docker/README.md)
