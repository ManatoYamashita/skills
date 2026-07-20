---
name: feedback-pnpm-test-filter
description: pnpm test -- <pattern> は filter として渡らず全件実行される。pattern は -- なしで直接渡す
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 92c54c68-9ae9-4749-a0be-756aa668949d
---

`pnpm test -- topics-section` のような呼び方では、`--` 以降の引数は vitest の **filter pattern として認識されず**、全 98 ファイル (5 分弱) を走らせてしまう。

**Why:** package.json の test スクリプトは `vitest run -- topics-section` のように `--` 込みで vitest に渡すが、vitest 側は `--` を「flag 終端」と解釈するため、後続の `topics-section` は filter として効かず無視される。結果、全件実行になる。

**How to apply:**
- 単体ファイルを狙い撃ちしたい時は `pnpm test topics-section` (npm script に直接 pattern を渡す) または `pnpm exec vitest run topics-section` を使う
- `--reporter=verbose` のような **vitest 自身のフラグ** を渡すときだけ `pnpm test -- --reporter=verbose` の `--` 形式が必要
- 全件実行で 5 分かかるので、PR レビュー時の反復確認では特に注意

関連: [[feedback-pnpm-filter-noop-exit0]] (kozoka-v2r は単一プロジェクトで `pnpm --filter frontend` も空振りする件)
