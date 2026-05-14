whisper_cpp
===========

OpenAI Whisper の C++ 実装 ([whisper.cpp](https://github.com/ggml-org/whisper.cpp))
を **ソースから** ビルドして配置するロール。GPU バックエンドは Vulkan を既定とし、
AMD RDNA2 iGPU (Radeon 680M / gfx1035) を含む幅広い AMD GPU で ROCm 不要で動作する。

なぜ Vulkan か
--------------

- whisper.cpp は GGML を介して Vulkan / HIP(ROCm) / CPU をサポート。
- AMD iGPU は ROCm 公式サポート外なので、Vulkan が**最も摩擦が少ない GPU 経路**。
- ROCm を使いたい場合は `whisper_cpp_backend: rocm` + `roles/rocm` を併用。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `register_home`
- (Vulkan 利用時) `roles/amd_gpu_base` を先に適用しておくとデバイス権限が揃う

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `whisper_cpp_version` | `v1.7.5` | git タグ |
| `whisper_cpp_backend` | `vulkan` | `vulkan` / `cpu` / `rocm` |
| `whisper_cpp_install_dir` | `{{ ansible_home }}/Tool/whisper.cpp` | clone + build 先 |
| `whisper_cpp_common_build_deps` | `[build-essential, cmake, git, pkg-config]` | 共通 apt deps |
| `whisper_cpp_vulkan_deps` | `[libvulkan-dev, mesa-vulkan-drivers, glslc, glslang-tools]` | Vulkan ビルド時のみ追加 (`glslc` は CMake FindVulkan が要求) |
| `whisper_cpp_models` | `[base.en]` | DL するモデル名 (`models/download-ggml-model.sh` の引数) |
| `whisper_cpp_build_jobs` | `-1` | `-1` で全コア利用 |

Dependencies
------------

None (apt 依存はロール内で扱う)

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  roles:
    - register_home
    - amd_gpu_base
    - whisper_cpp
```

```yaml
# 日本語モデル + 大きめモデルも欲しい場合
- hosts: localhost
  become: yes
  roles:
    - register_home
    - role: whisper_cpp
      vars:
        whisper_cpp_models:
          - base
          - small
          - medium
```

動作確認
--------

```bash
# ビルド成果物
~/Tool/whisper.cpp/build/bin/whisper-cli --help | head

# Vulkan が組み込まれているか
~/Tool/whisper.cpp/build/bin/whisper-cli --help | grep -i vulkan

# サンプル音声で文字起こし
cd ~/Tool/whisper.cpp
./build/bin/whisper-cli -m models/ggml-base.en.bin -f samples/jfk.wav

# 任意ファイル
./build/bin/whisper-cli -m models/ggml-base.en.bin -f /path/to/audio.wav -l auto
```

Tags
----

- `whisper_cpp`

License
-------

BSD
