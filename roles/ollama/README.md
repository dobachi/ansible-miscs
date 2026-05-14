ollama
======

公式 install.sh で [Ollama](https://ollama.com/) を導入し、
**サポート外 AMD GPU を認識させる systemd drop-in** を配置するロール。
モデルの初期 pull もオプションで実施。

Vulkan 経路が無いことの注意
---------------------------

Ollama は **Vulkan バックエンドを持たない** (CPU か ROCm のみ)。
よって本ロールは "Vulkan 優先" の例外: AMD GPU で GPU 推論したい場合は
`roles/rocm` を併用 (ROCm ランタイムを入れる) し、本ロールが drop-in で
`HSA_OVERRIDE_GFX_VERSION` を注入する。

`rocm_install: false` 環境では Ollama は **CPU 推論にフォールバック**する。
3B クラス量子化モデル (`llama3.2:3b`, `qwen2.5:3b` 等) なら実用速度が出る。

systemd drop-in の配置
----------------------

公式 install.sh が `/etc/systemd/system/ollama.service` を作る。本ロールは
そのユニットを **書き換えず**、`/etc/systemd/system/ollama.service.d/amd-igpu.conf`
を別ファイルとして追加する。これにより、Ollama アップデートでユニットが
再生成されても override は維持される。

```ini
[Service]
Environment="OLLAMA_HOST=127.0.0.1:11434"
Environment="HSA_OVERRIDE_GFX_VERSION=10.3.0"
Environment="HSA_ENABLE_SDMA=0"
Environment="OLLAMA_KEEP_ALIVE=10m"
```

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- GPU 推論を使うなら **`roles/rocm` を先に有効化** (`rocm_install: true`) しておくこと

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `ollama_install` | `true` | 全タスクのトップレベル ON/OFF |
| `ollama_version` | `""` | 空なら upstream 最新。`"0.5.4"` 等でピン留め |
| `ollama_host` | `"127.0.0.1:11434"` | バインド先 |
| `ollama_use_rocm` | `true` | drop-in に HSA_OVERRIDE を入れる |
| `ollama_hsa_override_gfx_version` | `"10.3.0"` | gfx1030 偽装 |
| `ollama_extra_env` | `{HSA_ENABLE_SDMA: "0", OLLAMA_KEEP_ALIVE: "10m"}` | drop-in に追加する env |
| `ollama_models` | `[llama3.2:3b, qwen2.5:3b]` | 初期 pull するモデル名一覧 |
| `ollama_service_user` | `"ollama"` | install.sh が作るサービスユーザ |
| `ollama_service_user_groups` | `[render, video]` | サービスユーザに append するグループ |

Dependencies
------------

None (ROCm が必要な場合は別途 `roles/rocm` を ON に)

Example Playbook
----------------

```yaml
# Vulkan 一式 + Ollama を CPU で
- hosts: localhost
  become: yes
  vars:
    ollama_use_rocm: false
    ollama_models: []   # 初期 pull スキップ
  roles:
    - ollama
```

```yaml
# AMD iGPU + ROCm 経由で GPU 推論
- hosts: localhost
  become: yes
  vars:
    rocm_install: true
  roles:
    - amd_gpu_base
    - rocm
    - ollama
```

動作確認
--------

```bash
systemctl status ollama
systemctl show ollama -p Environment | tr ' ' '\n' | grep -E 'HSA_|OLLAMA_'
journalctl -u ollama -n 100 | grep -iE 'rocm|gpu|compatible'

# API
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:11434/api/tags | head

# 推論
ollama run llama3.2:3b "こんにちは"
```

Tags
----

- `ollama`

License
-------

BSD
