---
name: feedback-merge-resurrects-dead-code
description: 「片方のブランチが消した dead code をもう片方が後から再追加していた」場合、コンフリクト解消で feature 側を採用すると lint (unused) で発覚する。go build/vet だけでは検出不能で golangci-lint v2 が必要
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e1bfa904-dcd4-4b83-96b0-b106bc0a30e5
---

PR #898 の merge で実遭遇 (2026-05-13):

長寿命 feature ブランチへ dev を merge したとき、`backend/app/internal/service/audio_processing/transcribe.go` の `transcribeWithGemini` (197 行) と `common.go` の `downloadAudioData`/`downloadSingle`/`downloadParallel` (合計 200+ 行) が conflict した。

時系列:
1. merge-base には 4 関数とも存在し、互いに呼び合っていた
2. dev 側 #863 (lint cleanup) が「未使用」と判定して 4 関数とも削除
3. 同時期 feature 側で spec 027 が `transcribeWithGemini` を新規追加 (実は merge-base にあった旧版を復活させただけ) → 旧 downloadAudioData チェーンも自動的に「使用中」に
4. 私が「feature 側採用」でコンフリクト解消 → dead code が復活
5. go build / go vet / go test は全 PASS (関数同士が呼び合うので意味的には繋がる) → 通常検証では検出不能
6. CI で golangci-lint v2 の `unused` チェックが「リポジトリ全体で参照無し」と判定して fail

**Why:** Go の build/vet は「使われている」を**互いの呼び合い**で判断するので、孤立した呼び出しグラフ全体が dead code でも検出できない。golangci-lint の `unused` は「**外部から呼ばれているか**」を見るのでこれを検出する。

**How to apply:**
1. 長寿命ブランチ間 merge で「片方のブランチが lint cleanup で削除した関数群」のコンフリクトに遭遇したら、削除側を採用するか、復活させた後に必ず外部参照 (`grep -rn "<funcName>" --include='*.go' | grep -v "func.*<funcName>"`) を確認する
2. PR push 後 CI に golangci-lint v2 のような unused チェックがあれば、それを「最終的な dead code 判定」として信頼する
3. PR body に「バックエンド変更なし」と書かれていても、merge 経由で spec PR が後追い取り込まれて変更がある可能性があるので、必ず実態を確認する
4. このリポジトリの transcription pipeline は [transcription pipeline の実態](project_transcription_pipeline.md) の通り、`transcribeWithGemini` (audio_processing 版) は元々使われていない dead code。`transcribeWithSegmentation` だけが現役
