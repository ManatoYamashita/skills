---
name: feedback-long-lived-branch-merge-semantic-loss
description: 長寿命ブランチへ dev を merge する際、Git の auto-merge がテキスト上は成功しても意味的に破壊するケースがある。git status の conflict 0 件を信用せず必ず go build / go vet / go test で意味検証する
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e1bfa904-dcd4-4b83-96b0-b106bc0a30e5
---

長寿命 feature ブランチ (PR #898 / feature/kosei-new-frontend は 100+ commit) に dev を merge した時、テキスト上のコンフリクトは 6 ファイルだけに見えても、auto-merge が「片方の差分だけ採用」して **依存関係を壊す** ケースがある。

具体例 (PR #898 で実遭遇):
- dev 側 #863 lint cleanup が `common.go` の `downloadSingle/downloadParallel/downloadAudioData` を「未使用」として削除
- 同時期 feature 側で spec 027 が `transcribeWithGemini` を新設し上記メソッドを呼ぶように
- `common.go` 自体には conflict marker が出ず auto-merge 成功 (dev 側削除が勝つ)
- 結果 `transcribe.go` (これは UU だった) を feature 側で採用しても `common.go` 側の依存元が消えているため `go build` で初めて「undefined: downloadAudioData」が判明
- 同様に `manual_guard_test.go` の `audioReport()` ヘルパが dev で削除され、`update_test.go` 等の参照だけ残って `go vet` 失敗

[[project-transcription-pipeline]] / [[project-branch-deployment-flow]] 関連。

**Why:** Git の merge は行ベースで、関数定義の削除と関数呼び出しの追加が別ファイルにあると衝突を検知しない。テキストの conflict 0 は意味の整合性を保証しない。

**How to apply:** 長寿命ブランチへ dev (や main) を merge した直後、`git status` がクリーンでも必ず:
1. `go build ./...` (backend)
2. `go vet ./...` (backend、未定義参照を検出)
3. `go test ./... -count=1 -short` (最低限)
4. frontend なら `pnpm typecheck` (devcontainer 側)
を走らせて意味検証する。エラーが出たら、`git log origin/dev ^origin/feature/... -- <該当ファイル>` で dev 側の lint cleanup 系 commit が原因か特定し、必要に応じて `git checkout HEAD -- <file>` で feature 側を取り戻す。

また atlas migration では別 PR が同じタイムスタンプ (`20260507000001` など) を採番していると `atlas migrate validate` がエラーを返す。merge 時にこちらも必ず走らせ、必要なら後発 PR 側を `+000002` などにリネームしてからコミットする。
