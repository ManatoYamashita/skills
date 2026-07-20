---
name: feedback-pnpm-filter-noop-exit0
description: "kozoka-v2r は pnpm-workspace.yaml 無しの単一プロジェクト構成。`pnpm --filter frontend` は「No projects matched」と言って exit 0 で素通りする。filter は使わず cd frontend && pnpm 直叩き"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e1bfa904-dcd4-4b83-96b0-b106bc0a30e5
---

このリポジトリは `pnpm-workspace.yaml` を持たない **単一プロジェクト構成** (frontend/package.json の name は `kozoka-v2r-frontend`)。Vercel next-forge 系のモノレポと違って `pnpm --filter` は機能しない。

`pnpm --filter frontend typecheck` のような呼び方をすると stderr に "No projects matched the filters in <path>" と出力しつつ exit 0 で素通りする。`set -e` を信用して作った検証スクリプトはこれを「OK」として誤判定する。

**Why:** pnpm の filter は workspace 前提。単一プロジェクトでは filter にマッチしようがないが、エラーではなく no-op として exit 0 を返す仕様。

**How to apply:**
1. このリポジトリで frontend スクリプトを走らせるときは `cd frontend && pnpm <script>` 直叩きにする (`--filter frontend` は使わない)
2. 検証スクリプトで pnpm を叩くなら、stdout/stderr をキャプチャして "No projects matched|No project matched" を grep で検出して fail させる (`tmp/pr898-verify.sh` の `run_and_check` ヘルパが実装例)
3. frontend に typecheck スクリプトは無い → `pnpm exec tsc --noEmit` で直接走らせる
4. backend は `make build` (本番 Lambda コンテナ) と `make build-dev` (= go build) の使い分けに注意。devcontainer 検証は `make build-dev`
