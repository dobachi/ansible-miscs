rocm
====

AMD ROCm ランタイムを `amdgpu-install` 経由で導入する任意ロール。
**既定では `rocm_install: false`** — Vulkan のみで完結するワークロード
(`whisper_cpp`, `llama.cpp` Vulkan ビルド) しか使わないなら不要。

ROCm が必要なケース:

- **Ollama を GPU で動かす** (Ollama は Vulkan バックエンドを持たない)
- **ComfyUI / Stable Diffusion** (PyTorch ROCm wheel を使う)
- HIP ベースの自作 / サードパーティ推論

サポート対象外 GPU の扱い (`HSA_OVERRIDE_GFX_VERSION`)
------------------------------------------------------

ROCm が公式サポートする gfx ターゲットは限られており、Radeon 680M
(`gfx1035`)、RX 6400 等の `gfx1036`、その他多くの iGPU/低位 dGPU が
**公式リストに無い**。これらは `HSA_OVERRIDE_GFX_VERSION=10.3.0` で
`gfx1030` (RX 6800/6900) として動作させる。本ロールは:

1. `/etc/profile.d/rocm-gfx1035.sh` にこの環境変数を定義 (対話シェル向け)
2. **systemd ユニットは別途 `Environment=` で再定義する必要がある**
   (systemd は `/etc/profile.d/` を読まない)。
   下流ロール (`ollama`, `comfyui`, `llama` の `llama-server` 等) が
   個別に再定義している。

Ubuntu 26.04 と ROCm 公式 apt の関係
------------------------------------

ROCm 6.x の公式 apt リポジトリは Ubuntu 22.04 / 24.04 までを
ターゲットとしている。本ロールは `rocm_apt_codename: "noble"` (24.04)
の amdgpu-install パッケージを取得し、ランタイムだけを 26.04 に乗せる。
DKMS は **使わない** (`--no-dkms`) — 26.04 の 7.x カーネルでは in-tree
`amdgpu` の方が安定。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `amd_gpu_base` (依存)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `rocm_install` | `false` | ROCm を実際に入れるか (false なら全タスク skip) |
| `rocm_version` | `6.4.4` | ROCm リリース系 |
| `rocm_installer_pkg_version` | `6.4.60404-1` | `amdgpu-install_*.deb` のファイル名バージョン (下表参照) |
| `rocm_apt_codename` | `noble` | amdgpu-install を取得するコードネーム |
| `rocm_usecases` | `[rocm, hiplibsdk]` | `amdgpu-install --usecase=` |
| `rocm_no_dkms` | `true` | DKMS スキップ (in-tree amdgpu を使う) |
| `rocm_hsa_override_gfx_version` | `10.3.0` | サポート外 GPU の偽装 gfx |
| `rocm_extra_env` | `{HSA_ENABLE_SDMA: "0"}` | profile.d に書く追加環境変数 |
| `rocm_profile_d_path` | `/etc/profile.d/rocm-gfx1035.sh` | snippet 配置パス |

amdgpu-install パッケージ名の対応表
-----------------------------------

AMD のファイル名規則は ROCm 7.x で変わったため、`rocm_version` を変えるなら
`rocm_installer_pkg_version` も合わせて差し替える必要がある。
実 URL は `https://repo.radeon.com/amdgpu-install/<rocm_version>/ubuntu/<codename>/`
で確認可能。

| `rocm_version` | `rocm_installer_pkg_version` | PyTorch wheel index |
| --- | --- | --- |
| `6.2.2` | `6.2.60202-1` | `https://download.pytorch.org/whl/rocm6.2` |
| `6.3.4` | `6.3.60304-1` | `https://download.pytorch.org/whl/rocm6.3` |
| `6.4.4` (既定) | `6.4.60404-1` | `https://download.pytorch.org/whl/rocm6.4` |
| `7.0.3` | `7.0.3.70003-1` | `https://download.pytorch.org/whl/rocm7.0` |
| `7.2.3` | `7.2.3.70203-1` | `https://download.pytorch.org/whl/rocm7.2` |

Dependencies
------------

- `amd_gpu_base`

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  vars:
    rocm_install: true                 # 明示的に ON
  roles:
    - rocm
```

動作確認
--------

```bash
# rocminfo で gfx ターゲットが列挙されるか
HSA_OVERRIDE_GFX_VERSION=10.3.0 /opt/rocm/bin/rocminfo | grep -E 'gfx|Marketing'

# ROCm SMI (バージョンによってはオプション)
/opt/rocm/bin/rocm-smi || true

# 対話シェルで自動 export 確認
bash -lc 'env | grep -E "HSA_OVERRIDE|HSA_ENABLE"'
```

Tags
----

- `rocm`
- `amd_gpu_base` (依存ロール由来)

License
-------

BSD
