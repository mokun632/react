# React イベントシステムの基本フロー分析

`createRoot` の分析中に明らかになった、React のイベント処理の基本的な流れを記録する。

## 1. イベント委任 (Event Delegation)

*   **`listenToAllSupportedEvents` (`DOMPluginEventSystem.js` L258):**
    *   `createRoot` の過程で呼び出され、React がサポートする全ての DOM イベント (click, change など多数) に対するリスナーを、アプリケーションの **ルートコンテナ要素に一括で登録** する。
    *   個々の要素ではなくルートでイベントを捕捉するため、パフォーマンスとメモリ効率が良い。動的に追加された要素のイベントも処理可能。

## 2. DOM ノードと Fiber の紐付け

*   **`markContainerAsRoot` (`ReactDOMComponentTree.js`):**
    *   ルートコンテナ要素に、対応する `HostRootFiber` への参照を内部プロパティ (例: `internalContainerInstanceKey`) として格納する。これはコンテナが React ルートであることを示すマークとなる。
*   **`precacheFiberNode` / `updateFiberProps` (`ReactDOMComponentTree.js`):**
    *   `HostComponent` Fiber がレンダリングする **個々の DOM 要素** にも、対応する Fiber インスタンスへの参照が内部プロパティ (例: `internalInstanceKey`, `__reactFiber$xxx`) として格納される。
*   **目的:** この紐付けにより、DOM ノードから対応する Fiber ノードへの **逆引き** が可能になる。

## 3. イベント発生時の Fiber 特定プロセス

1.  **イベント発生と捕捉:** ブラウザで DOM イベントが発生し、ルートコンテナのリスナーが捕捉する。
2.  **ターゲット特定:** ルートリスナーはネイティブイベントオブジェクトから `event.target` (イベントが最初に発生した DOM ノード) を取得する。
3.  **`getClosestInstanceFromNode` (`ReactDOMComponentTree.js` L129) 呼び出し:** イベントディスパッチャが `event.target` を引数にこの関数を呼び出す。
4.  **Fiber 探索:**
    *   まず `event.target` 自体に Fiber が紐付いているか確認する。
    *   見つかればその Fiber を返す。
    *   見つからなければ、**親 DOM ノードを上方向に辿りながら**、Fiber が紐付いているノードを探す。
        *   これは、`event.target` が React 管理外の要素 (例: SVG の子要素、`dangerouslySetInnerHTML` 内の要素、Shadow DOM 内要素) である場合や、テキストノードの場合に対応するため。
    *   Fiber が見つかるか、ルートコンテナに到達するまで探索を続ける。
5.  **結果:** イベントを処理すべき最も近い React 管理下の要素に対応する Fiber が特定される。

## 4. Fiber ツリー内でのイベント伝播シミュレーション

1.  **起点:** `getClosestInstanceFromNode` で特定された Fiber を起点とする。
2.  **キャプチャフェーズ:** ルート Fiber から起点 Fiber まで **下方向** に Fiber ツリーを辿り、各 Fiber の `props` に `onXxxxCapture` (例: `onClickCapture`) があれば、そのハンドラ関数を収集する。
3.  **バブリングフェーズ:** 起点 Fiber からルート Fiber まで **上方向** に Fiber ツリーを辿り、各 Fiber の `props` に `onXxxx` (例: `onClick`) があれば、そのハンドラ関数を収集する。
4.  **ハンドラ実行:** 収集されたハンドラ関数を **順番通り** (キャプチャ → バブル) に実行する。各ハンドラには、ブラウザ差異を吸収した **合成イベント (SyntheticEvent)** オブジェクトが渡される。

**結論:**

React のイベントシステムは、イベント委任によって効率的にイベントを捕捉し、DOM ノードと Fiber の紐付けを利用してイベント発生源の Fiber を特定する。その後、Fiber ツリー内でキャプチャ/バブルフェーズをシミュレートし、収集したハンドラを合成イベントと共に実行することで、宣言的なイベント処理を実現している。`event.target` から DOM を上に辿る処理は、React 管理外の要素やポータルからのイベントを正しく処理するために不可欠である。
