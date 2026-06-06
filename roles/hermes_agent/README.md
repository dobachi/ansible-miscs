hermes_agent
============

Nous Research の [Hermes Agent](https://hermes-agent.nousresearch.com/) CLI を
per-user で導入し、`~/.hermes/config.yaml` を配置するロール。

ローカル LLM ([`roles/llama_server_deb`](../llama_server_deb/) で立てた
llama-server) と Anthropic Claude を併用する構成で出荷する。

**認証情報はこのロールでは扱わない**。Claude の API key や OAuth トークンは
ロール実行後に手で設定する (Vault に入れて流す運用も可能だが、
本ロールは「中身が個人ごとに違う & 公開設定との混在を避けたい」という
理由でわざと外している)。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu (apt 経由で依存パッケージを入れる)
- 対象ユーザー (`hermes_agent_user`) がログイン可能 (Hermes インストーラを
  `become_user` で走らせるため)
- ネット接続 (公式インストーラを curl で落とす)

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `hermes_agent_enable` | `false` | このロールを実行するかのスイッチ |
| `hermes_agent_user` | `{{ ansible_user_id }}` | Hermes を入れる対象ユーザー |
| `hermes_agent_installer_url` | `https://hermes-agent.nousresearch.com/install.sh` | 公式 per-user installer URL |
| `hermes_agent_install_apt_deps` | `true` | 依存 apt パッケージを入れるか |
| `hermes_agent_apt_deps` | `[curl, ca-certificates, ripgrep, ffmpeg, nodejs]` | apt で揃える依存。Ubuntu 26.04 の `nodejs` は `npm` を内包するので `npm` は列挙しない |
| `hermes_agent_local_base_url` | `http://127.0.0.1:8080/v1` | `model.base_url` に書くローカル LLM の OpenAI 互換 endpoint |
| `hermes_agent_local_model_name` | `qwen2.5-coder-14b` | ローカル LLM の model id (`llama_server_deb_model_alias` と揃える) |
| `hermes_agent_local_context_length` | `65536` | `model.context_length`。Hermes Agent は最小 64K を要求。auto-detect だと `n_ctx_train=32768` で弾かれるので明示する必要あり |
| `hermes_agent_local_api_key` | `dummy` | `model.api_key`。llama-server はチェックしないが Hermes は値を要求 |

設定ファイルは初回 bootstrap (`force: false`) のみで配置する。`hermes setup`
/ `hermes model` 等で加えた修正は再ロール実行で消えない。撒き直したい場合は
`~/.hermes/config.yaml` を手で消してから再実行する。

Claude (Anthropic) 主脳構成にしたい場合は ansible 後に `hermes setup` を対話
実行する (本ロールは認証情報を扱わない)。

Dependencies
------------

なし。ただし実用上は [`roles/llama_server_deb`](../llama_server_deb/) と
組み合わせて使うことを想定している (ローカル LLM 側)。

Example Playbook
----------------

```yaml
- hosts: k16
  become: true
  roles:
    - { role: llama_server_deb, llama_server_deb_enable: true }
    - { role: hermes_agent, hermes_agent_enable: true }
```

実行後の手動ステップ:

1. **Claude 認証** (どちらか一方)
   - 対話: `hermes setup` を実行し、Anthropic OAuth または API key を入力
   - 環境変数: `echo 'export ANTHROPIC_API_KEY=sk-ant-...' >> ~/.bashrc && source ~/.bashrc`

2. **動作確認**
   ```bash
   hermes doctor                              # Hermes 健全性チェック
   systemctl status llama-server              # ローカル LLM 稼働確認
   curl http://127.0.0.1:8080/v1/models       # OpenAI 互換エンドポイント応答
   ```

3. **セッション内のモデル切替**
   ```
   hermes
   # 起動後:
   /model custom:local                        # ローカル LLM
   /model anthropic:claude-sonnet-4-6          # Claude
   ```

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
