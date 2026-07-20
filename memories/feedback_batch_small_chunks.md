---
name: feedback_batch_small_chunks
description: ツールコールが malformed で出力停止する時の暫定回避 — batch を小さく刻み 1ターン1ツールに寄せる
metadata: 
  node_type: memory
  type: feedback
  originSessionId: caa3afb4-89d8-4846-81bc-4ed9ee9d387a
---

ツールコール(特に `browser_batch` の `actions` 配列や複数行 `Bash`)を長くネストして発行すると、モデル側のツールコール生成が間欠的に整形不正(malformed / parse 不能)を起こし、出力が止まってユーザーが `continue` で再開させる事象が頻発する。外部障害ではなく生成側の揺らぎが真因。

回避策(予防):
- batch のアクション数を欲張らず小さく刻む(整形不正の確率が下がる)
- 1ターン1ツールに寄せる場面を増やす
- type のような長い文字列を含むアクションは特に malformed を誘発しやすい。長文字列×多数を1バッチに詰めない

回避策(復旧 = 一度 malformed が出たら):
- 即座に単発の最小ツール呼び出しに退避する。同じ巨大バッチを再送しない。今回 PR #1289 E2E で巨大 browser_batch を再送し続け malformed を多発させた。単発呼び出しに戻した瞬間から成功に転じた。
- malformed が2回続いたら、その操作は分割して逐次実行に切り替える。

**Why:** 長大なネスト構造ほど malformed の確率が上がり、その都度 `continue` 連打で数秒〜のロスとユーザー手間が発生するため。
**How to apply:** browser_batch は 3〜4 アクション程度に抑える。長い Bash 複数行連結も避け、必要なら分割して逐次実行する。

注: これはハーネス側のツールコール整形が改善されれば不要になり得る**暫定対処**。将来 malformed 停止が起きなくなったらこのメモリは見直し/削除候補。

関連: [[feedback_devcontainer_multiline_break]](複数行コマンドが devcontainer ターミナルで壊れる別系統の問題), pr-review-e2e スキル Phase 3 の E2E 運用。
