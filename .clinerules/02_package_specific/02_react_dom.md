# React DOM パッケージに関するルール

このドキュメントは、`packages/react-dom` および `packages/react-dom-bindings` ディレクトリに含まれる React DOM 関連パッケージのコードを理解・分析する際のガイドラインを定めます。

## 1. パッケージ概要

React DOM パッケージは、React のコア機能を Web ブラウザ上の DOM と連携させるブリッジの役割を果たします。主に以下の機能を提供します：

* **DOM レンダリング**: React 要素を DOM 要素に変換・更新
* **イベントシステム**: React の合成イベントシステムの実装
* **プロパティ管理**: DOM プロパティ、属性、スタイルの処理
* **Server Side Rendering**: サーバー側でのHTMLレンダリング（react-dom/server）

*   **主要ファイル:**
    *   `packages/react-dom/src/client/ReactDOMRoot.js`: `createRoot` APIの実装
    *   `packages/react-dom/src/client/ReactDOMClient.js`: クライアントサイドのエントリーポイント
    *   `packages/react-dom/src/server/ReactDOMServer.js`: サーバーサイドレンダリングのエントリーポイント
    *   `packages/react-dom-bindings/src/client/ReactDOMComponent.js`: DOM 要素の処理
    *   `packages/react-dom-bindings/src/events/`: イベントシステムの実装

## 2. 分析ポイント

*   **レンダリングフロー**
    *   `createRoot`/`createRoot().render()` の内部動作
    *   `ReactDOM.hydrateRoot` によるハイドレーションプロセス
    *   Server Side Rendering の実装詳細 (`renderToString`, `renderToPipeableStream` など)

*   **DOM 操作**
    *   React 要素から DOM 要素への変換プロセス
    *   プロパティと属性の処理方法
    *   DOM 更新時の最適化手法

*   **イベントシステム**
    *   イベント委譲（event delegation）の実装
    *   合成イベント（SyntheticEvent）の生成
    *   イベントの伝播とキャプチャリング
    *   バッチ処理と最適化

*   **Feature Flags**
    *   `shared/ReactFeatureFlags.js` で定義される機能フラグが DOM 実装にどう影響するか

## 3. Code Reading 戦略

*   `packages/react-dom/src/client/ReactDOMRoot.js` の `createRoot` から始め、Fiber ツリーの作成と更新の流れを追う
*   `react-dom-bindings` の実装を調査して、実際の DOM 操作がどのように行われるか理解する
*   イベントシステムの実装を確認し、React 独自のイベントシステムがブラウザネイティブのイベントとどう連携するか把握する

## 4. Code Tracing 例

*   `ReactDOM.createRoot(container).render(<App />)` の処理フロー:
    1. `createRoot` -> `createContainer` -> `createFiberRoot`
    2. `root.render` -> `updateContainer` -> `scheduleUpdateOnFiber`
    3. Reconciler によるレンダリング処理
    4. `commitRoot` -> DOM への反映

*   DOM 要素の生成と更新:
    1. Fiber ノードの `completeWork` フェーズでの DOM 要素生成
    2. `finalizeInitialChildren` によるプロパティ設定
    3. Commit フェーズの `mutation` パートでの DOM ツリーへの挿入

## 5. Cline への指示

*   レンダリングプロセスに関する質問の場合:
    *   `react-dom` から `react-reconciler` への連携フローを調査
    *   Fiber ツリーからDOMツリーがどのように構築されるかを説明
    *   最適化ポイント（例：バッチ更新）を強調

*   DOM 操作に関する質問の場合:
    *   `react-dom-bindings` 内の対応するコード箇所を特定
    *   プロパティ、属性、スタイルの処理方法の違いを説明
    *   ブラウザ間の互換性対応がどう処理されているかを解説

*   イベント関連の質問の場合:
    *   イベントシステムのアーキテクチャと登録方法を説明
    *   特定のイベントタイプがどのように処理されるかをコードで例示
    *   パフォーマンス最適化がどのように実装されているかを解説
