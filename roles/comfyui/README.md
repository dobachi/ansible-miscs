comfyui
=======

[ComfyUI](https://github.com/comfyanonymous/ComfyUI) (ノードベース Stable
Diffusion ワークフロー UI) を **venv + ROCm PyTorch + system systemd** で
導入するロール。AMD GPU での画像生成のための一式。

なぜ ROCm が必要か
------------------

ComfyUI / Stable Diffusion は PyTorch を使う。**PyTorch には Vulkan の
production バックエンドが無い**ため、AMD GPU で動かすには ROCm wheel
(`https://download.pytorch.org/whl/rocm6.x`) が事実上唯一の選択肢。
よって `roles/rocm` を `rocm_install: true` で先に適用する必要がある。

iGPU (Radeon 680M) の VRAM 制約
-------------------------------

- iGPU の VRAM は BIOS の UMA frame buffer 設定で決まる。
  **SDXL (~7 GB) を回したい場合は UMA = 4 GiB 以上を BIOS で設定**。
- それでも厳しいなら `--use-pytorch-cross-attention` (既定で付与) +
  SD1.5 系モデルで運用するのが現実的。
- 既定で `PYTORCH_HIP_ALLOC_CONF=garbage_collection_threshold:0.8,max_split_size_mb:256`
  が unit に焼かれており、反復生成での OOM を抑止する。

systemd ユニットの非自明ポイント
--------------------------------

```ini
[Service]
User={{ user }}
SupplementaryGroups=render video                 # ← User= は primary group しか拾わない
Environment="HSA_OVERRIDE_GFX_VERSION=10.3.0"    # ← systemd は profile.d を読まない
Environment="HIP_VISIBLE_DEVICES=0"              # ← dGPU 履歴があるマシンで device 1 になる事故を回避
DeviceAllow=/dev/kfd rw                          # ← 明示
DeviceAllow=/dev/dri/renderD128 rw
```

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `register_home`, `amd_gpu_base`, `python_userland`, **`rocm` (`rocm_install: true`)**
- 数 GB のディスク空き (PyTorch ROCm wheel は数 GB)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `comfyui_install` | `true` | 全タスクのトップレベル ON/OFF |
| `comfyui_version` | `master` | git ref |
| `comfyui_install_dir` | `{{ ansible_home }}/Tool/comfyui` | clone + venv 先 |
| `comfyui_python` | `{{ ai_python \| default('/usr/bin/python3.12') }}` | venv 用 Python |
| `comfyui_torch_index` | `https://download.pytorch.org/whl/rocm6.2` | torch wheel index |
| `comfyui_torch_packages` | `[torch, torchvision]` | ピン指定したい場合は `torch==2.5.1+rocm6.2` 等 |
| `comfyui_listen` | `127.0.0.1` | bind |
| `comfyui_port` | `8188` | bind |
| `comfyui_extra_args` | `--use-pytorch-cross-attention` | ExecStart 末尾に付与 |
| `comfyui_user` | `{{ ansible_facts['env']['SUDO_USER'] }}` | サービス実行ユーザ |
| `comfyui_hsa_override_gfx_version` | `10.3.0` | unit 内 Environment |
| `comfyui_hip_visible_devices` | `"0"` | unit 内 Environment |
| `comfyui_pytorch_hip_alloc_conf` | `garbage_collection_threshold:0.8,max_split_size_mb:256` | unit 内 Environment |

Dependencies
------------

None (推奨: `amd_gpu_base`, `python_userland`, `rocm`)

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  vars:
    rocm_install: true
  roles:
    - register_home
    - amd_gpu_base
    - python_userland
    - rocm
    - comfyui
```

```yaml
# CPU フォールバック (GPU を使わずに動作確認だけしたい場合)
- hosts: localhost
  become: yes
  vars:
    comfyui_torch_index: "https://download.pytorch.org/whl/cpu"
  roles:
    - register_home
    - python_userland
    - comfyui
```

動作確認
--------

```bash
systemctl status comfyui
journalctl -u comfyui -n 200 | grep -iE 'hip|rocm|device|cuda'
# 期待: "Using ROCm" や "found device gfx1030" のような行

curl -s http://127.0.0.1:8188/system_stats | head

# ブラウザで http://127.0.0.1:8188/ を開く (Workflow が表示されれば OK)
```

トラブルシューティング
----------------------

- **`No HIP GPUs are available`**: 1) `HSA_OVERRIDE_GFX_VERSION` が
  unit に入っているか確認 (`systemctl show comfyui -p Environment`)、
  2) `groups | grep render` (サービスユーザ + render グループ)、
  3) 再ログイン or 再起動。
- **`HIP out of memory`**: BIOS UMA を増やす、SD1.5 ベースに切り替え、
  またはバッチサイズを下げる。
- **PyTorch wheel が見つからない**: ROCm の minor バージョンを
  `comfyui_torch_index` で合わせる (例: `rocm6.1`)。
- **起動まで数分かかる**: PyTorch + ROCm の初回 JIT 待ち。
  `journalctl -fu comfyui` で進捗確認。

Tags
----

- `comfyui`

License
-------

BSD
