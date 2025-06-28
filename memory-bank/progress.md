# Progress: 学習の進捗と現状

## 1. 完了したこと

*   React 公式リポジトリのフォーク。
*   AI エージェント Cline との連携準備。
*   Memory Bank の導入とコアファイル (`projectbrief.md`, `productContext.md`, `activeContext.md`, `systemPatterns.md`, `techContext.md`, `progress.md`) の作成・初期化。
*   React 内部実装学習のための `.clinerules` ディレクトリ構造と全ルールファイル (`00`〜`07`) の作成。
*   学習記録用の `learning-log` ディレクトリ作成と初期ログ (`react-philosophy.md`, `initial-rendering.md`) の作成。
*   `learning-log/initial-rendering.md` 内の `completeWork` 関数の参照行番号を更新。
*   最初の学習テーマとして「初期レンダリング」を選定し、大まかな学習計画を立てた。
*   React プロジェクトのフォルダ構成を確認し、学習ログ (`learning-log/folder-structure.md`) に記録した。
*   現在のプロジェクトが React 公式リポジトリ (またはそのフォーク) であることを確認した。
*   学習ログの整理 (`learning-log/01_initial_rendering/` 作成、`initial-rendering.md` -> `00_overview.md` へ移動・改名、`02_createRoot_analysis.md` 作成)。
*   `.clinerules/00_overview.md` に図解の推奨を追記。
*   `createRoot` の呼び出しフローを分析 (`ReactDOMRoot.js` -> `ReactFiberReconciler.js` -> `ReactFiberRoot.js`)。
*   `FiberNode` と `FiberRootNode` の違い、`HostComponent` と `HostSingleton` の役割について理解。
*   学習ログ (`learning-log/02_createRoot_analysis.md`) に `createRoot` 分析の初期内容と図を記録。
*   `scheduleUpdateOnFiber` から始まる Render/Commit フェーズの分析 (`ReactFiberWorkLoop.js`, `ReactFiberBeginWork.js`, `ReactFiberCompleteWork.js`, `ReactFiberCommitWork.js`) を実施。
*   学習ログの構成を見直し、`02_createRoot_analysis.md` から `learning-log/01_initial_rendering/03_scheduling_and_workloop.md` へ関連内容を移動。
*   `.clinerules/00_overview.md` にログ記録ワークフローと複雑な質問への対応ルールを追記。
*   `createRoot` の初期化プロセス (キャッシュ、初期状態、更新キュー、後処理) の分析結果を `learning-log/01_initial_rendering/02_createRoot_analysis.md` に記録完了。
*   イベントシステムの基本フロー (`listenToAllSupportedEvents`, `markContainerAsRoot`, `getClosestInstanceFromNode`, 伝播シミュレーション) を分析し、`learning-log/02_event_system/01_basic_flow.md` に記録完了。
*   Fiber の基本的な種類と構造 (`HostComponent`, `HostSingleton` 含む) について分析し、`learning-log/03_fiber_architecture/01_fiber_types_and_structure.md` に記録完了。
*   `updateContainerImpl` 内の `createUpdate`, `enqueueUpdate` (および関連する `enqueueConcurrentClassUpdate`, 内部 `enqueueUpdate`, `getRootForUpdatedFiber`) の役割、特に Concurrent Mode における更新のバッチングメカニズムについて分析・議論した。
*   `startUpdateTimerByLane` が Profiler 用の機能であることを確認した。
*   **Suspense と Offscreen の統合の学習**: `Suspense` が内部的に `Offscreen` コンポーネントを使用して子要素の表示・非表示を制御していることを、`packages/react-reconciler/src/ReactFiberBeginWork.js` のコード分析を通じて確認し、`learning-log/03_fiber_architecture/02_suspense_and_offscreen_integration.md` に記録完了。

## 2. 現在進行中・次のタスク (次回セッションから)

*   **未定**: 次の学習テーマを決定します。

## 3. 未着手の主要な学習項目

*   Fiber アーキテクチャの全体像 (構造、役割、ツリー構築プロセス)。
*   React Lane (レーンモデル) と優先度管理。
*   Hooks の内部実装 (`useState`, `useEffect` など)。
*   Context API の実装。
*   Scheduler パッケージとの連携、優先度管理。
*   イベントハンドリングの仕組み。
*   React Compiler の詳細。
*   テストコード (`fixtures/`) の活用。
*   SSR (Server-Side Rendering) / Server Components 関連の実装。

## 4. 既知の問題・課題

*   コードベースの全体像を把握するのが難しい。
*   特定の機能に関連するコード箇所を見つけるのが困難な場合がある (Cline の検索機能に期待)。
*   Flow 型に慣れていない場合、コードの読解に時間がかかる可能性がある。

## 5. プロジェクト判断の変遷（随時更新）

*   (まだ特になし)
