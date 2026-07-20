---
name: ハイブリッド route 構造の選好
description: origin/main の (app) route group をそのまま受け入れず、フラット構造 + 機能側 layout のハイブリッドを好む
type: feedback
originSessionId: a9c063c5-235d-4e2c-b9f7-76301a259196
---
origin/main で導入された Next.js `(app)` route group をそのまま受け入れる代わりに、HEAD のフラット `app/` 構造を維持し、AppShell + business-plan gate を機能側の `layout.tsx` に分離するハイブリッド構造を選好する。

**Why:** ユーザーは「`feature/kosei-new-settings-layout` の方が美しい」と表現。`(app)/layout.tsx` での集約は (a) gate 対象パスを文字列ハードコードするためグローバルリストの保守が必要 (b) `'use client'` を強制し配下 RSC の旨味を消す (c) root の AuthGuard と二重ガード になる、という設計上の弱点があるため。

**How to apply:** Next.js App Router で route group が「単に共通 layout を被せたい」だけの理由で導入される場面では、機能側の layout.tsx に分離する案を提示する。`(app)` のような全体ラップ用 group は、本当に AppShell の重複を排除できる場合のみ採用を提案。実例: `app/page.tsx` を AppShell でラップ + `app/reports/layout.tsx` で AppShell ラッパー + `app/daily-reports/layout.tsx` `app/schedules/layout.tsx` で AppShell + gate という形が美しい。
