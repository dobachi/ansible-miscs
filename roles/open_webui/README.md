open_webui
==========

[Open WebUI](https://github.com/open-webui/open-webui) を **pipx + ユーザ
systemd サービス** として導入する Docker 不要構成のロール。Ollama (or 任意の
OpenAI 互換 API) を裏に置いて ChatGPT 風の Web UI を提供する。

特徴
----

- Docker 不要 (`pipx install open-webui` + `systemctl --user`)
- データディレクトリはユーザ home 配下 (`~/.open-webui/`) — アンインストール時もチャット履歴が残る
- `loginctl enable-linger` 済みなので **ログアウトしてもサービス稼働継続**
- Python 3.12 (pyenv 経由) を使う ((`ai_python` fact 経由)。3.14 では Open WebUI が起動しない可能性が高い)

Requirements
------------

- Ansible 2.9 以上
- Debian / Ubuntu 系 (apt — pipx 用)
- `become: yes`
- 先行ロール: `register_home`, `python_userland` (本ロールは `ai_python` ファクトを参照)
- バックエンド: `ollama` を同ホストか別ホストで稼働させておくのが標準構成

Role Variables
--------------

| 変数 | デフォルト | 説明 |
| --- | --- | --- |
| `open_webui_install` | `true` | 全タスクのトップレベル ON/OFF |
| `open_webui_version` | `""` | 空で latest、`"0.5.7"` 等でピン |
| `open_webui_host` | `"127.0.0.1"` | bind |
| `open_webui_port` | `8080` | bind |
| `open_webui_ollama_base_url` | `"http://127.0.0.1:11434"` | バックエンド Ollama URL |
| `open_webui_data_dir` | `{{ ansible_home }}/.open-webui` | チャット/RAG/設定の保存先 |
| `open_webui_python` | `{{ ai_python \| default('/usr/bin/python3.12') }}` | pipx --python |
| `open_webui_user` | `{{ ansible_facts['env']['SUDO_USER'] }}` | サービス実行ユーザ |
| `open_webui_extra_env` | `{}` | unit に追加注入する環境変数 |

Dependencies
------------

None (推奨: `python_userland`, `ollama`)

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  roles:
    - register_home
    - python_userland
    - ollama
    - open_webui
```

```yaml
# 別ホストの Ollama を指す
- hosts: localhost
  become: yes
  vars:
    open_webui_ollama_base_url: "http://192.168.1.10:11434"
  roles:
    - register_home
    - python_userland
    - open_webui
```

動作確認
--------

```bash
# linger 有効になっているか
loginctl show-user $USER | grep Linger     # Linger=yes

# ユーザサービス稼働
systemctl --user status open-webui

# ヘルス
curl -s http://127.0.0.1:8080/health

# ブラウザで http://127.0.0.1:8080/ を開く (初回はアカウント作成画面)
```

トラブルシューティング
----------------------

- **pipx が systemctl --user を見つけられない**: `XDG_RUNTIME_DIR` が
  未設定。`systemctl --user --no-pager status open-webui` を一度対話 SSH
  で叩くと環境変数が正しく評価される。
- **ChatGPT 風 UI がモデル一覧を出さない**: バックエンド (Ollama)
  が listen しているか、`OLLAMA_BASE_URL` が正しいかを確認。
- **3.14 環境で起動失敗**: `roles/python_userland` を必ず先に適用し、
  `ai_python` が 3.12 を指していることを確認 (`pipx list` で確認)。

Tags
----

- `open_webui`

License
-------

BSD
