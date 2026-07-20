---
name: feedback-usereportsentiments-polling-conflict
description: useReportSentiments を audio-detail-container で呼ぶと useReportProgress の polling が 1 回で止まる回帰が起きる
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e3c21c1b-1628-4525-b1a7-9f52925e5111
---

`useReportSentiments` を `audio-detail-container.tsx` 内で直接呼び出すと、`useReportProgress` の polling が `/progress` 1 回で停止する回帰が発生する。`SentimentSection` 内 (`react-query` の queryKey 共有) で呼ぶ分には問題なし。

**Why:** L commit 検証時の e2e で再現確認済。`useReportSentiments` の `enabled = status === "reqCompleted"` 制御が container 階層で評価されると、useEffect 連鎖 + status 遷移時の `enabled` flip が `useReportProgress` の React Query refetchInterval scheduler を妨害する仮説 (Detail は未特定だが現象は確実)。L-FIX commit で `useReportSentiments` を container から外したら polling が即座に回復 (1 → 10 回継続)。

**How to apply:**
- 分析系データ (sentiment/speakers/overview の取得 hook) を `audio-detail-container.tsx` 直接呼び出しで参照したくなったら回避する
- 必要なら子セクション (`SentimentSection` 等) 内で local effect で発火させて、`onCompleted` callback / useRef bus で親に通知する設計に倒す
- 関連: L-5 (全分析完了 Toast) の実装はこの制約に従って **SentimentSection 内に Toast 発火を移す** べき
- 同様の制約は `useReportSpeakers` にもあり得るので、新規 hook を container に追加する前に必ず polling 動作の e2e 確認をする

実例: PR #1090 L commit (462a8d297) → e2e で polling 停止発覚 → L-FIX commit (a0f019870) で即座に回復。

**L2 で確定した代替設計** (commit 8f618153c): 分析系完了 Toast は `useReportSentiments` を呼ばずに `report.states` の `processing-like → reqCompleted` 遷移検知だけで発火可能。backend orchestrator が Stage 3 並列全完了後に reqCompleted を書く設計のため、states 遷移 = 全分析系完了の確定信号として扱える。新規 hook 呼びゼロで polling 阻害ゼロ。
