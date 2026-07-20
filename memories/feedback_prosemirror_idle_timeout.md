---
name: feedback_prosemirror_idle_timeout
description: ProseMirror/Tiptap に大量テキストを投入すると Claude in Chrome の idle待ちツールが45s timeoutする。軽量ツールで代替
metadata: 
  node_type: memory
  type: feedback
  originSessionId: fec90e65-3e04-4d1e-8e82-144fb3c87d0e
---

要約/文字起こしエディタ(ProseMirror/Tiptap の contenteditable)に数千〜8000バイトの本文を投入すると、Claude in Chrome の `screenshot` / `get_page_text` / `javascript_tool` が `executeScript waited 45000ms for document_idle` や `CDP sendCommand timed out` で失敗する。エディタの再描画でメインスレッドが飽和し document_idle 判定に到達できないのが原因。ページ自体は壊れていない(ネットワークリクエストは0件=ロード完了済み)。

**Why:** これらのツールは内部で document_idle を待つ。巨大 contenteditable の再レンダリングで idle に45秒以内に到達せずタイムアウトする。通常サイズの編集では一切起きない。

**How to apply:**
- 状態確認は idle を待たない軽量ツール(`read_console_messages` / `read_network_requests`)を優先する。実際、サイズ超過ガードの発火は console の `ApiError: Request body exceeds maxBodyBytes` で確実に取れた。
- トースト文言など DOM 値が要る時は、保存クリックも含めて1回の `javascript_tool` 内で `button.click()` → トースト DOM をポーリング取得する(idle 待ちを最小化。ただし重い時はこれも timeout する)。
- 大量テキストは一気に入れず小分けに。検証ボリュームは閾値ギリギリ(8000バイト超を少しだけ)に留め、エディタ負荷を抑える。
- 重くなったタブは閉じて新タブで軽く再現する。リロードしても未保存の大量テキストは送信前ガードで弾かれサーバ未保存なので消える。

関連: [[feedback_batch_small_chunks]](こちらは別系統=長大バッチで生成側のツールコールが malformed になる問題), pr-review-e2e スキル Phase 3。
