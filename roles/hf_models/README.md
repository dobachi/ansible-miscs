hf_models
=========

HuggingFace 上のモデルファイル (主に llama.cpp / whisper.cpp 用 GGUF) を
**Ansible で宣言的に取得**するロール。`hf` CLI を pipx 経由で `ai_python`
(3.12) に紐付けて導入し、リストで指定された各ファイルを `~/Models/` に
ダウンロードする。冪等 (既存ファイルはスキップ)。

なぜ ai_python 経由か
---------------------

`huggingface_hub` の最新版は依存ツリーが大きく、Python 3.14 で wheel が
揃わないリスクがある。`roles/python_userland` が用意する 3.12 に pipx を
ピン留めすることで、システム pipx (3.14) との干渉と PEP 668
(externally-managed-environment) を両方回避する。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `register_home`, `python_userland` (推奨)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `hf_models_install` | `false` | 全タスクのトップレベル ON/OFF |
| `hf_models_dir` | `{{ ansible_home }}/Models` | DL 先 |
| `hf_models_python` | `{{ ai_python \| default('/usr/bin/python3.12') }}` | pipx --python |
| `hf_models_user` | `{{ ansible_facts['env']['SUDO_USER'] }}` | 実行ユーザ |
| `hf_models_token` | `""` | (任意) ゲート付きモデル用 HF トークン |
| `hf_models_files` | `[]` | `{repo, file}` 辞書のリスト (下記例) |

`hf_models_files` の指定例
--------------------------

```yaml
hf_models_files:
  - repo: Qwen/Qwen2.5-3B-Instruct-GGUF
    file: qwen2.5-3b-instruct-q4_k_m.gguf

  - repo: Qwen/Qwen2.5-7B-Instruct-GGUF
    file: qwen2.5-7b-instruct-q4_k_m.gguf

  - repo: bartowski/Llama-3.2-3B-Instruct-GGUF
    file: Llama-3.2-3B-Instruct-Q4_K_M.gguf

  - repo: ggerganov/whisper.cpp
    file: ggml-base.en.bin
```

同じ repo から複数ファイルを取りたい場合は同 repo を複数行にする
(`hf download` の流儀に合わせている)。

Dependencies
------------

None (推奨: `python_userland`)

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  vars:
    hf_models_install: true
    hf_models_files:
      - { repo: Qwen/Qwen2.5-3B-Instruct-GGUF, file: qwen2.5-3b-instruct-q4_k_m.gguf }
  roles:
    - register_home
    - python_userland
    - hf_models
```

ゲート付きモデル (Llama 3 等) を取得する場合
--------------------------------------------

`hf_models_token` で HuggingFace の Read トークンを渡す。**コミットしない**こと:

```bash
ansible-playbook ... -e hf_models_token=hf_xxxxxxxxxxxxxxxxxxxx
```

または `host_vars/<host>.yml` を gitignore して管理。

Tags
----

- `hf_models`

License
-------

BSD
