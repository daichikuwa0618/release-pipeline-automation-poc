# リリースパイプライン自動化 PoC

このリポジトリは、GitHub Actions ワークフローを使用した自動化されたリリースパイプライン管理の概念実証（PoC）を示しています。この自動化により、並行開発とリリースプロセスのブランチ管理が効率化されます。

## 概要

この PoC では、以下を処理する自動化ワークフローを実装しています：

1. **バージョンブランチ作成**: 並行開発用の `develop-x.y.z` ブランチを自動設定
2. **リリースブランチ管理**: リリースブランチ作成時のブランチ遷移を自動管理
3. **ブランチ同期**: *(計画中)* `develop` の変更を適切なバージョンブランチに自動マージ

## 実装済みワークフロー

### 1. 開発バージョンブランチ自動化

**ファイル**: `.github/workflows/branch-automation-develop-version.yml`

**トリガー**: `develop-x.y.z` パターンのブランチ作成

**アクション**:

- 新しい `develop-x.y.z` ブランチから `chore/bump-x.y.z` ブランチを作成
- choreブランチでブランチバージョンに合わせて `pubspec.yaml` のバージョンを更新
- choreブランチでバージョン更新のコミットを作成
- `chore/bump-x.y.z` から `develop-x.y.z` へのバージョン更新プルリクエストを作成
- 詳細なセットアップ手順とレビューガイドラインを提供

**例**: `develop-1.2.0` を作成すると、ワークフローは：

- `develop-1.2.0` から `chore/bump-1.2.0` ブランチを作成
- choreブランチで `pubspec.yaml` のバージョンを `1.2.0+<現在のビルド番号>` に更新
- "chore: bump version to 1.2.0 for parallel development" というタイトルのPRを作成
- 並行開発セットアップのためのレビューポイントと次のステップを含む

**PRベースアプローチのメリット**:
- バージョン変更を適用前にコードレビュー可能
- 開発ブランチのクリーンなコミット履歴を維持
- バージョン形式と正確性の検証機会を提供
- GitFlowのベストプラクティスに従ったブランチ管理

### 2. リリースブランチ自動化

**ファイル**: `.github/workflows/branch-automation-release.yml`

**トリガー**: `release/x.y.z` パターンのブランチ作成

**アクション**:

- 次のマイナーバージョンを計算（例：`release/1.1.0` → `develop-1.2.0` を検索）
- 対応する `develop-x.(y+1).z` ブランチを検索
- 開発ブランチを `develop` にマージするPRを作成
- 並行開発からメイン開発ラインへの移行を促進

**例**: `release/1.1.0` を作成すると、ワークフローは：

- `develop-1.2.0` ブランチを検索
- 見つかった場合、`develop-1.2.0` → `develop` へのマージPRを作成
- 移行に関する詳細なコンテキストを含む

## 計画中の機能

### 3. 開発ブランチ同期 *(未実装)*

**仕様**:

- **トリガー**: `develop` ブランチへのプッシュ
- **アクション**:
  - すべての `develop-*` ブランチを検索
  - `develop` の変更を最小バージョン番号のブランチにマージ
  - コンフリクトがない場合は自動マージ（マージコミット作成、fast-forwardではない）
  - コンフリクトがある場合はPRを作成
  - `develop-x.y.z` ブランチが存在しない場合は何もしない

## ブランチ戦略

この自動化は並行開発ワークフローをサポートしています：

1. **メイン開発**: `develop` ブランチで実行
2. **並行開発**: 特定バージョン用の `develop-x.y.z` ブランチを使用
3. **リリース準備**: リリース候補用の `release/x.y.z` ブランチを使用
4. **自動移行**: リリース作成時、並行開発は自動的にメインラインにマージバック

## 使用方法

### 並行開発の開始

1. `develop` から `develop-x.y.z` ブランチを作成
2. ブランチをプッシュして自動化をトリガー
3. `chore/bump-x.y.z` から `develop-x.y.z` への自動生成PRをレビュー
   - バージョン更新が正しいことを確認
   - バージョン形式がプロジェクト標準に従っていることを確認
4. バージョンバンプPRをマージして開発ブランチに変更を適用
5. 必要に応じて新しい `develop-x.y.z` ブランチのブランチ保護ルールを更新
6. 設定済みブランチで開発を継続

### バージョンのリリース

1. 適切な開発ブランチから `release/x.y.z` ブランチを作成
2. 自動化により次のバージョン開発を `develop` に移行するPRが作成される
3. リリースプロセスを完了
4. 移行PRをマージして `develop` での開発を継続

## ワークフロー権限

両方のワークフローには以下が必要です：

- `contents: write` - コミット作成とプッシュ変更用
- `pull-requests: write` - プルリクエスト作成用

## 設定

### ブランチ命名規約

- **開発ブランチ**: `develop-x.y.z` (例：`develop-1.2.0`)
- **リリースブランチ**: `release/x.y.z` (例：`release/1.1.0`)

### バージョンファイル

ワークフローは以下の形式の `version` フィールドを持つ `pubspec.yaml` ファイルを想定しています：

```yaml
version: x.y.z+build_number
```

## メリット

- **自動セットアップ**: 開発ブランチ作成時の手動バージョン更新が不要
- **一貫したプロセス**: 適切なドキュメント付きの標準化されたPR作成
- **エラー削減**: バージョン番号とブランチ関係の自動計算
- **明確な移行**: 開発フェーズ間の移行のための明示的なPR
- **監査証跡**: 詳細なコンテキスト付きのPRを通じたすべての変更の追跡

---

この PoC は、開発プロセスの明確な監視と制御を維持しながら、GitHub Actions がリリースパイプライン管理を大幅に効率化できることを実証しています。
