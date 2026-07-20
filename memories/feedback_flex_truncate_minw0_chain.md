---
name: flex-truncate-minw0-chain
description: Tailwind で truncate/max-w が効かずテキストが見切れる時は、末端だけでなく親チェーン全段の min-w-0 欠落を疑う
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 26d1df96-c243-4624-91ba-ab6d76df7e7a
---

flex/grid 配下のテキスト省略 (truncate = overflow:hidden; text-overflow:ellipsis; white-space:nowrap) が効かず、長い文字列 (フォルダ名・URL等) が max-w-[80vw] 等の上限を超えて見切れる時の定石。

**Why:** flex item は既定で content size 未満に縮まない (min-width:auto)。末端要素に truncate を付けても、その親が min-w-0 でないと親自身が content 幅まで膨らみ、子の truncate 基準幅も膨らんで省略が無効化される。modern-web-guidance の css-layout ガイドの確定事項:「長い unbreakable content を含む flex item には min-inline-size:0 (min-width:0) を付ける」。

**How to apply:**
- 症状: max-w/w- 上限を付けたのにテキストが見切れる。truncate も付いているのに効かない。
- 対処: truncate する末端から、それを囲む flex/grid ラッパーを親方向に辿り、全段に min-w-0 を通す。一段でも欠けると破綻する。
- kozoka-v2r 実例 (sidebar): ScrollArea > .p-4 > div > .pl-3(アコーディオン) > ConversationLogMenu > .space-y-0.5(フォルダ一覧) > SidebarNavItem(truncate済)。末端 SidebarNavItem は元々 min-w-0+truncate だったが、中間4ラッパーに min-w-0 が無く見切れていた。4箇所に min-w-0 を通して解決。
- 幅上限は w-64 max-w-[80vw] のように「固定幅 + vw 上限」で min(256px, 80vw) を表現できる。上限の実効化には上記チェーンが前提。

関連: [[feedback_pnpm_filter_noop_exit0]] (検証は cd frontend / node_modules/.bin 直叩き、npx tsc は別パッケージを拾う罠)。
