# Hermes Agent + ローカル LLM セットアップ

このドキュメントは、本リポジトリの Ansible ロール群を使って **Hermes Agent CLI**
を「ローカル LLM (llama-server) + Anthropic Claude (任意)」のハイブリッド構成で
立ち上げる手順をまとめたもの。

ローカル推論基盤 (`llama.cpp` / `Ollama` / `Open WebUI` / `ComfyUI` 等) を
組み合わせた包括的な AMD ローカル AI 環境のセットアップは
[`docs/local_ai.md`](./local_ai.md) を参照。本ドキュメントはその上に乗る
**エージェント CLI レイヤー** (`hermes_agent` ロール + 周辺) を扱う。

## アーキテクチャ

```
[Hermes Agent CLI] ── OpenAI 互換 HTTP ──> [llama-server :8080]
   per-user                                  systemd (Ubuntu 26 標準 apt)
                                             Vulkan backend → AMD iGPU/dGPU
                                                  ↑
                                             amdgpu_gtt で GTT 拡張
                                             (14B クラスを iGPU で動かす用)

[Hermes Agent CLI] ── terminal tool ──> [Docker]
   sandbox 隔離 (TERMINAL_ENV=docker)
```

## 構成ロール

| 役割 | ロール | 補足 |
|---|---|---|
| Hermes Agent CLI (per-user) | [`hermes_agent`](../roles/hermes_agent/) | `~/.hermes/config.yaml` 配置 |
| ローカル LLM サーバ | [`llama_server_deb`](../roles/llama_server_deb/) | Ubuntu 26 `llama.cpp-tools` + Vulkan backend |
| GTT メモリ拡張 | [`amdgpu_gtt`](../roles/amdgpu_gtt/) | iGPU で 14B を動かす場合に必要 |
| サンドボックス | [`docker`](../roles/docker/) | Hermes の `terminal` ツール用 (Compose v2 plugin) |

## 前提

### ハードウェア

- AMD iGPU (Radeon 680M = gfx1035) または Vulkan 対応 GPU
- 32 GB RAM 以上 (14B Q4_K_M を 20 GB GTT で動かす想定)

### OS

- **Ubuntu 26.04 (resolute)** を主対象 (24.04 以前は `llama.cpp` apt パッケージ
  が無く、`llama_server_deb` が使えない)

### Ansible 制御ノード

- Ansible 2.9 以上
- 対象ホストへの SSH (もしくは `localhost`)
- 対象ホストの `sudo` 権限 (NOPASSWD なら `-K` 不要)

---

## 1. プレイブック実行

### 1.1 Hermes 一式 (LLM + GTT + Hermes Agent)

[`playbooks/conf/linux/hermes.yml`](../playbooks/conf/linux/hermes.yml) を実行:

```bash
# 全部入り (初回)
ansible-playbook -i hosts playbooks/conf/linux/hermes.yml \
  -e server=localhost \
  -e amdgpu_gtt_enable=true \
  -e llama_server_deb_enable=true \
  -e hermes_agent_enable=true

# GTT 拡張済みなら amdgpu_gtt は不要
ansible-playbook -i hosts playbooks/conf/linux/hermes.yml \
  -e server=localhost \
  -e llama_server_deb_enable=true \
  -e hermes_agent_enable=true
```

`amdgpu_gtt_enable=true` を有効にした場合、ロールは `/etc/default/grub` を
書き換えて `update-grub` まで実行する (handler は `lineinfile` 直後で
`meta: flush_handlers` するので、後続ロールが失敗しても grub.cfg は更新される)。
**反映には再起動が必須**。

### 1.2 Docker (Hermes の terminal ツール用)

[`playbooks/conf/linux/docker.yml`](../playbooks/conf/linux/docker.yml) を実行:

```bash
ansible-playbook -i hosts playbooks/conf/linux/docker.yml \
  -e server=localhost \
  -e docker_enable=true \
  -e docker_users='[dobachi]'
```

- `docker_users` は string / YAML flow / カンマ区切り / 本物のリストを受け付ける
  (ロール内で list に正規化)
- `docker_codename` は既定 `noble`。Ubuntu 26.04 (resolute) は 2026-06 時点で
  Docker 公式の `download.docker.com/linux/ubuntu/dists/` に未提供のため LTS の
  noble にフォールバックしている

---

## 2. 手動ステップ

### 2.1 GTT 反映のため再起動 (amdgpu_gtt_enable=true で初回適用したとき)

```bash
sudo reboot

# 再起動後に確認
cat /proc/cmdline | grep gttsize    # amdgpu.gttsize=20480 が見えれば OK
```

### 2.2 Docker グループ反映

ansible で `docker_users: [dobachi]` を指定して `usermod -aG docker` 済みでも、
**現セッションには反映されない**。以下のいずれかが必要:

```bash
# 現セッションで反映
newgrp docker

# もしくは一度ログアウト → ログインし直し
```

確認:

```bash
groups dobachi | tr ' ' '\n' | grep docker
docker run --rm hello-world
```

### 2.3 Hermes Agent 補助修復

```bash
hermes doctor --fix      # ~/.local/bin/hermes シンボリックリンク作成 等
```

### 2.4 Claude (Anthropic) 認証 — ハイブリッド構成の場合のみ

ローカル LLM だけで遊ぶならスキップ可。Claude も併用したい場合:

```bash
# 対話 (推奨: OAuth or API key を選択)
hermes setup

# もしくは環境変数
echo 'export ANTHROPIC_API_KEY=sk-ant-...' >> ~/.bashrc
source ~/.bashrc
```

### 2.5 動作確認

```bash
# llama-server (ローカル LLM) が active か
systemctl status llama-server
curl http://127.0.0.1:8080/v1/models   # qwen2.5-coder-14b が見える

# Hermes 健全性
hermes doctor                          # ✗ が出ていないか

# Hermes セッション起動 → モデル切替
hermes
# プロンプト内で:
/model qwen2.5-coder-14b                  # ローカル 14B (provider プレフィクス無しでOK)
/model claude-sonnet-4-6                  # Claude (hermes setup 済みなら)
```

---

## 3. トラブルシューティング

### 3.1 再起動したのに `cat /proc/cmdline | grep gttsize` が空

`/etc/default/grub` 側に `amdgpu.gttsize=N` が入っているのに `/boot/grub/grub.cfg`
が古い場合、過去にロールが古い実装で動いて `update-grub` がフラッシュされずに
再起動した可能性がある (現行 `roles/amdgpu_gtt` は `meta: flush_handlers` 入り
なので発生しないはず)。

```bash
# 状況確認
stat /etc/default/grub /boot/grub/grub.cfg     # mtime の前後関係

# 修復
sudo update-grub && sudo reboot
```

### 3.2 `ansible-playbook ... -e docker_users='[dobachi]'` が loop エラー

旧版 (2026-06-07 以前) の `roles/docker` では `-e` の値が文字列扱いされて
`loop` が落ちていた。現行ロールは内部で list に正規化するので発生しないはず。

それでも厳密に list を渡したい場合は JSON dict 形式で:

```bash
-e '{"docker_users":["dobachi"]}'
```

### 3.3 `llama-server` の初回起動が遅い / `journalctl` に進捗

`llama_server_deb` は HF (`bartowski/Qwen2.5-Coder-14B-Instruct-GGUF`) から GGUF
を初回起動時に取りに行く (~9 GB)。systemd 経由なので非対話で進行する。

```bash
journalctl -u llama-server -f          # 進捗を見る
```

### 3.4 `hermes doctor` で `✗ docker not found`

`roles/docker` をまだ流していない or docker.service が起動していない。`docker
version` でクライアント・サーバ両方が見えることを確認 (Server 側が missing なら
`systemctl status docker` で診断)。

### 3.5 `Failed to initialize agent: Model ... has a context window of 32,768 tokens`

Hermes Agent はモデルに **最低 64K context** を要求する。`hermes model` で
Custom OpenAI 互換 endpoint を auto-detect 登録すると、Hermes は llama-server の
`/v1/models` から `n_ctx_train` を拾うが、Qwen2.5-Coder の素の値は 32768 で
要件に満たない。`roles/hermes_agent` は `model.context_length: 65536` を
config に明示するのでこのエラーは出ないが、CLI から登録した場合は手で追加が
必要:

```yaml
model:
  provider: custom
  default: qwen2.5-coder-14b
  base_url: http://127.0.0.1:8080/v1
  api_key: dummy
  context_length: 65536    # ← これを足す
```

(llama-server 側は `roles/llama_server_deb` が `--ctx-size 65536` で起動して
RoPE 延長しているので、Hermes 側はその値を信用するだけで良い。)

### 3.6 Hermes が長文セッションで挙動がおかしい

ローカルモデル (Qwen2.5-Coder-14B) は本来 `n_ctx_train=32768` だが、
`llama_server_deb_ctx_size: 65536` で起動するので **RoPE extrapolation** で
context を伸ばしている。長文タスクで品質が落ちる場合は ctx_size を 32K に
落とすか、より長 context な GGUF (YaRN 等で 128K 化された版) を選ぶ。

---

## 関連ドキュメント

- [docs/local_ai.md](./local_ai.md) — AMD ローカル AI 環境セットアップ
  (llama.cpp / Ollama / Open WebUI / ComfyUI / whisper.cpp)
- [roles/hermes_agent/README.md](../roles/hermes_agent/README.md)
- [roles/llama_server_deb/README.md](../roles/llama_server_deb/README.md)
- [roles/amdgpu_gtt/README.md](../roles/amdgpu_gtt/README.md)
- [roles/docker/README.md](../roles/docker/README.md)
