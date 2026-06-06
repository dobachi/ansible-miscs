llama_server_deb
================

Ubuntu 26.04 (resolute) universe の `llama.cpp-tools` パッケージで
`llama-server` を system service として常駐させるロール。
Vulkan backend は `libggml0-backend-vulkan` (dlopen される) を入れるだけで
有効化される。**ソースからビルドしない**。

OpenAI 互換 API (`http://127.0.0.1:8080/v1`) を提供するので、
そのまま [`roles/hermes_agent`](../hermes_agent/) や他の OpenAI 互換
クライアントから使える。

既存の [`roles/llama`](../llama/) (公式 release zip を `~/Llama/` に展開する
ユーザーローカル版) とは独立。両方の併存も可能 (ポートだけ衝突しないようにする)。

Requirements
------------

- Ansible 2.9 以上
- **Ubuntu 26.04 (resolute) 以降**
  - 24.04 (noble) には `llama.cpp` パッケージは無いので使えない
- Vulkan 対応 GPU (AMD iGPU / dGPU、Intel Arc、NVIDIA 等)
  - AMD iGPU で 14B クラスを動かす場合は別途 [`roles/amdgpu_gtt`](../amdgpu_gtt/) で
    GTT メモリを拡張すること
- ネット接続 (HF からのモデル DL)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `llama_server_deb_enable` | `false` | このロールを実行するかのスイッチ |
| `llama_server_deb_enable_unit` | `true` | パッケージ同梱の `llama-server.service` を enable/start するか |
| `llama_server_deb_host` | `127.0.0.1` | 待ち受けアドレス。`0.0.0.0` で外部公開 |
| `llama_server_deb_port` | `8080` | 待ち受けポート |
| `llama_server_deb_ctx_size` | `65536` | context size。Hermes Agent は 64K 以上を要求 |
| `llama_server_deb_n_gpu_layers` | `99` | GPU offload layer 数。0 で CPU フォールバック |
| `llama_server_deb_jinja` | `true` | Jinja tool calling テンプレート。Hermes ローカル LLM に必須 |
| `llama_server_deb_hf_repo` | `bartowski/Qwen2.5-Coder-14B-Instruct-GGUF` | HF repo (auto DL) |
| `llama_server_deb_hf_file` | `Qwen2.5-Coder-14B-Instruct-Q4_K_M.gguf` | repo 内の GGUF ファイル名 |
| `llama_server_deb_model_path` | `""` | 設定するとローカル GGUF 直指定 (HF より優先) |
| `llama_server_deb_model_alias` | `qwen2.5-coder-14b` | API が返す model id |
| `llama_server_deb_extra_env` | `{}` | `/etc/default/llama-server` に追記したい環境変数 dict |

`llama_server_deb_hf_repo` / `llama_server_deb_model_path` が両方空だと
assert で弾く (起動できないため)。

Dependencies
------------

なし。Ubuntu 26.04 標準 repo の `llama.cpp`, `llama.cpp-tools`,
`libggml0-backend-vulkan` を apt 経由で導入する。

Example Playbook
----------------

```yaml
- hosts: k16
  become: true
  roles:
    - { role: llama_server_deb, llama_server_deb_enable: true }
```

実行後の動作確認:

```bash
systemctl status llama-server
journalctl -u llama-server -f          # 初回は HF から GGUF DL に時間がかかる
curl http://127.0.0.1:8080/v1/models   # モデル alias が見える
```

カスタマイズ例 (7B モデルに切替 + Jinja off):

```yaml
- role: llama_server_deb
  llama_server_deb_enable: true
  llama_server_deb_hf_repo: "bartowski/Qwen2.5-Coder-7B-Instruct-GGUF"
  llama_server_deb_hf_file: "Qwen2.5-Coder-7B-Instruct-Q4_K_M.gguf"
  llama_server_deb_model_alias: "qwen2.5-coder-7b"
  llama_server_deb_jinja: false
```

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
