# AMD ローカル AI 環境セットアップ

このドキュメントは、本リポジトリの Ansible ロール群を使って **AMD 環境
(特に Radeon 680M / gfx1035 のような iGPU)** にローカル AI 一式
(LLM / チャット UI / 画像生成 / 音声) を立ち上げる手順をまとめたもの。

## 対応ワークロードと使用ロール

| ワークロード | 主要ツール | バックエンド | ロール |
|---|---|---|---|
| LLM (CLI/サーバ) | llama.cpp | Vulkan (既定) / CPU | `llama` |
| LLM (常駐) | Ollama | ROCm (HSA override) / CPU | `ollama` |
| チャット UI | Open WebUI | — | `open_webui` |
| 画像生成 | ComfyUI + PyTorch | ROCm 必須 | `comfyui` |
| 音声 (STT) | whisper.cpp | Vulkan (既定) | `whisper_cpp` |
| OpenAI 互換 (任意) | Lemonade SDK | NPU/dGPU 向け | `lemonade` (既定 OFF) |

基盤ロール:

- `amd_gpu_base` — Mesa Vulkan + render/video グループ + 各種 assert
- `python_userland` — pyenv 経由の Python 3.12 + pipx (Python 3.14 問題の回避)
- `rocm` — ROCm ランタイム + HSA override (任意。`rocm_install: true` で有効)
- `register_home` — SUDO_USER のホームを `ansible_home` ファクトに格納 (既存ロール)

## 前提条件

### ハードウェア

- AMD GPU を搭載 (iGPU / dGPU 不問)
- 画像生成 (ComfyUI) を使うなら **BIOS の UMA frame buffer = 4 GiB 以上推奨**
  (iGPU の場合)
- ディスク空き 30 GB 以上 (PyTorch ROCm wheel + モデル含む)

### OS

- Ubuntu 22.04 / 24.04 / **26.04** で動作想定
- カーネル 6.x 以降 (`amdgpu` モジュールが gfx1035 を認識する世代)

### Ansible 制御ノード

- Ansible 2.9 以上
- 対象ホストへの SSH (もしくは `localhost`)
- 対象ホストの `sudo` 権限

---

## 1. 事前環境チェック

セットアップ前に対象マシンが想定どおりかを確認する。

```bash
# OS
lsb_release -a

# カーネル
uname -a

# AMD GPU 認識
lspci -nn | grep -iE 'vga|3d|display'

# amdgpu モジュール / 計算デバイス
lsmod | grep -E 'amdgpu|amdkfd'
ls -l /dev/kfd /dev/dri/renderD128

# RAM (画像生成用に 16 GiB 以上推奨)
free -h
```

期待値:

- `lspci` に `Advanced Micro Devices` の VGA エントリ
- `lsmod` で `amdgpu` がロード済
- `/dev/kfd` と `/dev/dri/renderD128` が存在
- iGPU の場合、BIOS で UMA frame buffer = 4 GiB 以上 (画像生成を使う場合)

---

## 2. プレイブック実行手順

### 2.1 フルセット (推奨初回)

```bash
cd /path/to/ansible-miscs
ansible-playbook -i hosts -K playbooks/conf/linux/local_ai.yml -e server=localhost
```

実行されるロール (順序):

1. `register_home` — SUDO_USER のホームを取得
2. `amd_gpu_base` — Vulkan + render/video グループ
3. `python_userland` — Python 3.12 (pyenv) + pipx
4. `rocm` — ROCm ランタイム + HSA override (`rocm_install: true` のとき)
5. `llama` — llama.cpp Vulkan ビルド配置
6. `whisper_cpp` — whisper.cpp Vulkan ビルド + base.en モデル
7. `ollama` — Ollama 導入 + drop-in
8. `open_webui` — Open WebUI (pipx + user systemd)
9. `comfyui` — ComfyUI (venv + ROCm PyTorch + system systemd)
10. `lemonade` — (既定 OFF)

所要時間目安: ROCm ランタイム (~6 GB) と PyTorch ROCm wheel (~2-3 GB) の
ダウンロードがあるため、回線速度次第で **20 分 〜 1 時間**。

### 2.2 サブセット実行

| 用途 | コマンド |
|---|---|
| テキスト + チャット UI のみ | `ansible-playbook -i hosts -K playbooks/conf/linux/local_ai_text.yml -e server=localhost` |
| 画像生成のみ | `ansible-playbook -i hosts -K playbooks/conf/linux/local_ai_image.yml -e server=localhost` |
| 音声のみ (ROCm 不要・最軽量) | `ansible-playbook -i hosts -K playbooks/conf/linux/local_ai_audio.yml -e server=localhost` |

### 2.3 個別ロールスキップ

`-e <var>=false` で個別に外せる。

```bash
# ComfyUI を入れない
ansible-playbook ... playbooks/conf/linux/local_ai.yml -e comfyui_install=false

# ROCm 全停止 (Ollama は CPU 推論)
ansible-playbook ... playbooks/conf/linux/local_ai.yml \
    -e rocm_install=false -e ollama_use_rocm=false -e comfyui_install=false
```

### 2.4 タグ実行 (再実行・部分更新)

```bash
# 例: ollama のみ再適用
ansible-playbook -i hosts -K playbooks/conf/linux/local_ai.yml \
    -e server=localhost --tags ollama

# 複数タグ
ansible-playbook ... --tags 'ollama,open_webui'
```

### 2.5 一度ログアウト or 再起動

`render` / `video` グループへのユーザ追加は **新規ログインまで反映されない**。
GPU を使う動作確認の前に一度ログアウト → ログイン (もしくは再起動) すること。

---

## 3. 動作確認手順

### 3.1 ハードウェア / ドライバ層

```bash
# Vulkan で AMD GPU が見えているか
vulkaninfo --summary | grep -iE 'radeon|vendor'

# 自分が render/video に入っているか
groups | grep -E 'render|video'

# デバイスノード
ls -l /dev/kfd /dev/dri/renderD128

# (rocm_install=true の場合) gfx ターゲット列挙
HSA_OVERRIDE_GFX_VERSION=10.3.0 /opt/rocm/bin/rocminfo | grep -E 'gfx|Marketing'
```

### 3.2 LLM (llama.cpp Vulkan)

```bash
~/Llama/b5604/build/bin/llama-cli --version

# Vulkan 経由の簡易推論 (任意の GGUF を用意して)
~/Llama/b5604/build/bin/llama-cli -m /path/to/model.gguf \
    -ngl 99 --device Vulkan0 -p "hello"
```

### 3.3 LLM (Ollama)

```bash
# サービス
systemctl status ollama
systemctl show ollama -p Environment | tr ' ' '\n' | grep -E 'HSA_|OLLAMA_'

# ログから GPU 検出を確認
journalctl -u ollama -n 100 | grep -iE 'rocm|gpu|compatible'
# 期待: "looking for compatible GPUs" → "compatible amdgpu devices: 1"

# API
curl -s http://127.0.0.1:11434/api/version
curl -s http://127.0.0.1:11434/api/tags

# 推論
ollama run llama3.2:3b "こんにちは、自己紹介してください"
```

### 3.4 チャット UI (Open WebUI)

```bash
# ユーザサービスが linger ON で動いているか
loginctl show-user $USER | grep Linger          # Linger=yes
systemctl --user status open-webui

# ヘルス
curl -s http://127.0.0.1:8080/health
```

ブラウザで `http://127.0.0.1:8080/` を開くと初回はアカウント作成画面。
登録後、左ペインのモデルセレクタに Ollama が pull 済みのモデルが並ぶ。

### 3.5 画像生成 (ComfyUI)

```bash
systemctl status comfyui
journalctl -u comfyui -n 200 | grep -iE 'hip|rocm|device|cuda'
# 期待: "Using ROCm" など。device 列挙が成功している行を探す

curl -s http://127.0.0.1:8188/system_stats
```

ブラウザで `http://127.0.0.1:8188/` を開くと既定ワークフローが表示される。
モデルファイル (`models/checkpoints/*.safetensors`) は別途配置。

### 3.6 音声 (whisper.cpp)

```bash
~/Tool/whisper.cpp/build/bin/whisper-cli --help | grep -i vulkan
cd ~/Tool/whisper.cpp
./build/bin/whisper-cli -m models/ggml-base.en.bin -f samples/jfk.wav
```

`base.en` のサンプル文字起こしが流れれば成功。日本語音声を処理したいなら
`whisper_cpp_models: [base, small, medium]` のように追加して再適用。

### 3.7 Lemonade (`lemonade_install: true` のとき)

```bash
systemctl --user status lemonade
curl -s http://127.0.0.1:13305/api/v1/models
```

OpenAI 互換クライアントから `base_url=http://127.0.0.1:13305/api/v1`, `api_key=lemonade` で接続可能。

---

## 4. トラブルシューティング

### 4.1 Ollama が GPU を使ってくれない

- `journalctl -u ollama` で `looking for compatible GPUs` の直後の行を確認。
- `systemctl show ollama -p Environment` で `HSA_OVERRIDE_GFX_VERSION=10.3.0`
  が入っていることを確認。入っていなければ `roles/ollama` のタスクを再実行:
  `ansible-playbook ... --tags ollama`
- `ollama` ユーザが `render` グループに入っているか:
  `id ollama | grep render`
- `roles/rocm` の `rocm_install: true` が反映されているか確認。

### 4.2 ComfyUI が `No HIP GPUs are available`

1. `systemctl show comfyui -p Environment | tr ' ' '\n' | grep HSA`
   → `HSA_OVERRIDE_GFX_VERSION=10.3.0` があるか
2. `groups <user>` で `render` と `video` 両方あるか
   (再ログインしないと反映されない)
3. `journalctl -u comfyui -n 200` で `pytorch_hip_alloc_conf` 等の
   環境変数が unit から渡っているか確認

### 4.3 ComfyUI で `HIP out of memory`

- BIOS で UMA frame buffer を 4 GiB 以上に設定
- SD1.5 ベースのモデルに切替 (SDXL は iGPU では厳しい)
- `comfyui_extra_args` に `--lowvram` を追加して再実行
- バッチサイズを 1 に落とす

### 4.4 `pip install torch` が遅い / 失敗

- ROCm wheel は数 GB あるため、`async: 3600` で 1 時間まで待たせている
- ROCm マイナーバージョンを `roles/rocm` の `rocm_version` と合わせる
  (例: ROCm 6.4 系を入れたら `comfyui_torch_index=https://download.pytorch.org/whl/rocm6.4`)
- 一時ディスク空き不足の場合は `~/.cache/pip` を確認

### 4.5 `amdgpu-install_*.deb` ダウンロードが 404

AMD のファイル名規則は不規則で、ROCm 6.x と 7.x で形式が変わる。
`rocm_version` を変えたら `rocm_installer_pkg_version` も合わせて差し替えること。

- 実在ファイル名は `https://repo.radeon.com/amdgpu-install/<rocm_version>/ubuntu/<codename>/`
  をブラウザで開いて確認
- 対応表は `roles/rocm/README.md` 参照

### 4.6 Open WebUI 起動失敗 / Python 3.14 由来のエラー

- `pipx list` でどの Python が使われているか確認
- 3.14 系で動かない場合は `roles/python_userland` を必ず先に通し、
  `ai_python` が 3.12 を指していることを確認:
  `ansible-playbook ... --tags python_userland,open_webui`

### 4.7 `loginctl enable-linger` が効かない

- WSL は `loginctl` が機能しない (本構成は **WSL 非対応** の前提)
- Linux ネイティブで失敗するなら `systemd-logind` サービスの稼働を確認

### 4.8 `vulkaninfo --summary` で GPU が出ない

- `mesa-vulkan-drivers` が入っているか: `dpkg -l | grep mesa-vulkan-drivers`
- `vulkaninfo 2>&1 | head -30` の冒頭エラーで原因が分かる場合が多い
- カーネル `amdgpu` が読み込まれていない可能性あり

### 4.9 BIOS UMA を変えたい

Ansible からは変更不可。マシン起動時に F2 / Del 等で BIOS に入り、
"UMA Frame Buffer Size" / "GPU Memory" を 4 GiB 以上に設定して保存。

---

## 5. アンインストール / クリーンアップ

各コンポーネントは独立しているので個別に外せる。

```bash
# Ollama
sudo systemctl disable --now ollama
sudo rm -rf /usr/local/bin/ollama /etc/systemd/system/ollama.service{,.d}
sudo userdel ollama

# Open WebUI (ユーザ環境で)
systemctl --user disable --now open-webui
pipx uninstall open-webui
rm -rf ~/.config/systemd/user/open-webui.service ~/.open-webui

# ComfyUI
sudo systemctl disable --now comfyui
sudo rm /etc/systemd/system/comfyui.service
rm -rf ~/Tool/comfyui

# whisper.cpp
rm -rf ~/Tool/whisper.cpp

# llama.cpp
rm -rf ~/Llama

# ROCm
sudo amdgpu-install --uninstall   # 慎重に
sudo rm /etc/profile.d/rocm-gfx1035.sh
```

`pyenv` で導入した Python 3.12 と `~/.pyenv` は他用途で使う可能性があるので
本リポジトリの構成では自動削除しない。

---

## 6. 参考リンク

- [llama.cpp](https://github.com/ggml-org/llama.cpp)
- [whisper.cpp](https://github.com/ggml-org/whisper.cpp)
- [Ollama](https://ollama.com/)
- [Open WebUI](https://github.com/open-webui/open-webui)
- [ComfyUI](https://github.com/comfyanonymous/ComfyUI)
- [Lemonade SDK](https://github.com/lemonade-sdk/lemonade)
- [PyTorch ROCm wheels](https://download.pytorch.org/whl/)
- [ROCm 公式](https://rocm.docs.amd.com/)
- [Mesa RADV (Vulkan)](https://docs.mesa3d.org/drivers/radv.html)
- [Radeon 680M (gfx1035) と HSA_OVERRIDE_GFX_VERSION](https://github.com/ROCm/ROCm/discussions/2932)
