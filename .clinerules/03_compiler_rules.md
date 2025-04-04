# React Compiler に関するルール

このドキュメントは、`compiler/` ディレクトリに含まれる React Compiler のコードを理解・分析する際のガイドラインを定めます。

## 1. React Compiler の概要

React Compiler は React コンポーネントのパフォーマンスを自動的に最適化するためのコンパイラツールです。従来は手動で最適化する必要があった再レンダリングの問題を、静的解析とコード変換によって自動的に解決します。

*   **主要ディレクトリ/ファイル:**
    *   `compiler/packages/babel-plugin-react-compiler/`: Babel プラグイン実装
    *   `compiler/packages/react-compiler-runtime/`: コンパイル後のコードで使用されるランタイム機能
    *   `compiler/packages/eslint-plugin-react-compiler/`: ESLint との統合
    *   `compiler/docs/`: 設計ドキュメントと開発ガイド

## 2. 分析ポイント

*   **最適化手法**
    *   不要な再レンダリングを削減するためのメモ化戦略
    *   静的解析による依存性の追跡
    *   自動的な `useMemo`/`useCallback` の挿入方法
    *   クロージャの最適化手法

*   **コンパイル変換**
    *   入力コードから出力コードへの変換パターン
    *   AST 変換の仕組みとアルゴリズム
    *   React コアライブラリとの連携方法

*   **ランタイムサポート**
    *   コンパイラが生成したコードをサポートするランタイム関数
    *   最適化されたレンダリングのための特殊処理

*   **デバッグとトラブルシューティング**
    *   コンパイル過程の診断情報の出力方法
    *   最適化に関する警告やエラーの処理

## 3. Code Reading 戦略

*   まず `compiler/docs/DESIGN_GOALS.md` で設計目標を理解する
*   次に `compiler/packages/babel-plugin-react-compiler/src/` の主要なエントリポイントを特定する
*   コンパイル過程のパイプラインを追跡し、入力から出力までの変換を理解する
*   テストケースを通じて実際の変換例を確認する

## 4. Code Tracing 例

*   React コンポーネントのコンパイル例:
    1. 入力コンポーネント（元の React コード）
    2. 静的解析によるコンポーネントの依存関係の特定
    3. 最適化のための AST 変換
    4. 出力コンポーネント（最適化された React コード）

*   ランタイム関数の呼び出しフロー:
    1. コンパイル後のコンポーネントでのランタイム関数の使用
    2. ランタイムの内部処理と React コアとの連携
    3. メモ化と依存性追跡の実装

## 5. Cline への指示

*   コンパイラの最適化手法について質問された場合:
    *   具体的な最適化パターンを例示し、なぜそれがパフォーマンス向上に寄与するかを説明
    *   変換前と変換後のコードの違いを明確に示す
    *   該当する最適化が適用される条件を詳細に解説

*   コンパイラのエラーや警告について質問された場合:
    *   エラーメッセージの意味を説明し、対処方法を提案
    *   コンパイラの制限事項と回避策を示す
    *   最適化が適用できないケースの理由を解析

*   コンパイラの内部実装について質問された場合:
    *   該当する変換ロジックが実装されているファイルと関数を特定
    *   変換アルゴリズムのステップを説明
    *   使用されているデータ構造やパターンを解説

## 6. 注意点

*   React Compiler はまだ発展途上の機能であり、仕様が変更される可能性がある
*   最新の情報を `compiler/README.md` や Meta のブログ記事などで確認することが重要
*   コンパイラに対応していない React パターンや特殊なケースにも注意が必要
*   ランタイムと静的解析の両方が連携することで最適化が実現される仕組みを理解することが重要
