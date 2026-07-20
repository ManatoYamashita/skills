---
name: Next.js RSC 無限ループのデバッグ手順
description: GET /?_rsc=xxxx 連打時の切り分けフロー。Network Initiator を最初に見る、useCallback と useSearchParams.toString() で解消。
type: feedback
originSessionId: df59d2a9-a6a2-46d1-907e-46702eb59210
---
**ルール: Next.js App Router で `GET /?_rsc=xxxx` が無限連打されたら、以下の順で調査する**

## Step 1: ブラウザ DevTools Network タブの Initiator カラムを最初に見る

**Why:**
- サーバーログ (Turbopack) は `?_rsc=...` のクエリを省略して `GET / 200 in XXms` として表示するため、発火元が分からない
- Network タブの **Initiator カラムには `hookName.ts:行番号` まで出る** ので一次証拠として最も強い
- 2026-04 のインシデントで、AuthGuard の searchParams を疑って別修正を入れたが誤診。Initiator を見れば `useReportFilters.ts:44` が即座に判明した

**How to apply:**
- ユーザーが「dev server のログが止まらない」「メモリ不足ダイアログが出る」と言ったら、まず DevTools Network タブのスクリーンショット (Initiator カラム込み) を要求する
- サーバーログだけで原因を推定しない。Network が見られない場合のみサーバーログで仮説を立てる

## Step 2: 複数種類の `?_rsc=xxxx` が交互なら URL 状態間の往復ループ

**Why:**
- `_rsc` クエリは Next.js が URL ごとに一意なキーを割り当てるため、**異なるキーが交互に出る = 異なる URL を交互に訪れている**
- これは `router.push` / `router.replace` ループの典型。useEffect で router 呼び出し → URL 変化 → 再レンダー → 同じ useEffect 再実行 のパターン

**How to apply:**
- `router.push` / `router.replace` を grep し、useEffect 内か、useEffect deps に入る関数 (debounce setter 等) から呼ばれていないか確認

## Step 3: 真犯人は useCallback されていない hook 返却関数

**Why:**
- カスタム hook が返す関数を `useCallback` で包んでいないと、**呼び出し元の再レンダー毎に新しい参照**を返す
- 呼び出し元がそれを useEffect 依存配列に入れていると、毎レンダー useEffect 再実行で無限ループ
- 特に debounce パターン (`setTimeout(setter, 300)`) は、再実行ごとに新 timer を張って最後は setter を呼ぶので、URL 変更ループの引き金になりやすい

**How to apply:**
- hook を書く/レビューする時は「返却する関数は全て useCallback で包む」をデフォルトルール化
- 既存 hook をデバッグする時は、返却関数が useCallback か最初に確認

## Step 4: useSearchParams() を deps に入れるなら .toString() で文字列化

**Why:**
- Next.js 16 + React 19 の `useSearchParams()` の戻り値は RSC 再検証時や HMR 経由で参照が変わることがある
- 依存配列に直接入れると参照比較で再実行されてしまう
- `.toString()` で**値比較**にすれば、URL が変わらない限り安定

**How to apply:**
- `const searchParamsString = searchParams.toString()` をコンポーネント/hook の先頭で 1 度だけ計算
- useEffect / useMemo / useCallback の依存配列には文字列を使う
- `new URLSearchParams(searchParamsString)` で元のオブジェクト相当を再構築可能
