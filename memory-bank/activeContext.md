# Active Context: 現在の状況と次のステップ

## 1. 現在のフォーカス

*   **React 内部実装の学習**: `.clinerules` を活用し、コード分析や概念理解を進める。
*   **現在のテーマ**: イベントシステムの基本フローの理解完了。次は `updateContainer` からスケジューリング/レンダリングループへ。

## 2. 最近の決定事項・変更

*   React プロジェクトのフォルダ構成を確認し、学習用にカスタマイズされたディレクトリを除いた構成を `learning-log/folder-structure.md` に記録した。
*   現在のプロジェクトが React 公式リポジトリ (またはそのフォーク) であることを相互に確認した。
*   `.clinerules` ディレクトリと全ルールファイル (`00`〜`07`) の作成完了。
*   Memory Bank のコアファイル (`projectbrief.md`, `productContext.md`, `activeContext.md`, `systemPatterns.md`, `techContext.md`, `progress.md`) の作成・初期化完了。
*   `learning-log/initial-rendering.md` 内の `completeWork` 関数の参照行番号を更新。
*   学習ログの整理:
    *   `learning-log/initial-rendering.md` を `learning-log/01_initial_rendering/00_overview.md` に移動・改名。
    *   `learning-log/01_initial_rendering/02_createRoot_analysis.md` を新規作成し、`createRoot` から `scheduleUpdateOnFiber` までの流れを分析。
    *   `learning-log/01_initial_rendering/03_scheduling_and_workloop.md` を新規作成し、`scheduleUpdateOnFiber` 以降の Render/Commit フェーズの分析内容を `02_createRoot_analysis.md` から移動。
*   `.clinerules/00_overview.md` に図解の推奨を追記。
*   `FiberRoot` と `HostRoot Fiber` の循環参照の理由を考察 (`02_createRoot_analysis.md`)。詳細な理解はレンダリングループ分析後に後回し。
*   学習ログの記録は、一連の分析が完了してからまとめて行うルールを `.clinerules/00_overview.md` に追記。
*   複雑な質問への対応ルールを `.clinerules/00_overview.md` に追記。
*   `createRoot` の初期化プロセス (キャッシュ、初期状態、更新キュー、後処理) の分析結果を `learning-log/01_initial_rendering/02_createRoot_analysis.md` に記録完了。
*   イベントシステムの基本フロー (`listenToAllSupportedEvents`, `markContainerAsRoot`, `getClosestInstanceFromNode`, 伝播シミュレーション) を分析し、`learning-log/02_event_system/01_basic_flow.md` に記録完了。
*   Fiber の基本的な種類と構造 (`HostComponent`, `HostSingleton` 含む) について分析し、`learning-log/03_fiber_architecture/01_fiber_types_and_structure.md` に記録完了。
*   **Suspense と Offscreen の統合の学習**: `Suspense` が内部的に `Offscreen` コンポーネントを使用して子要素の表示・非表示を制御していることを、`packages/react-reconciler/src/ReactFiberBeginWork.js` のコード分析を通じて確認し、`learning-log/03_fiber_architecture/02_suspense_and_offscreen_integration.md` に記録完了。

## 3. 次のステップ (次回セッションから)

*   **未定**: 次の学習テーマを決定します。

## 4. 学習上の気づき・パターン（随時更新）

*   `.clinerules` という形で学習ガイドラインを明文化することで、AIエージェントとの連携をスムーズにし、学習プロセスを体系化できる可能性が見えてきた。
*   `createRoot` の呼び出しは、複数の主要な関数 (`createContainer`, `createFiberRoot`) を経由して `FiberRoot` と `HostRoot Fiber` を生成・初期化する流れになっている。
*   `FiberNode` は個々の要素、`FiberRootNode` はアプリケーション全体を管理する役割を持つ。
*   `HostComponent` と `HostSingleton` はネイティブ要素を表す Fiber タグであり、後者はシングルトン要素の最適化に使われる。
