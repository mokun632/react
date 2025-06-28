# Suspense と Offscreen の統合

React の `Suspense` コンポーネントは、内部的に `Offscreen` コンポーネントを使用してその子要素の表示・非表示を制御しています。これにより、サスペンド状態のコンテンツを効率的に管理し、フォールバック表示への切り替えや、非表示状態でのレンダリング（プリレンダリング）を可能にしています。

## 実装箇所

この統合の主要なロジックは、`packages/react-reconciler/src/ReactFiberBeginWork.js` ファイル内の `updateSuspenseComponent` 関数にあります。

### `updateSuspenseComponent` (packages/react-reconciler/src/ReactFiberBeginWork.js:1808)

この関数は、`SuspenseComponent` Fiber の `beginWork` フェーズで呼び出され、Suspense の状態（サスペンドしているか、解決済みか）に基づいて、プライマリコンテンツをラップする `Offscreen` Fiber を作成または更新します。

### ヘルパー関数による Offscreen Fiber の管理

`updateSuspenseComponent` は、以下のヘルパー関数を介して `Offscreen` Fiber を操作します。

1.  **`mountSuspensePrimaryChildren` (packages/react-reconciler/src/ReactFiberBeginWork.js:2011)**
    *   **目的**: Suspense が最初にマウントされ、サスペンドしていない（または解決済みである）場合に、プライマリコンテンツをレンダリングするために使用されます。
    *   **Offscreen の `mode`**: `'visible'` に設定された `Offscreen` Fiber を作成します。
    *   **関連コード**:
        ```javascript
        function mountSuspensePrimaryChildren(
          workInProgress: Fiber,
          primaryChildren: $FlowFixMe,
          renderLanes: Lanes,
        ) {
          const mode = workInProgress.mode;
          const primaryChildProps: OffscreenProps = {
            mode: 'visible', // Offscreen の mode が 'visible'
            children: primaryChildren,
          };
          const primaryChildFragment = mountWorkInProgressOffscreenFiber( // Offscreen Fiber を作成
            primaryChildProps,
            mode,
            renderLanes,
          );
          primaryChildFragment.return = workInProgress;
          workInProgress.child = primaryChildFragment;
          return primaryChildFragment;
        }
        ```

2.  **`mountSuspenseFallbackChildren` (packages/react-reconciler/src/ReactFiberBeginWork.js:2029)**
    *   **目的**: Suspense が最初にマウントされ、サスペンドした場合に、プライマリコンテンツを非表示にし、フォールバックコンテンツをレンダリングするために使用されます。
    *   **Offscreen の `mode`**: プライマリコンテンツをラップする `Offscreen` Fiber の `mode` を `'hidden'` に設定します。
    *   **関連コード**:
        ```javascript
        function mountSuspenseFallbackChildren(
          workInProgress: Fiber,
          primaryChildren: $FlowFixMe,
          fallbackChildren: $FlowFixMe,
          renderLanes: Lanes,
        ) {
          // ...
          const primaryChildProps: OffscreenProps = {
            mode: 'hidden', // Offscreen の mode が 'hidden'
            children: primaryChildren,
          };
          // ...
          primaryChildFragment = mountWorkInProgressOffscreenFiber( // Offscreen Fiber を作成
            primaryChildProps,
            mode,
            NoLanes,
          );
          // ...
          primaryChildFragment.sibling = fallbackChildFragment;
          workInProgress.child = primaryChildFragment;
          return fallbackChildFragment;
        }
        ```

3.  **`updateSuspensePrimaryChildren` (packages/react-reconciler/src/ReactFiberBeginWork.js:2110)**
    *   **目的**: Suspense が更新され、サスペンドしていない場合に、既存の `Offscreen` Fiber を更新します。
    *   **Offscreen の `mode`**: `'visible'` に設定されます。

4.  **`updateSuspenseFallbackChildren` (packages/react-reconciler/src/ReactFiberBeginWork.js:2139)**
    *   **目的**: Suspense が更新され、サスペンドした場合に、既存の `Offscreen` Fiber を更新します。
    *   **Offscreen の `mode`**: `'hidden'` に設定されます。

### `mountWorkInProgressOffscreenFiber` と `updateWorkInProgressOffscreenFiber`

これらのヘルパー関数は、それぞれ `createFiberFromOffscreen` と `createWorkInProgress` を呼び出し、`Offscreen` Fiber の作成と更新を抽象化しています。

*   `mountWorkInProgressOffscreenFiber` (packages/react-reconciler/src/ReactFiberBeginWork.js:2088)
*   `updateWorkInProgressOffscreenFiber` (packages/react-reconciler/src/ReactFiberBeginWork.js:2097)

## まとめ

`Suspense` は、その子要素を `Offscreen` コンポーネントでラップし、`Offscreen` の `mode` プロパティを動的に切り替えることで、コンテンツの表示状態を管理しています。これにより、React はサスペンド時のフォールバック表示や、バックグラウンドでのコンテンツレンダリングといった高度な UI パターンを実現しています。
