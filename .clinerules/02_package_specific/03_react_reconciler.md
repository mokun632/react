# React Reconciler パッケージに関するルール

このドキュメントは、`packages/react-reconciler` ディレクトリに含まれる React Reconciler パッケージのコードを理解・分析する際のガイドラインを定めます。

## 1. パッケージ概要

React Reconciler は React のコアアルゴリズムを実装するパッケージで、Fiber アーキテクチャによる仮想 DOM の差分検出と更新適用を担います。このパッケージは特定のレンダリング環境（DOM、Native など）に依存しない抽象レイヤーとして機能します。

*   **主要ファイル:**
    *   `packages/react-reconciler/src/ReactFiberWorkLoop.js`: レンダリングループの実装
    *   `packages/react-reconciler/src/ReactFiberBeginWork.js`: 下降フェーズ（`beginWork`）の実装
    *   `packages/react-reconciler/src/ReactFiberCompleteWork.js`: 上昇フェーズ（`completeWork`）の実装
    *   `packages/react-reconciler/src/ReactFiberCommitWork.js`: Commit フェーズの実装
    *   `packages/react-reconciler/src/ReactFiberHooks.js`: Hooks の実装
    *   `packages/react-reconciler/src/ReactFiberLane.js`: 優先度（Lane）モデルの実装

## 2. 分析ポイント

*   **Fiber アーキテクチャ**
    *   Fiber ツリーの作成と更新プロセス
    *   Work-in-Progress ツリーと Current ツリーの関係性
    *   Fiber ノード間のリンク構造（`child`, `sibling`, `return` など）

*   **Render フェーズ（作業スケジュール + 差分計算）**
    *   `workLoopSync`/`workLoopConcurrent` によるレンダリングループ
    *   `beginWork`: 各 Fiber タイプに応じた処理と子 Fiber の生成
    *   `completeWork`: プラットフォーム固有のインスタンス生成と副作用マーキング
    *   副作用リストの構築

*   **Commit フェーズ（DOM 更新など）**
    *   `commitRoot`: 副作用リストに基づいた更新適用
    *   Before Mutation、Mutation、Layout の各サブフェーズ
    *   `commitPlacement`、`commitWork`、`commitDeletion` などの更新操作

*   **優先度とスケジューリング**
    *   Lane モデルによる更新の優先度付け
    *   協調スケジューリングの実装（Scheduler との連携）
    *   更新のバッチ処理と割り込み機能

*   **Hook システム**
    *   `renderWithHooks`: 関数コンポーネントの実行環境設定
    *   各種フックの内部実装（`useState`, `useEffect`, `useContext` など）
    *   フックの依存配列と再評価ロジック

## 3. Code Reading 戦略

*   まず `ReactFiberReconciler.js` の公開 API から追跡を開始する
*   `updateContainer` -> `scheduleUpdateOnFiber` -> `performSyncWorkOnRoot`/`performConcurrentWorkOnRoot` と続けて読む
*   Fiber ツリーの作成・更新・コミットの全フローを理解する
*   特定の機能（例：Hook の実装）に焦点を当てたトレースも行う

## 4. Code Tracing 例

*   Render フェーズの追跡:
    1. `performSyncWorkOnRoot`/`performConcurrentWorkOnRoot`
    2. `renderRootSync`/`renderRootConcurrent`
    3. `workLoopSync`/`workLoopConcurrent`
    4. `performUnitOfWork` -> `beginWork` -> `completeWork`
    5. `finishQueueingConcurrentUpdates`

*   Commit フェーズの追跡:
    1. `commitRoot`
    2. `commitBeforeMutationEffects`
    3. `commitMutationEffects`
    4. `commitLayoutEffects`
    5. `requestPaint`/`flushPassiveEffects`（非同期）

*   Hooks の呼び出しフロー:
    1. `renderWithHooks`
    2. `HooksDispatcherOnMount`/`HooksDispatcherOnUpdate`
    3. 各 Hook 関数（`mountState`, `updateState` など）
    4. `dispatchSetState`/`dispatchReducerAction`

## 5. Cline への指示

*   Reconciliation アルゴリズムに関する質問の場合:
    *   Fiber アーキテクチャの各部分の役割とデータフローを説明
    *   Render フェーズと Commit フェーズの違いと連携を説明
    *   具体的なコード例を提示して、更新プロセスを追跡

*   パフォーマンス関連の質問の場合:
    *   Lane モデルと優先度システムの実装を調査
    *   Concurrent Mode の実装と利点を説明
    *   時間スライスの仕組みとユーザー操作への応答性向上方法を解説

*   Hook に関する質問の場合:
    *   `ReactFiberHooks.js` の具体的な実装を調査
    *   Hook の内部データ構造とリスト管理の仕組みを説明
    *   特定の Hook の動作や最適化の詳細を解説
