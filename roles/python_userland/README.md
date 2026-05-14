python_userland
===============

ローカル AI ロール群が **Python 3.14 未対応**なツール (PyTorch ROCm wheel,
ComfyUI, lemonade-sdk, open-webui) を扱うために、安定版 Python (3.12 系)
と `pipx` を導入する基礎ロール。

実体は `pyenv` ロール (依存) を使った CPython のソースビルド + apt の
`pipx`。後段ロールには `ai_python` ファクトとして CPython のフルパスを
公開し、各ロールが `--python {{ ai_python }}` で揃える。

なぜ pyenv を使うか
-------------------

- Ubuntu 26.04 (`resolute`) 既定 Python は 3.14。AI 系 wheel が間に合わない。
- deadsnakes PPA は `resolute` 向けパッケージの提供有無が不安定。
- 既存 `roles/pyenv` がそのまま使えるため、新たな第三者リポジトリを増やさない。

含まれるもの
------------

| カテゴリ | 内容 |
| --- | --- |
| ビルド依存 | `build-essential`, `libssl-dev`, `zlib1g-dev`, `libsqlite3-dev` 等 (CPython のソースビルドに必須) |
| Python 本体 | `pyenv install <python_userland_python_version>` |
| 隔離インストーラ | apt の `pipx` |

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `register_home` (SUDO_USER のホームを `ansible_home` に格納)

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `python_userland_python_version` | `3.12.7` | pyenv で導入する CPython バージョン |
| `python_userland_install_pipx` | `true` | apt の pipx を入れるか |
| `python_userland_build_deps` | (一覧) | CPython ビルドに必要な apt パッケージ群 |

公開ファクト
------------

| ファクト | 例 | 用途 |
| --- | --- | --- |
| `ai_python` | `/home/<user>/.pyenv/versions/3.12.7/bin/python3` | 後段ロール (`open_webui`, `comfyui`, `lemonade`) が `--python` 等に渡す |

Dependencies
------------

- `pyenv`

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  roles:
    - register_home
    - python_userland
```

Tags
----

- `python_userland`
- `pyenv` (依存ロール由来)

License
-------

BSD
