---
name: feedback-useeffect-edge-detect-no-prefill
description: edge 検知 useEffect で「初回観測時の prefill」設計は polling race で誤動作する。per-key last-status ref + 真の遷移検知に倒す
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e3c21c1b-1628-4525-b1a7-9f52925e5111
---

`useEffect` で「pending/running → ok の遷移を検知して Toast 発火」型のロジックを書くとき、「初回 observe で既に ok ならページ再訪扱いで silent prefill」設計は polling race で誤動作する。

**Why:** polling 間隔 (2s) と LLM 完了タイミングが噛み合うと、polling 1 回目の observe 時点で entries に既に `ok` が含まれているシナリオが発生する。この時 prefill 設計だと該当 templateId を toastedSet に silent 登録 → 以降の edge 検知が永遠に空振り → 0 toast。PR #1090 L commit (462a8d297) で実機 e2e 0 件で再現確認、L2 (783d5f817) で per-template last-status ref 方式に置換して修正済。

**How to apply:**
- 状態遷移検知 useEffect では `Record<key, status>` 型の ref を使い、各 key の最後に観測した status を持つ
- **初回 observe** (`prev === undefined`) は status を記録するだけで何もしない (ページ再訪での silent 扱い)
- **真の遷移** (`prev !== "完了状態" && current === "完了状態"`) でのみ発火
- 一度 "完了" を観測した key は同サイクル内で再 "完了" 観測しても再発火しない (key ごとの冪等性)
- 「cycle 開始 (processing 遷移など) で全 key リセット」は不要。再要約や部分再生成も「前回完了 → 新 cycle pending/running 着信 → 完了」と通常遷移として再検知される
- 例: `frontend/components/new/detail/audio-detail-container.tsx` の `perTemplateStatusRef`

**やってはいけないこと:**
- 「初回 entries scan で `ok` を toastedSet に prefill」設計 (上記 race で発火しない)
- cycle reset block (`states === "processing"` で flag リセット) と組み合わせると挙動が複雑化、テスト困難
