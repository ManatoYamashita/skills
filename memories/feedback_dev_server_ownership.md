---
name: pnpm dev は Cursor + devcontainer 側で起動する
description: Ghostty (Claude Code) で pnpm dev を起動しない。port 3000 衝突と Docker バックエンド不在で機能しないため。
type: feedback
originSessionId: df59d2a9-a6a2-46d1-907e-46702eb59210
---
**ルール: Ghostty (Claude Code) 側で `pnpm dev` / `next dev` を起動しない。フロントエンド dev server は Cursor + devcontainer 側で起動する。**

**Why:**
- Ghostty 側で pnpm dev を起動すると **Docker 上のバックエンド API が見えず機能しない**（devcontainer 内のネットワークに繋がらない）
- Cursor の `anysphere.remote-containers` 拡張が `localhost:3000` を forwarder として常時 LISTEN している + Docker compose が `0.0.0.0:3000->3000/tcp` を publish している → **Ghostty から pnpm dev を起動するとポート衝突で EADDRINUSE、または HMR が混線してログループ + メモリ膨張 → macOS OOM ダイアログ**
- 2026-04 に実際にこの組み合わせで「アプリケーションメモリが不足しています」ダイアログが発火し、Ghostty が 65〜95 GB 表記になるインシデント発生

**How to apply:**
- Claude Code (Ghostty) の責務は **lint / build / test / コード調査・修正 / git 操作** に限定する (CLAUDE.md の役割分担と一致)
- ユーザーから「開発サーバを起動して」「dev server 立ち上げて」と言われても、**Ghostty 側では起動しない**。「Cursor + devcontainer で起動してください」と返す
- E2E テストで dev server が必要な場合は、**Cursor 側で既に起動済みの dev server に接続する**（自分で起動しない）
- 例外: ユーザーが明示的に「Ghostty で別ポートで起動して」と指示した場合のみ、`pnpm dev -- --port 3100` 等で別ポート起動を検討。ただし Docker バックエンドに繋がらないので UI 検証の範囲は限定的
- Ghostty で何らかの dev server 系プロセスを見かけたら、残骸の可能性が高いので `lsof -i :3000` で占有元を確認してから判断

**絶対ルール: `pnpm install` は devcontainer 側でのみ実行する。Ghostty (Claude Code) では絶対に走らせない。**

**Why:**
- macOS 上で `node_modules` を Ghostty (darwin-arm64) と devcontainer (linux-arm64) の双方から bind mount で共有しているため、`pnpm install` で書き込まれる native binding (rolldown / lightningcss / sharp 等) はどちらか一方のプラットフォーム用しか同居できない
- Ghostty で `pnpm install` を走らせると `node_modules/.pnpm/` 配下が darwin-arm64 binding で上書きされ、devcontainer 側の `pnpm dev` が `lightningcss.linux-arm64-gnu.node` を見つけられず **Build Error / ERR_EMPTY_RESPONSE** で停止する
- 2026-04-30 に実際このシナリオで dev server を 2 回ダウンさせている。復旧には devcontainer 側で再 install + dev server 再起動が必要で、ユーザーへ手間を強いる

**How to apply:**
- Ghostty で `pnpm install` / `pnpm add` / `pnpm remove` / `pnpm update` 等の **lockfile / node_modules を書き換えるコマンドは一切実行しない**
- 必要が生じたら必ずユーザーに「devcontainer 側で `pnpm install` をお願いします」と依頼する
- vitest / tsc 実行時に `Cannot find module '@rolldown/binding-darwin-arm64'` 等の native binding エラーが出ても、**Ghostty で install して直そうとしない**。devcontainer 側で install 済みなら darwin binding は無いのが正常状態。テストは devcontainer 側で走らせるか、ユーザーに依頼する
- git worktree 配下で `tsc` / `lint` を走らせたい場合の安全な迂回路は [[worktree-node-modules-symlink]] 参照 (メイン repo の node_modules を symlink で借りるだけで `pnpm install` は不要)
- **devcontainer に test 実行を依頼する際は、対象ブランチへの `git checkout` 手順を必ずセットで伝える**。Ghostty と devcontainer は同一 worktree (`/Users/m-yamashita/Desktop/dev/x/kozoka-v2r` = `/workspace`) を共有するため、Ghostty 側が別ブランチに居る間 devcontainer 側も同じブランチ。テストファイルが対象ブランチにしか無い場合 vitest が `No test files found` で失敗する (2026-05-12 PR #972 で発生)
- 例外なし。「ちょっとだけ」「すぐ戻す」も禁止。最後に install したプラットフォーム側しか動かないため、片方を壊す行為そのもの

**検出指標 (devcontainer 側 dev server が壊れたサイン):**
- Claude in Chrome の navigate で `ERR_EMPTY_RESPONSE` / `body: "This page isn't working"`
- Next.js Build Error 画面で `Cannot find module '../lightningcss.linux-arm64-gnu.node'` 等の linux binding 不在エラー
- これらが出たら、わたくしが直前に Ghostty で `pnpm install` を走らせてしまった可能性が高い。即座にユーザーに devcontainer 側 install を依頼する

**派生ルール: Ghostty 側で `rm -rf .next` (Next.js cache 削除) も dev server 起動中に実行禁止**
- 2026-04-30 観測: Turbopack が globals.css の `@custom-variant` 変更を hot reload で反映しないとき、 `rm -rf .next` でキャッシュ強制リセットを Ghostty 側から実施した結果、 dev server プロセスは生きていたが in-memory state と `.next` ファイル間の不整合で `Internal Server Error` を返し続けた → ユーザーに dev server 再起動を再依頼する手間
- 教訓: Turbopack の永続キャッシュを更新したい場合、 必ず先に dev server を停止 (Ctrl+C) してから `rm -rf .next` → `pnpm dev` 起動を依頼する
- Ghostty 側でできる代替策: `globals.css` を `touch` で再保存しても効かないとき、それ以上は dev server 操作領域なので諦めてユーザーに依頼
