openspec
========

OpenSpec (https://github.com/Fission-AI/OpenSpec) を Debian/Ubuntu に
npm 経由でグローバル導入するロール。

OpenSpec はスペック駆動開発を AI コーディングアシスタント (Claude Code,
Cursor, etc.) と組み合わせるための CLI で、npm パッケージ
`@fission-ai/openspec` として配布されている。

Requirements
------------

- Ansible 2.9 以上
- Debian/Ubuntu
- `community.general` コレクション (npm モジュール用)
  - 未導入なら `ansible-galaxy collection install community.general`

Role Variables
--------------

| 変数 | 既定値 | 説明 |
| --- | --- | --- |
| `openspec_version` | `latest` | npm からインストールする OpenSpec のバージョン。`latest` の場合は `state=latest`。固定したい場合は `0.x.y` 等 |
| `openspec_npm_package` | `@fission-ai/openspec` | npm パッケージ名 |
| `openspec_node_min_version` | `20.19.0` | OpenSpec が要求する Node.js の最低バージョン (README 記載値) |
| `openspec_node_major` | `22` | Node が未導入 / 要件未満時に NodeSource から入れる Node のメジャー系列 (Active LTS) |
| `openspec_nodesource_keyring` | `/usr/share/keyrings/nodesource.asc` | NodeSource apt 鍵の保存先 |

挙動:

1. `node --version` を確認し、未導入 or `openspec_node_min_version` 未満なら
   NodeSource の apt リポジトリから Node.js `{{ openspec_node_major }}.x`
   を導入する (既に十分新しい Node が入っていれば触らない)。
2. `npm -g` で `{{ openspec_npm_package }}` を導入する。

Dependencies
------------

なし (Node.js が無ければロール内で NodeSource から入れる)。

Example Playbook
----------------

```yaml
- hosts: localhost
  become: yes
  roles:
    - openspec
```

特定バージョンを入れたい場合:

```bash
ansible-playbook -i hosts playbooks/conf/linux/openspec.yml \
  -e server=home_desktops \
  -e openspec_version=0.5.0
```

Tags
----

- `openspec`: このロールが追加するすべてのタスクに付与。

License
-------

BSD-3-Clause

Author Information
------------------

dobachi
