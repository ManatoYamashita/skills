---
name: feedback-turbopack-stale-chunk
description: Turbopack で SpeakerBar 等の props 追加変更がブラウザに反映されない時のキャッシュパージ手順
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fe83beb0-38b7-4df6-8100-ae5e69c5a01e
---

Next.js 16 + Turbopack の dev server で「ローカルファイルは編集済み、tsc/vitest は緑、なのにブラウザの React Fiber を覗くと古い関数本体がロードされている」現象が起きる。HMR でも full reload でも `Cmd+Shift+R` でも治らない場合がある。

**Why:** Turbopack は `.next/` 配下にビルドキャッシュを持ち、稀にファイル変更検知をこぼす。特に「component の props シグネチャ変更 + 上位 chain (Container → View → Section → Component) 全てに改修が走る」ケースで、依存グラフの更新が一部だけスキップされることがある。実害: ブラウザに古い chunk が配信され、新 props が attach されず handler が wire されていないように見える。

**How to apply:**
1. ブラウザで `Object.keys(el).find(k => k.startsWith('__reactProps'))` の結果が期待した props を欠いている、かつ `__reactFiber` から取った `fiber.type.toString()` に期待文字列が無い → Turbopack stale chunk 確定。
2. 解決: `Ctrl+C` で pnpm dev 停止 → `cd /workspace/frontend && rm -rf .next` → `pnpm dev` → ブラウザを Cmd+Shift+R で強制リロード。
3. **Ghostty 側ではなく devcontainer 側で実行** ([[feedback-dev-server-ownership]] と整合)。
4. CDP の `Runtime.evaluate` は Promise + setTimeout で renderer freeze を引き起こしやすい。`javascript_tool` で確認するときは**同期式の値返し**に徹し、Promise/setTimeout は避ける。
