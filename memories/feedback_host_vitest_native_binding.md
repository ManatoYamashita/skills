---
name: feedback-host-vitest-native-binding
description: devcontainer 由来の Linux arm64 node_modules を Mac host で vitest 完走させる手順。pnpm install を叩かず darwin-arm64 binding を npm pack で個別配置
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e3c21c1b-1628-4525-b1a7-9f52925e5111
---

Mac host (Ghostty) で `pnpm exec vitest run` を走らせると、`rolldown@1.0.0-rc.16` / `lightningcss@1.32.0` / `@tailwindcss/oxide@4.2.3` の native binding が `darwin-arm64` で見つからず即落ちする。devcontainer で生成された `frontend/node_modules` は Linux arm64 用 binding しか持たないため。

**Why:** 単純な解決策である `pnpm install` を host で叩くと、[[feedback-dev-server-ownership]] の通り devcontainer 側の dev server を Build Error で壊す。だが個別の binding を `npm pack` で取得して `.pnpm/<name>@<ver>/node_modules/<name>/` に手動配置するだけなら、devcontainer 側 node_modules を一切触らずに host vitest を完走できる (128 ファイル / 1436 ケース緑、2分43秒)。

**How to apply:** host で vitest を回す必要が出たら以下の手順:

1. 独立 worktree を別ブランチ名で派生 (本ブランチ占有制約のため `git worktree add -b <tmp> <path> <main-branch>` 形式)
2. `rsync -a` で main repo の `frontend/node_modules` を worktree にコピー (symlink を保ったまま、約 1 分で 1.4GB)
3. 3 種類の darwin-arm64 binding を取得して展開:
   - `@rolldown/binding-darwin-arm64@<ver>` (vitest 起動に必要)
   - `lightningcss-darwin-arm64@<ver>` (PostCSS 経由)
   - `@tailwindcss/oxide-darwin-arm64@<ver>` (Tailwind CSS 経由)
4. 各々を `.pnpm/<name>@<ver>/node_modules/<name>/` に展開 (`tar -xzf <tarball> --strip-components=1`)
5. 親パッケージの peer ディレクトリ `.pnpm/<parent>@<ver>/node_modules/` に **symlink** を貼って resolution を成立させる (例: `.pnpm/rolldown@<ver>/node_modules/@rolldown/binding-darwin-arm64`)
6. lightningcss のみ fallback 用に `.node` ファイルを `.pnpm/lightningcss@<ver>/node_modules/lightningcss/` 直下にもコピー (本体ソースが `require('../lightningcss.<arch>.node')` を fallback で呼ぶため)
7. vitest 完走後、worktree を `git worktree remove --force` + 派生ブランチを `git branch -D` + 一時 tarball 削除で完全撤去

**禁忌:** host で `pnpm install` / `pnpm add` を叩く ([[feedback-dev-server-ownership]] 違反)、main repo の `frontend/node_modules` に直接配置する、`pnpm-lock.yaml` を更新する。

**バージョン取得方法:** `cat node_modules/.pnpm/<name>@<ver>/node_modules/<name>/package.json` の `version` を確認。`optionalDependencies` を見ると全プラットフォーム binding 一覧が取れる。バージョンが変わったら手動で取り直す。

CI / Vercel preview は Linux 環境なので影響なし、本手順は **個人 host での開発高速化のみ**。
