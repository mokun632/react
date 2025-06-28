# React プロジェクト フォルダ構成概要

React リポジトリの主要なディレクトリとその役割の概要です。（プロジェクトルート `react/` からの相対パス）

```
react/
├── .github/         # GitHub アクションや Issue テンプレートなど
├── build/           # ビルド成果物 (通常は .gitignore される)
├── compiler/        # React Compiler (Forget) 関連のコード
│   ├── packages/    # Compiler の Babel プラグイン、ランタイムなど
│   └── docs/        # Compiler の設計ドキュメント
├── fixtures/        # テストやデバッグ用のサンプルアプリケーション群
│   ├── dom/         # DOM 関連のテストフィクスチャ
│   ├── concurrent/  # Concurrent Mode 関連のテストフィクスチャ
│   └── ...          # 他にも多くのテストシナリオ用フィクスチャ
├── packages/        # React のコア機能や関連ライブラリ (モノレポ構成)
│   ├── react/       # React のコア API (createElement, Component, Hooks など)
│   ├── react-dom/   # ブラウザ DOM レンダラー (createRoot, hydrateRoot など)
│   ├── react-dom-bindings/ # DOM と Reconciler を繋ぐ低レベルな実装
│   ├── react-reconciler/ # Fiber アーキテクチャと Reconciliation アルゴリズム
│   ├── scheduler/   # 協調スケジューリングの実装
│   ├── shared/      # 複数のパッケージ間で共有されるユーティリティコード
│   ├── react-native-renderer/ # React Native 用レンダラー
│   ├── react-art/   # ART ライブラリ用レンダラー
│   ├── react-devtools*/ # React DevTools 関連パッケージ
│   └── ...          # 他にも多くの内部パッケージや実験的パッケージ
├── scripts/         # ビルド、テスト、リリースなどの開発用スクリプト
│   ├── rollup/      # Rollup を使ったビルドスクリプト
│   ├── jest/        # Jest を使ったテスト関連スクリプト
│   └── ...
├── package.json     # プロジェクト全体の依存関係とスクリプト定義 (ルート)
├── yarn.lock        # 依存関係のロックファイル
├── README.md        # プロジェクト概要
└── CONTRIBUTING.md  # コントリビューションガイドライン
# ... 他、設定ファイル (.eslintrc.js, .gitignore など)
```

**主要ディレクトリの役割:**

*   **`packages/`**: React のコア機能や関連ライブラリが格納されています。モノレポ構成になっており、各機能が独立したパッケージとして管理されています。
*   **`compiler/`**: 実験的な React Compiler (旧称: Forget) のコードが含まれています。
*   **`fixtures/`**: 特定の機能やバグ修正を検証するための、小規模なテスト用 React アプリケーション群です。
*   **`scripts/`**: プロジェクトのビルド、テスト、リリースなどの開発ワークフローを自動化するためのスクリプト群です。

**初期レンダリング学習における重要パッケージ:**

1.  **`react`**: `createElement` や JSX ランタイム。
2.  **`react-dom`**: `createRoot`, `render` のエントリーポイント、DOM 操作。
3.  **`react-reconciler`**: Fiber ツリーの構築、Render/Commit フェーズの実行。
