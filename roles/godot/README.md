godot
=====

Godot Engine（標準版 / GDScript）を公式 GitHub リリースのバイナリからインストールするロール。
`goland` ロールと同じ流儀で `~/Applications/godot/<version>/` に展開し、`alternatives` で
`~/.local/bin/godot` を選択中バージョンの実行ファイルに向ける（PATH から `godot` 起動可）。

Requirements
------------

- 対象は Linux x86_64（WSL2 / WSLg を含む）。
- **`sudo ansible-playbook ...` で実行すること**（プロセス全体を sudo 起動）。タスクは
  `ansible_facts['env']['SUDO_USER']` を参照して、sudo 元ユーザーのホーム配下にインストールする。
  `goland` / `clion` など他のアプリ系ロールと同じ前提。
- `unzip` はロール内で導入する（zip 展開に必要）。

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `godot_version` | `4.7-stable` | リリースタグ。`https://github.com/godotengine/godot/releases` 参照 |
| `godot_arch` | `linux.x86_64` | アーティファクトのプラットフォーム種別 |
| `godot_home` | `~/Applications/godot` | 展開先ベース（SUDO_USER のホーム配下） |
| `godot_create_desktop_entry` | `true` | `~/.local/share/applications/godot.desktop` を作成 |

別バージョンを入れる場合は `-e godot_version=4.6-stable` のように上書きする。
`alternatives` により `~/.local/bin/godot` のリンク先が切り替わり、複数バージョンを共存できる。

Example Playbook
----------------

    - hosts: "{{ server | default('wsl') }}"
      roles:
        - commons
        - godot

実行例（プロセス全体を sudo で起動する点に注意）:

    # この環境 (localhost / wsl) に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/godot.yml

    # k16 に適用
    sudo ansible-playbook -i hosts playbooks/conf/linux/godot.yml -e server=k16

インストール後は `godot`（PATH 経由）またはアプリ一覧の "Godot Engine" から起動する。
WSLg 環境ではエディタの GUI がそのまま表示される。

License
-------

MIT

Author Information
------------------

dobachi
