# ansible-miscs プロジェクト固有指示書

このプロジェクトはAnsibleを使用したシステム構成自動化のためのプレイブックおよびロール集です。
Linux、WSL、Windows、Raspberry Piなど複数プラットフォームの開発環境セットアップと構成管理を行います。

## プロジェクト概要
- **目的**: システム構成と開発環境の自動セットアップ
- **技術スタック**: Ansible 2.9+
- **対象プラットフォーム**: Linux, WSL, Windows, Raspberry Pi
- **規模**: 49プレイブック、78ロール

## AI開発支援設定

このプロジェクトでは`instructions/ai_instruction_kits/`のAI指示書システムを使用します。
タスク開始時は`instructions/ai_instruction_kits/instructions/ja/system/ROOT_INSTRUCTION.md`を読み込んでください。

## プロジェクト設定
- 言語: 日本語 (ja)
- チェックポイント管理: 有効
- チェックポイントスクリプト: scripts/checkpoint.sh
- ログファイル: checkpoint.log

## 重要なパス
- AI指示書システム: `instructions/ai_instruction_kits/`
- チェックポイントスクリプト: `scripts/checkpoint.sh`
- プロジェクト固有の設定: このファイル（`instructions/PROJECT.md`）
- Ansibleプレイブック: `playbooks/`
- Ansibleロール: `roles/`
- インベントリ: `hosts` (Linux/Unix), `hosts.win` (Windows)
- Ansible設定: `ansible.cfg`

## コミットルール
- **必須**: `bash scripts/commit.sh "メッセージ"` または `git commit -m "メッセージ"`
- **禁止**: AI署名付きコミット（自動検出・拒否されます）

## プロジェクト固有の追加指示

### Ansibleコーディング規約
- **YAMLフォーマット**: 2スペースインデント、タブ禁止
- **命名規則**:
  - プレイブック: snake_case (例: `kafka_broker.yml`)
  - ロール: snake_case (例: `docker`, `wsl_common`)
  - 変数: snake_case (例: `server_name`)
- **タスク名**: 必ず`name:`属性を含める（分かりやすい英語で記述）
- **変数定義**: `defaults/main.yml`または`vars/main.yml`に配置
- **ハンドラー**: `handlers/main.yml`に配置

### テストフレームワーク
- **構文チェック**: `ansible-playbook --syntax-check <playbook.yml>`
- **Lintチェック**: `ansible-lint <playbook.yml>`
- **ロールテスト**: 各ロールの`tests/test.yml`を使用
  ```bash
  cd roles/<role_name>
  ansible-playbook -i tests/inventory tests/test.yml
  ```

### ビルドコマンド
該当なし（Ansibleプロジェクトのため）

### リントコマンド
```bash
# プレイブックの構文チェック
ansible-playbook --syntax-check playbooks/conf/wsl/wsl.yml

# Ansible Lintの実行
ansible-lint playbooks/conf/wsl/wsl.yml

# YAMLフォーマットチェック（yamllintがインストールされている場合）
yamllint playbooks/ roles/
```

### その他の制約事項

#### プラットフォーム別の考慮事項
- **WSL環境**:
  - Dockerインストールには管理者権限が必要
  - 日本語環境は`playbooks/conf/wsl/japanese.yml`で設定
- **Windows環境**:
  - WinRM設定が必要
  - `hosts.win`インベントリを使用
- **Raspberry Pi**:
  - ARM アーキテクチャ対応のパッケージを使用

#### ロール開発のベストプラクティス
1. **べき等性の確保**: すべてのタスクは複数回実行しても同じ結果になること
2. **README.md必須**: 各ロールに使用方法を記載したREADMEを含める
3. **依存関係**: `meta/main.yml`に依存ロールを明記
4. **変数のデフォルト値**: `defaults/main.yml`に適切なデフォルト値を設定
5. **OSサポート**: 複数OSをサポートする場合は`tasks/`内で分岐
   ```yaml
   - include_tasks: "{{ ansible_os_family }}.yml"
   ```

#### 実行例
```bash
# WSL基本設定
ansible-playbook -i hosts playbooks/conf/wsl/wsl.yml

# 特定ホストの設定
ansible-playbook -i hosts playbooks/conf/linux/basic.yml -e server=hostname

# Kafkaクラスタの起動
ansible-playbook -i hosts playbooks/operation/kafka_pseudo/start_all.yml
```

#### セキュリティガイドライン
- パスワードや秘密鍵はAnsible Vaultで暗号化
- センシティブ情報はno_log: trueを使用
- 実行ユーザーの権限を最小限に制限

#### 貢献ガイドライン
- 新しいロールは`roles/`ディレクトリに作成
- プレイブックは適切なカテゴリの`playbooks/conf/`または`playbooks/operation/`に配置
- 既存のロールやプレイブックのスタイルに合わせる 