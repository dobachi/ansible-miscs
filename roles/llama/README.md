llama
=====

Installs llama.cpp prebuilt release binaries into `~/Llama/{{ llama_version }}/`.
バックエンド (CPU / Vulkan / ROCm) を `llama_backend` で切替可能。

バックエンド
------------

| `llama_backend` | アセット | 用途 |
| --- | --- | --- |
| `cpu` (default) | `llama-<ver>-bin-ubuntu-x64.zip` | 既存挙動。GPU 無し / CPU のみ |
| `vulkan` | `llama-<ver>-bin-ubuntu-vulkan-x64.zip` | AMD/Intel/NVIDIA GPU を Mesa RADV / Vulkan 経由で利用 (AMD iGPU 含む) |
| `rocm` | (公式アセット無し) | 本ロールでは `fail`。`https://github.com/lemonade-sdk/llamacpp-rocm` のビルドを使うか、ソースからビルド |

**互換性**: `llama_backend: cpu` をデフォルトとして残しているため、
既存の `playbooks/conf/linux/llama.yml` 等の挙動は変更されない。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt) — Vulkan ビルド時のみ `libvulkan1` / `mesa-vulkan-drivers` を追加導入
- `become: yes`
- 先行ロール: `register_home`

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `llama_version` | `b5604` | 対象 llama.cpp リリースタグ |
| `llama_backend` | `cpu` | `cpu` / `vulkan` / `rocm` |
| `llama_install_server_unit` | `false` | `llama-server` の systemd ユニットを配置するか |
| `llama_server_host` | `127.0.0.1` | `--host` |
| `llama_server_port` | `8081` | `--port` |
| `llama_server_model_path` | `""` | unit を有効化する場合は GGUF の絶対パス必須 |
| `llama_server_n_gpu_layers` | `99` | `--n-gpu-layers` (Vulkan/ROCm のとき有効) |
| `llama_server_extra_args` | `""` | 追加引数 |
| `llama_server_hsa_override_gfx_version` | `10.3.0` | rocm バックエンドのとき unit 内に注入 |

Dependencies
------------

None (apt 経由の Vulkan 依存は本ロール内で扱う)

Example Playbook
----------------

```yaml
# CPU (既存と同じ挙動)
- hosts: localhost
  become: yes
  roles:
    - register_home
    - llama
```

```yaml
# Vulkan ビルドを取得 (AMD iGPU 推奨)
- hosts: localhost
  become: yes
  roles:
    - register_home
    - role: llama
      vars:
        llama_backend: vulkan
```

```yaml
# llama-server を systemd で常駐させる
- hosts: localhost
  become: yes
  roles:
    - register_home
    - role: llama
      vars:
        llama_backend: vulkan
        llama_install_server_unit: true
        llama_server_model_path: /home/user/Models/qwen2.5-3b-instruct-q4_k_m.gguf
```

既知の上流問題: RUNPATH のハードコード
--------------------------------------

llama.cpp の公式プリビルド zip は、バイナリの RUNPATH に GitHub Actions の
ビルドホストパス (`/home/runner/work/llama.cpp/llama.cpp/build/bin`) が
そのまま焼き込まれている。そのため展開直後の `llama-cli` は

```
error while loading shared libraries: libllama.so: cannot open shared object file: ...
```

で起動失敗する。本ロールは `patchelf --set-rpath '$ORIGIN'` で各バイナリ
および `lib*.so` の RUNPATH を「自分と同じディレクトリ」に書き換えるタスクを
unarchive 直後に実行する。**過去に旧版で展開済みだった場合**は手動で:

```bash
sudo apt-get install -y patchelf
cd ~/Llama/b5604/build/bin
for f in llama-* lib*.so; do patchelf --set-rpath '$ORIGIN' "$f"; done
```

動作確認
--------

```bash
~/Llama/b5604/build/bin/llama-cli --version

# Vulkan で AMD GPU が見えているか
vulkaninfo --summary | grep -i radeon

# Vulkan バックエンドで簡易推論
~/Llama/b5604/build/bin/llama-cli -m <gguf> -ngl 99 --device Vulkan0 -p "hello"

# systemd ユニット稼働確認
systemctl status llama-server
curl -s http://127.0.0.1:8081/health
```

Tags
----

- `llama`

License
-------

BSD
