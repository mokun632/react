# Fiber の種類と構造

React の内部実装における中心的なデータ構造である Fiber ノードについて、その基本的な構造と主要な種類 (タグ) を記録する。

## 1. Fiber ノードの基本構造 (`ReactFiber.js`)

Fiber ノードは、コンポーネント、DOM 要素、またはその他の作業単位を表す JavaScript オブジェクトである。主なプロパティは以下の通り。

*   **`tag`**: Fiber の種類を示すタグ (数値)。`ReactWorkTags.js` で定義されている (例: `FunctionComponent`, `ClassComponent`, `HostComponent`)。`beginWork` や `completeWork` はこのタグを見て処理を分岐する。
*   **`key`**: Reconciliation で要素を特定するためのキー (`key` prop)。
*   **`elementType`**: Fiber が生成される元の要素タイプ (例: 関数コンポーネントの関数自身、クラスコンポーネントのクラス、DOM 要素のタグ名文字列 'div')。
*   **`type`**: Fiber が表すコンポーネントまたは要素のタイプ。`elementType` と同じ場合もあれば、`LazyComponent` のように解決後のコンポーネントが入る場合もある。
*   **`stateNode`**: Fiber に関連付けられたローカル状態やインスタンスへの参照。
    *   `HostComponent`: 対応する DOM 要素インスタンス。
    *   `ClassComponent`: コンポーネントクラスのインスタンス。
    *   `HostRoot`: 対応する `FiberRootNode` インスタンス (循環参照)。
    *   `FunctionComponent`: 通常は `null`。
*   **`return`**: 親 Fiber へのポインタ。ツリー構造を形成する。
*   **`child`**: 最初の子 Fiber へのポインタ。
*   **`sibling`**: 次の兄弟 Fiber へのポインタ。
*   **`pendingProps`**: 新しいレンダリングで処理されるべき props。
*   **`memoizedProps`**: 前回のレンダリングで使われた props。
*   **`memoizedState`**: 前回のレンダリングで使われた state。`HostRoot` では `RootState`、関数コンポーネントでは Hooks のリンクリストなど。
*   **`updateQueue`**: この Fiber に対する更新 (Update オブジェクト) を保持するキュー。主に `ClassComponent` と `HostRoot` で使用される。
*   **`flags`**: この Fiber に適用されるべき副作用 (DOM 操作など) を示すビットマスク (`ReactFiberFlags.js`)。
*   **`subtreeFlags`**: この Fiber の **子孫** が持つ副作用フラグのビットマスク。Commit フェーズでの効率的なツリー走査に使われる。
*   **`lanes`**: この Fiber にスケジュールされている更新の優先度レーン。
*   **`childLanes`**: この Fiber の **子孫** にスケジュールされている更新の優先度レーン。
*   **`alternate`**: `current` ツリーと `workInProgress` ツリーの間で対応する Fiber へのポインタ。ダブルバッファリングを実現する。

Fiber ノードは `createFiber` (`ReactFiber.js` L218) や、特定のタイプに応じたヘルパー関数 (`createHostRootFiber` など) によって生成される。

## 2. 主要な Fiber タグ (`ReactWorkTags.js`)

`createRoot` の分析過程で登場した主要なタグは以下の通り。

*   **`HostRoot` (3):** Fiber ツリー全体のルートを表す特別な Fiber。`stateNode` は `FiberRootNode` を指す。`updateHostRoot` (`ReactFiberBeginWork.js`) で処理される。
*   **`HostComponent` (5):** ネイティブなプラットフォーム要素 (DOM 環境では `div`, `span`, `button` など) を表す Fiber。`stateNode` は対応する DOM 要素インスタンスを指す。`updateHostComponent` (`ReactFiberBeginWork.js`) や `completeWork` (`ReactFiberCompleteWork.js`) で DOM 生成・更新準備が行われる。
*   **`HostSingleton` (6):** シングルトンとして扱われる特別なネイティブ要素 (DOM 環境では `<html>`, `<head>`, `<body>`) を表す Fiber。`supportsSingletons` フラグが有効な場合に使用される。`updateHostSingleton` (`ReactFiberBeginWork.js`) や `completeWork` で処理される。

これら以外にも、`FunctionComponent` (0), `ClassComponent` (1), `Fragment` (7), `SuspenseComponent` (13) など、多くのタグが存在し、それぞれ異なる処理ロジックを持つ。

## 3. 関連ファイル

*   `packages/react-reconciler/src/ReactFiber.js`: Fiber ノードの定義と生成関数。
*   `packages/react-reconciler/src/ReactWorkTags.js`: Fiber タグの定義。
*   `packages/react-reconciler/src/ReactFiberFlags.js`: 副作用フラグの定義。
*   `packages/react-reconciler/src/ReactFiberBeginWork.js`: Render フェーズの下降処理 (タグに応じた分岐)。
*   `packages/react-reconciler/src/ReactFiberCompleteWork.js`: Render フェーズの上昇処理 (タグに応じた分岐、DOM 生成準備など)。

## 4. 次のステップ

*   Fiber アーキテクチャの全体像 (ツリー構築、更新プロセス) をさらに詳しく学ぶ。
*   他の主要な Fiber タグ (`FunctionComponent`, `ClassComponent`, `SuspenseComponent` など) の `beginWork`/`completeWork` における処理を分析する。
