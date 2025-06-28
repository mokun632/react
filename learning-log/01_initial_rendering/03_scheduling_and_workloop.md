# スケジューリングとレンダリングループの分析

## 0. 更新の起点: `updateContainer` (`ReactFiberReconciler.js`)

`ReactDOM.createRoot().render()` などから呼び出され、React コンポーネントツリーの更新を開始する主要なエントリーポイントの一つ。

**役割:**
*   新しい要素ツリー (`element`) を受け取る。
*   適切な更新優先度 (`Lane`) を決定する (`requestUpdateLane`)。
*   Update オブジェクトを作成し、更新内容 (`element`) とコールバックを格納する (`createUpdate`)。
*   Update オブジェクトをルート Fiber (`container.current`) の更新キューに追加する (`enqueueUpdate`)。
*   最終的に `scheduleUpdateOnFiber` を呼び出し、更新プロセス (Render/Commit フェーズ) を開始させる。

**主要な内部関数:** `updateContainerImpl`

```javascript
// packages/react-reconciler/src/ReactFiberReconciler.js

export function updateContainer(
  element: ReactNodeList, // 更新後のReact要素ツリー
  container: OpaqueRoot, // FiberRootオブジェクト
  parentComponent: ?React$Component<any, any>, // 親コンポーネント (レガシーContext用)
  callback: ?Function, // 更新完了後のコールバック関数
): Lane {
  const current = container.current;
  const lane = requestUpdateLane(current); // 1. Lane (優先度) を決定
  updateContainerImpl( // 2. 内部実装を呼び出し
    current,
    lane,
    element,
    container,
    parentComponent,
    callback,
  );
  return lane;
}

function updateContainerImpl(
  rootFiber: Fiber, // HostRoot Fiber
  lane: Lane,
  element: ReactNodeList,
  container: OpaqueRoot,
  parentComponent: ?React$Component<any, any>,
  callback: ?Function,
): void {
  // ... Context処理 ...

  // 3. Updateオブジェクト作成
  const update = createUpdate(lane);
  update.payload = {element};
  if (callback !== null) {
    update.callback = callback;
  }

  // 4. Updateをキューに追加 & スケジューリング準備
  const root = enqueueUpdate(rootFiber, update, lane); // ★ enqueueUpdateを呼び出し

  // 5. スケジューリング開始
  if (root !== null) { // enqueueUpdateがルートを返した場合
    startUpdateTimerByLane(lane); // Profiling用
    scheduleUpdateOnFiber(root, rootFiber, lane); // ★ WorkLoopスケジューリングへ
    entangleTransitions(root, rootFiber, lane); // Transition関連
  }
}
```

---

## 1. `updateContainerImpl` 内の詳細: `createUpdate` と `enqueueUpdate`

`updateContainerImpl` の中で、Update オブジェクトの作成とキューへの追加、そしてスケジューリングの準備が行われます。

### 1.1. `createUpdate(lane)` (`ReactFiberClassUpdateQueue.js`)

*   **役割:** 指定された `lane` を持つ新しい Update オブジェクトを作成します。
*   **実装:** 非常にシンプルな関数で、Update オブジェクトに必要なプロパティ（`lane`, `tag: UpdateState`, `payload: null`, `callback: null`, `next: null`）を初期化して返します。
*   **`updateContainerImpl` での使われ方:**
    *   `const update = createUpdate(lane);` でオブジェクトが作成されます。
    *   `update.payload = {element};` で、`render()` に渡された新しい React 要素ツリーがペイロードとして設定されます。
    *   オプションの `callback` があれば、`update.callback` に設定されます。

### 1.2. `enqueueUpdate(rootFiber, update, lane)` (`ReactFiberClassUpdateQueue.js`)

*   **役割:** 作成された `update` オブジェクトを、対象となる `fiber` (ここでは `rootFiber`) の更新キュー (`updateQueue`) に追加し、最終的にスケジューリングの起点となるルート (`FiberRoot`) を返します。
*   **処理フロー:**
    1.  **Update をキューに追加 (共通処理):** まず、`update` オブジェクトを `updateQueue.shared.pending` が指す循環リストの末尾に追加します。この処理は、後続の条件分岐に関わらず **常に** 行われます。（具体的なキュー追加のロジックは `finishQueueingConcurrentUpdates` でバッチ処理される場合と、Unsafe な更新で直接行われる場合があるため、図解は該当箇所で後述）。
    2.  **条件分岐 (`isUnsafeClassRenderPhaseUpdate`):** 次に、この更新が Unsafe なレンダーフェーズ更新かどうかで処理を分岐します。
        *   **`true` (Unsafe Render Phase Update):** 古いクラスコンポーネントのレンダーフェーズ更新の場合。互換性のために、`unsafe_markUpdateLaneFromFiberToRoot` を呼び出してレーンをルートまで伝播させ、ルートを返します。Concurrent Mode の標準的なバッチングやスケジューリングはバイパスされます。（この経路では `enqueueConcurrentClassUpdate` は呼ばれません）。
        *   **`false` (Concurrent Update):** 通常の更新（レンダーフェーズ外、関数コンポーネント、Concurrent Mode のクラスコンポーネントなど）の場合。`enqueueConcurrentClassUpdate` (`ReactFiberConcurrentUpdates.js`) を呼び出します。

*   **`enqueueConcurrentClassUpdate` の役割 (`ReactFiberConcurrentUpdates.js`):**
    *   この関数は、`enqueueUpdate` の `false` (Concurrent Update) 経路から呼び出されます。
    *   **主な役割:** Concurrent Mode のスケジューリングに関連する処理（特に更新のバッチングとレーン伝播の準備）を行います。
    *   **処理内容:**
        1.  型キャストを行い、内部の `enqueueUpdate` (同ファイル内 L61) を呼び出します。
        2.  内部 `enqueueUpdate` は、更新情報 (`fiber`, `queue`, `update`, `lane`) を **`concurrentQueues` 配列に一時的に格納（バッチング）** します。**注意:** この内部 `enqueueUpdate` は `sharedQueue.pending` を直接変更しません。
        3.  最適化のために `fiber.lanes` と `alternate.lanes` は **即座に更新** されます。
        4.  `enqueueConcurrentClassUpdate` は最終的に `getRootForUpdatedFiber` を呼び出してルートを返します。
    *   **バッチングの効果:** この `concurrentQueues` への一時格納により、同じイベントループ内で発生した複数の更新に対するレーン伝播処理が `finishQueueingConcurrentUpdates` でまとめて行われ、効率化と一貫性（Automatic Batching）が実現されます。`finishQueueingConcurrentUpdates` 内で、`concurrentQueues` の情報が処理され、最終的に `markUpdateLaneFromFiberToRoot` が呼び出されます。

*   **戻り値と次のステップ:** `enqueueUpdate` (`ReactFiberClassUpdateQueue.js`) が `null` でないルート (`FiberRoot`) を返した場合、`updateContainerImpl` は次に `scheduleUpdateOnFiber` を呼び出すことになります（次回の学習範囲）。

---

## 2. `scheduleUpdateOnFiber` からレンダリングループ開始までの流れ

`scheduleUpdateOnFiber` (`ReactFiberWorkLoop.js`) は、更新をスケジュールし、必要に応じて既存のレンダリングを中断します。そして、最終的にSchedulerを介して実際のレンダリング作業を開始します。

### 2.1. `scheduleUpdateOnFiber` (`ReactFiberWorkLoop.js`)

*   **役割**: 更新をスケジュールし、必要に応じて既存のレンダリングを中断します。
*   **処理内容**:
    *   レンダーフェーズ中の更新や、既にサスペンドしているルートへの更新の場合、既存のスタックをリセットし、ルートをサスペンド状態としてマークします。
    *   `markRootUpdated(root, lane)` を呼び出し、ルートに保留中の更新があることをマークします。
    *   `ensureRootIsScheduled(root)` を呼び出し、ルートがスケジュールされていることを確認し、Schedulerにタスクを登録します。
    *   同期レーン（`SyncLane`）の場合、`flushSyncWorkOnLegacyRootsOnly()` を呼び出して同期作業を即座にフラッシュする場合があります。

### 2.2. `ensureRootIsScheduled` (`ReactFiberRootScheduler.js`)

*   **役割**: ルートをスケジュールに追加し、ルートスケジュールを処理するためのマイクロタスクが保留中であることを保証します。
*   **処理内容**:
    *   `firstScheduledRoot` と `lastScheduledRoot` のリンクリストに `root` を追加します。
    *   `scheduleImmediateRootScheduleTask()` を呼び出し、ルートスケジュールを処理するためのマイクロタスクをスケジュールします。

### 2.3. `scheduleImmediateRootScheduleTask` (`ReactFiberRootScheduler.js`)

*   **役割**: ルートスケジュールを処理するための即時タスク（マイクロタスクまたはSchedulerタスク）をスケジュールします。
*   **処理内容**:
    *   `processRootScheduleInMicrotask()` を呼び出します。`act` スコープ内では `act` キューに、それ以外では `scheduleMicrotask` または `Scheduler_scheduleCallback` を使用してマイクロタスクをスケジュールします。

### 2.4. `processRootScheduleInMicrotask` (`ReactFiberRootScheduler.js`)

*   **役割**: マイクロタスク内でルートスケジュールを処理します。
*   **処理内容**:
    *   `firstScheduledRoot` から始まるリンクリストを反復処理します。
    *   各ルートに対して `scheduleTaskForRootDuringMicrotask()` を呼び出し、適切な優先度でタスクがスケジュールされていることを確認します。
    *   最後に、保留中の同期作業があれば `flushSyncWorkAcrossRoots_impl()` を呼び出してフラッシュします。

### 2.5. `scheduleTaskForRootDuringMicrotask` (`ReactFiberRootScheduler.js`)

*   **役割**: 特定のルートに対して、Schedulerにタスクをスケジュールします。
*   **処理内容**:
    *   `markStarvedLanesAsExpired(root, currentTime)` を呼び出し、飢餓状態のレーンを期限切れとしてマークします。
    *   `getNextLanes` を呼び出して、作業する次のレーンとその優先度を決定します。
    *   作業するレーンがない場合や、ルートがサスペンドしている場合は、既存のコールバックをキャンセルし、`NoLane` を返します。
    *   同期レーンが含まれる場合、Schedulerにタスクをスケジュールせず、同期作業はマイクロタスクの最後にフラッシュされることを期待します。
    *   それ以外の場合、`scheduleCallback` を呼び出して、Schedulerに `performWorkOnRootViaSchedulerTask` をスケジュールします。優先度は `lanesToEventPriority(nextLanes)` から決定されます。

### 2.6. `performWorkOnRootViaSchedulerTask` (`ReactFiberRootScheduler.js`)

*   **役割**: Schedulerによってスケジュールされたタスクのエントリーポイントです。
*   **処理内容**:
    *   `flushPendingEffects(true)` を呼び出して、保留中のパッシブエフェクトをフラッシュします。
    *   `performWorkOnRoot(root, lanes, forceSync)` を呼び出し、実際のレンダリング作業ループ（Renderフェーズ）を開始します。

### 2.7. `performWorkOnRoot` (`ReactFiberWorkLoop.js`)

*   **役割**: 実際のレンダリング作業ループを開始します。
*   **処理内容**:
    *   `renderRootConcurrent` または `renderRootSync` を呼び出し、Fiberツリーの構築と差分検出を行います。
    *   `workLoopSync` または `workLoopConcurrent` を呼び出し、`performUnitOfWork` を介して各Fiberノードの作業を実行します。

---

## 3. 更新スケジューリングからレンダリングループ開始までのフロー (Mermaid)

```mermaid
graph TD
    A[updateContainer] --> B{updateContainerImpl};
    B --> C[createUpdate];
    B --> D[enqueueUpdate];
    D --> E[scheduleUpdateOnFiber];
    E --> F[markRootUpdated];
    E --> G[ensureRootIsScheduled];
    G --> H[scheduleImmediateRootScheduleTask];
    H --> I[processRootScheduleInMicrotask];
    I --> J[scheduleTaskForRootDuringMicrotask];
    J --> K{Scheduler_scheduleCallback};
    K --> L[performWorkOnRootViaSchedulerTask];
    L --> M[flushPendingEffects];
    L --> N[performWorkOnRoot];
    N --> O{renderRootConcurrent / renderRootSync};
    O --> P{workLoopSync / workLoopConcurrent};
    P --> Q[performUnitOfWork];
    Q --> R[beginWork];
    Q --> S[completeUnitOfWork];
    S --> T[completeWork];

    subgraph ファイルパス
        A --- ReactFiberReconciler.js
        E --- ReactFiberWorkLoop.js
        G --- ReactFiberRootScheduler.js
        N --- ReactFiberWorkLoop.js
        R --- ReactFiberBeginWork.js
        T --- ReactFiberCompleteWork.js
    end
