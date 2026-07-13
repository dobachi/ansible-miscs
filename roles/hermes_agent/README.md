hermes_agent
============

Nous Research の [Hermes Agent](https://hermes-agent.nousresearch.com/) CLI を
per-user で導入し、`~/.hermes/config.yaml` + `~/.hermes/.env` を配置するロール。

**OpenRouter primary** で bootstrap する。Hermes Agent の公式一次推奨は Nous Portal
/ OpenRouter などクラウド provider で、Nous 自身が Hermes-3 / Hermes-4 を
"NOT agentic" と警告している (`hermes_cli/model_switch.py:53-58`) ため、
ローカル自前推論 (`roles/llama_server_deb` + `roles/amdgpu_gtt`) は本ロールから
分離してあり、opt-in で組み合わせる形。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (apt 経由で依存パッケージを入れる)
- 対象ユーザー (`hermes_agent_user`) がログイン可能 (Hermes インストーラを
  `become_user` で走らせるため)
- ネット接続 (公式インストーラを curl で落とす)
- OpenRouter API key (`https://openrouter.ai/keys`) を Vault で暗号化して渡す

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_enable` | `false` | このロールを実行するかのスイッチ |
| `hermes_agent_user` | 実行ユーザー (`SUDO_USER` / `ansible_user` / env `USER`) | Hermes を入れる対象ユーザー |
| `hermes_agent_installer_url` | `https://hermes-agent.nousresearch.com/install.sh` | 公式 per-user installer URL |
| `hermes_agent_install_apt_deps` | `true` | 依存 apt パッケージを入れるか |
| `hermes_agent_apt_deps` | `[curl, ca-certificates, ripgrep, ffmpeg, nodejs]` | Ubuntu 26.04 の `nodejs` は `npm` を内包するので `npm` は列挙しない |
| `hermes_agent_openrouter_model` | `anthropic/claude-sonnet-4.5` | OpenRouter で primary として使うモデル slug |
| `hermes_agent_openrouter_api_key` | `""` | OPENROUTER_API_KEY。空文字列だと `.env` 書き込みを skip。Vault 化した値を渡す想定 |
| `hermes_agent_reset_config` | `false` | true にするとロールが `~/.hermes/config.yaml` を強制上書きする (元ファイルは backup: で `.~` 退避) |

設定ファイルは既定で初回 bootstrap (`force: false`) のみで配置する。
`hermes setup` / `hermes model` / `hermes config set` 等で加えた修正は再ロール
実行で消えない。ロール側のテンプレを変えた直後に反映したい場合は
`-e hermes_agent_reset_config=true` を渡して再実行する。

別 provider (Nous Portal / Anthropic 直 / ローカル llama-server 等) を併用したい
場合は ansible 実行後に `hermes setup` を対話実行する
(本ロールは `model:` ブロックと `OPENROUTER_API_KEY` のみ管理)。

Dependencies
------------

なし。ローカル LLM 併用の実験用途で
[`roles/llama_server_deb`](../llama_server_deb/) と組み合わせることも可能だが、
その場合は `hermes setup` で custom endpoint を追加する必要がある
(本ロールは触らない)。

Example Playbook
----------------

```yaml
- hosts: k16
  become: true
  vars_files:
    - "{{ inventory_dir }}/host_vars/k16/vault.yml"
  roles:
    - role: hermes_agent
      vars:
        hermes_agent_enable: true
        hermes_agent_openrouter_model: "anthropic/claude-sonnet-4.5"
        hermes_agent_openrouter_api_key: "{{ vault_hermes_openrouter_api_key }}"
```

Vault に API key を登録する例:

```bash
ansible-vault encrypt_string 'sk-or-v1-...' \
  --name 'vault_hermes_openrouter_api_key' \
  >> host_vars/k16/vault.yml
```

実行後の動作確認:

```bash
hermes doctor                              # Hermes 健全性チェック
hermes config show                         # provider: openrouter が反映されているか
hermes                                     # 起動して適当に会話 → tool call 発火を確認
```

セッション内のモデル切替:

```
/model openrouter/anthropic/claude-sonnet-4.5
/model openrouter/qwen/qwen3-coder
/model openrouter/deepseek/deepseek-v3.2
```

推奨モデル (OpenRouter 経由):

| 用途 | モデル slug | 備考 |
| --- | --- | --- |
| メイン (agentic) | `anthropic/claude-sonnet-4.5` | Hermes docs 一次推奨クラス |
| コスト重視 | `anthropic/claude-haiku-4-5` | 廉価、tool-use OK |
| コーディング特化 | `qwen/qwen3-coder`, `deepseek/deepseek-v3.2` | Hermes curated list 掲載 |
| コンパクション (aux) | `google/gemini-flash-*` | 安価・高速 |

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
