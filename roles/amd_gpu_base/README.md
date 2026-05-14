amd_gpu_base
============

AMD GPU 上で Vulkan/ROCm を使うすべての後段ロール (`llama`, `ollama`,
`comfyui`, `whisper_cpp`, `lemonade`) が前提とする **共通基盤ロール**。

やること:

- Mesa Vulkan ドライバ + 診断 CLI (`vulkaninfo`, `radeontop`, `clinfo`) 導入
- 対象ユーザを `render` / `video` グループに追加 (`/dev/kfd`, `/dev/dri/renderD128` 利用権)
- `amdgpu` カーネルモジュール / `/dev/kfd` / `/dev/dri/renderD128` の存在 assert
- iGPU UMA に対する VRAM 警告 (RAM が閾値未満なら debug メッセージで通知)

iGPU 利用上の注意 (Radeon 680M / Phoenix 等)
--------------------------------------------

- iGPU の VRAM は BIOS の UMA frame buffer 設定で決まる **システム RAM の切り出し**。
  Ansible からは変更不可。**画像生成 (SDXL) を使う場合は BIOS で UMA = 4 GiB 以上を推奨**。
- `render` / `video` グループ追加は **再ログインしないと有効にならない**。
  ログアウト → ログイン (もしくは再起動) のあとに GPU ワークロードを実行する。

含まれるもの
------------

| パッケージ | 役割 |
| --- | --- |
| `mesa-vulkan-drivers` | RADV (Mesa の Vulkan 実装)。RDNA1/2/3 をカバー |
| `vulkan-tools` | `vulkaninfo`, `vkcube` 等の診断 CLI |
| `libdrm2` | DRM ユーザ空間ライブラリ |
| `radeontop` | iGPU/dGPU 利用率の TUI モニタ |
| `clinfo` | OpenCL デバイス列挙 (ROCm OpenCL 入れた場合の確認用) |

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- AMD GPU を搭載したマシン (`amdgpu` モジュールが起動時にロード済)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `amd_gpu_base_packages` | `[mesa-vulkan-drivers, vulkan-tools, libdrm2, radeontop, clinfo]` | apt で導入するパッケージ群 |
| `amd_gpu_base_groups` | `[render, video]` | 対象ユーザに追加するグループ |
| `amd_gpu_base_target_user` | `{{ ansible_facts['env']['SUDO_USER'] }}` | グループ追加先ユーザ |
| `amd_gpu_base_min_ram_mb_warn` | `16000` | この値未満なら警告を出す (画像生成での OOM 予防) |

Dependencies
------------

None

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  roles:
    - amd_gpu_base
```

動作確認
--------

```bash
# Vulkan で AMD GPU が見えているか
vulkaninfo --summary | grep -iE 'radeon|vendor'

# 自分が render/video に入っているか (再ログイン後)
groups | grep -E 'render|video'

# デバイスノード
ls -l /dev/kfd /dev/dri/renderD128

# GPU 使用率
radeontop
```

Tags
----

- `amd_gpu_base`

License
-------

BSD
