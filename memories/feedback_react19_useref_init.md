---
name: React 19 で useRef は初期値必須
description: React 19 で useRef<T>() はエラー、useRef<T | undefined>(undefined) など初期値を与える
type: feedback
originSessionId: a9c063c5-235d-4e2c-b9f7-76301a259196
---
React 19 では `useRef<ReturnType<typeof setTimeout>>()` のような引数なし呼び出しが TS2554 (Expected 1 arguments, but got 0) エラーになる。React 18 では nullable 型なら引数省略可能だったが、19 で型シグネチャが「初期値必須」に厳格化された。

**Why:** React 19 の型定義で useRef のオーバーロードが整理され、`useRef<T>()` (引数なし) は `T extends void` のときのみ許容される形になった。

**How to apply:** React 18 → 19 の merge / migrate 時、`useRef<T>()` を grep で洗い出し、以下のいずれかに書き換える:
- `useRef<T | undefined>(undefined)` — 初期値なし用途
- `useRef<T | null>(null)` — DOM ref / 後で代入する用途
- `useRef<T>(initialValue)` — 初期値あり用途

実例: `frontend/components/schedules/schedule-search-bar.tsx:50` の `useRef<ReturnType<typeof setTimeout>>()` を `useRef<ReturnType<typeof setTimeout> | undefined>(undefined)` に修正して解消。
