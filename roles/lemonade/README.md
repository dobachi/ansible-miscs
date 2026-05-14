lemonade
========

[Lemonade SDK](https://github.com/lemonade-sdk/lemonade) を pipx + ユーザ
systemd で導入する**任意ロール**。AMD 公認の OpenAI 互換ローカル AI サーバで、
内部に llama.cpp / OnnxRuntime-GenAI / FastFlowLM (NPU) / vLLM-ROCm を抱える。

**既定で OFF** (`lemonade_install: false`)
------------------------------------------

Radeon 680M (gfx1035) のような iGPU マシンでは Lemonade の差別化機能
(Ryzen AI NPU, Strix Halo / Strix Point ROCm, dGPU vLLM) がいずれも使えず、
Ollama と同じ守備範囲を二重に持つ形になる。**iGPU 単独構成では Ollama を
使うほうが運用が単純**。

ON にする価値があるのは:

- Ryzen AI NPU 搭載機 (XDNA / XDNA2)
- Strix Halo / Strix Point の dGPU 級 ROCm
- 「OpenAI 互換 API を厳密に欲しい」場面 (Continue / Aider 等の純粋な OpenAI クライアント)

Python バージョン制約 (重要)
----------------------------

`lemonade-sdk` の wheel は `requires-python = ">=3.10, <3.14"`。
Ubuntu 26.04 既定の Python 3.14 では **インストールできない**。
本ロールは `roles/python_userland` の `ai_python` (3.12) を pipx に渡し、
事前に `assert` で 3.14+ を弾く。

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt)
- `become: yes`
- 先行ロール: `register_home`, **`python_userland` (必須)**

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `lemonade_install` | `false` | 全タスクのトップレベル ON/OFF |
| `lemonade_version` | `""` | 空で latest |
| `lemonade_host` | `127.0.0.1` | bind |
| `lemonade_port` | `13305` | upstream 既定 |
| `lemonade_python` | `{{ ai_python \| default('/usr/bin/python3.12') }}` | pipx --python |
| `lemonade_extras` | `[]` | pip extras (`oga-ryzenai`, `llamacpp` 等) |
| `lemonade_user` | `{{ ansible_facts['env']['SUDO_USER'] }}` | サービス実行ユーザ |
| `lemonade_extra_env` | `{}` | unit に追加注入する環境変数 |

Dependencies
------------

None (推奨: `python_userland`)

Example Playbook
----------------

```yaml
# Ryzen AI NPU 搭載機での想定構成
- hosts: ryzen_ai_hosts
  become: yes
  vars:
    lemonade_install: true
    lemonade_extras: [oga-ryzenai]
  roles:
    - register_home
    - python_userland
    - lemonade
```

動作確認
--------

```bash
loginctl show-user $USER | grep Linger
systemctl --user status lemonade
curl -s http://127.0.0.1:13305/api/v1/models
# OpenAI 互換クライアントから:
#   base_url=http://127.0.0.1:13305/api/v1, api_key=lemonade
```

Tags
----

- `lemonade`

License
-------

BSD
