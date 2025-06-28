# React コア概念に関するルール

このドキュメントは、React のコア概念 (Fiber, Reconciliation, Hooks, Context など) をコードレベルで理解・分析する際のガイドラインを定めます。

## 1. Fiber

Fiber は React のコアとなる Reconciliation アルゴリズムの実装単位です。

*   **目的:** Fiber ノードの構造 (type, key, stateNode, child, sibling, return など) とその役割を理解する。
*   **分析時の注目点:**
    *   Fiber ノードがどのように生成、更新、削除されるか (`ReactFiber.js`, `ReactChildFiber.js` など)。
    *   Work-in-progress ツリー (`alternate` プロパティ) と Current ツリーの関係性。
    *   更新処理 (state の変更、props の変更など) がどのように Fiber ノードに影響を与えるか。
    *   `lanes` や優先度 (`ReactFiberLane.js`) がスケジューリングにどう関わるか。
*   **Cline への指示:**
    *   特定の Fiber ノードのプロパティについて質問された場合、その意味と関連コード箇所を説明してください。
    *   Fiber ツリーの構造や特定のノード間の関係性を可視化するために Mermaid 図を使用してください。
    *   「このコンポーネントに対応する Fiber ノードは？」といった質問に対して、関連する Fiber ノードを特定してください。

## 2. Reconciliation (差分検出と更新適用)

Reconciliation は、仮想 DOM (Fiber ツリー) と実際の DOM (または他のターゲット環境) の差分を効率的に検出し、最小限の操作で更新を適用するプロセスです。

*   **目的:** Reconciliation プロセス (Render フェーズと Commit フェーズ) の流れと、各フェーズで行われる処理を理解する。
*   **分析時の注目点:**
    *   **Render フェーズ:**
        *   `beginWork` (`ReactFiberBeginWork.js`): コンポーネントのレンダリング、差分検出の開始。
        *   `completeWork` (`ReactFiberCompleteWork.js`): 子要素の処理完了後、DOM ノードの生成や更新内容 (副作用) の準備。
        *   どのようにして更新が必要な Fiber ノードが特定されるか。
    *   **Commit フェーズ:**
        *   `commitMutationEffects` (`ReactFiberCommitWork.js`): DOM の挿入、更新、削除などの実際の変更。
        *   `commitLayoutEffects` (`ReactFiberCommitWork.js`): `useLayoutEffect` の実行、DOM 計測など。
        *   `commitPassiveEffects` (`ReactFiberCommitWork.js`): `useEffect` の実行。
    *   キー (`key` prop) が Reconciliation にどのように影響するか。
*   **Cline への指示:**
    *   特定の更新シナリオ (例: state 更新、リストアイテムの追加/削除) における Reconciliation のステップを説明してください。
    *   Render フェーズと Commit フェーズの役割分担について説明してください。
    *   `beginWork` や `completeWork` 内の特定の処理について質問された場合、その目的と影響を解説してください。

---
*(今後、Hooks, Context, Scheduler 連携などの概念に関するルールを追記予定)*
