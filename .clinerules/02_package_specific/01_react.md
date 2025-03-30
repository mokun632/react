# React コアパッケージに関するルール

このドキュメントは、`packages/react` ディレクトリに含まれる React コアパッケージのコードを理解・分析する際のガイドラインを定めます。

## 1. パッケージ概要

React コアパッケージは、React の基本的な API (createElement, Component, hooks など) を提供します。ただし、実際のレンダリングロジックは含まれておらず、それらは `react-reconciler` や `react-dom` などの他のパッケージに委譲されています。

*   **主要ファイル:**
    *   `packages/react/src/React.js`: メインエントリポイント、公開 API の定義
    *   `packages/react/src/ReactHooks.js`: フック API の実装
    *   `packages/react/src/ReactElement.js`: React 要素 (createElement) の実装
    *   `packages/react/src/ReactContext.js`: Context API の実装

## 2. 分析ポイント

*   **React API**
    *   `createElement` がどのようにして JSX をオブジェクト表現に変換するか
    *   `jsx` ランタイム関数と `createElement` の関係
    *   各 API の内部実装と他のパッケージへの参照
    *   `_current` や他の内部変数がどのように利用されているか

*   **Component API**
    *   `Component` と `PureComponent` クラスの実装とその違い
    *   ライフサイクルメソッドの定義とその呼び出し方法
    *   クラスコンポーネントと関数コンポーネントの内部実装の違い

*   **その他の API**
    *   `React.memo`, `React.forwardRef`, `React.lazy` などの高階コンポーネントの実装
    *   `Suspense`, `Transition` などの新機能の実装方法

## 3. Code Reading 戦略

*   まず `packages/react/src/React.js` から読み始め、エクスポートされる API を特定する
*   各 API の実装を追跡し、それぞれの役割と他パッケージとの連携方法を理解する
*   JSX 変換から始まり、レンダリングまでのデータフローを追跡する

## 4. Code Tracing 例

*   JSX コードが React 要素に変換される過程:
    1. JSX → `jsx()` または `createElement()`
    2. `ReactElement.js` での要素オブジェクト生成
    3. 要素の検証と最適化

*   フック呼び出しの流れ:
    1. `useXXX` 呼び出し → `ReactHooks.js`
    2. `ReactCurrentDispatcher.current.useXXX` 経由で `react-reconciler` の実装にディスパッチ
    3. `react-reconciler` 内での実際の処理

## 5. Cline への指示

*   React API の動作や役割について質問された場合:
    *   まず該当する API が定義されている正確なファイルと行番号を特定
    *   その API の内部動作と目的を説明
    *   必要に応じて関連するコード部分を引用
    *   API が他のパッケージとどのように連携しているかを説明

*   特定のコンポーネント型 (クラス/関数) の処理方法について質問された場合:
    *   React パッケージでの定義部分と Reconciler での処理部分の両方を説明
    *   コンポーネントライフサイクルにおける処理の流れを説明

*   Hook に関する質問については、`ReactHooks.js` と `ReactFiberHooks.js` の両方を参照して回答
