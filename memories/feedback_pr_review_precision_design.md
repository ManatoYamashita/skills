---
name: feedback_pr_review_precision_design
description: PRレビュー精度の核は並列化でなくdiffmap前処理と反証専任の批判検証。pr-review-e2eスキル設計の教訓
metadata: 
  node_type: memory
  type: feedback
  originSessionId: b131bb29-cae4-4654-9709-19fa584ac444
---

PRレビュー精度を上げる際、「観点別 subagent の並列化」だけでは質はほぼ上がらない。4視点の批評で
一致した結論: **精度を上げるのは (A) 決定論的前処理(diffmap)と (C) 反証専任の批判検証**であって、
(B) 並列化は壁時計時間を縮めるだけ。優先順位は A → C → B。

**Why:** LLM は自分の初回出力を sycophantic に追認するため、生成と検証が同一コンテキストだと誤検知
(false positive)が素通りする。また Agent ツールの subagent は親の会話・CLAUDE.md・MEMORY.md を
継承しないため、プロジェクト固有 failure-mode を prompt に明示注入しないと汎用レビューに退行し、
むしろ単一スレッドより精度が落ちる。

**How to apply:**
- **diffmap 前処理**: `gh api .../pulls/{PR}/files` の patch を1回解析し、RIGHT 側実在行の許可集合 +
  各行の is_new_code フラグを作る。これで 422(差分外行)と既存コード誤検知を機械的に封殺できる
  (LLM を増やさず最も費用対効果が高い)。
- **反証専任の批判検証**: 元指摘の結論を伏せて別 subagent に渡し「これを refute せよ」と反証バイアスを
  与える。判定は KEEP/DOWNGRADE/REJECT/REVIEW の4値。安全則「データ破損・セキュリティ・確実なバグは
  REJECT 禁止、迷ったら REVIEW(要確認)」で over-suppression(正しい High の握り潰し=false negative)を防ぐ。
- **subagent には全 diff を配る**(観点別に切り分けない)。横断バグ・離れた箇所のリグレッションを拾うため。
- line は subagent に算出させず code_anchor だけ返させ、親が diffmap で逆引き(算出誤差源の一元化)。
- 規模/effort 両ゲートで起動深度を制御。超大規模 PR は本数でなくターンを分けて batch 鉄則と整合させる。

成果物は `~/.claude/skills/pr-review-e2e/` の SKILL.md + references/{subagent-contracts, gh-inline-review,
e2e-runbook}.md(SKILL.md ~444 行)。関連: [[feedback_batch_small_chunks]]
[[feedback_long_lived_branch_merge_semantic_loss]] [[feedback_merge_resurrects_dead_code]]

**第2弾改修で潰した「骨格は作ったが動かない」落とし穴(実 PR 実測ベース):**
- **diffmap の真実源は patch でなく行テキストを持つ JSON**。code_anchor を grep する対象(text)を JSON に
  保持しないと逆引きが原理的に 0 ヒット。さらに **行頭 diff marker(`+`/スペース)と CRLF(`\r`)の両方を
  両辺で strip** しないと固定文字列マッチがズレる(CRLF だけ揃えるのは片手落ち)。grep は **finding.path に
  限定**(定型行 `if err != nil {` は実 PR で40回出現し別ファイルへ誤確定する)。
- **`gh api .../files` は30件ページネーション**。`--paginate "?per_page=100"` 必須、取得数を PR 総ファイル数と検算。
- **patch==null は binary/truncate/rename を分岐**(binary は許可集合から除外、rename は新パスをキーに)。
- **general-purpose subagent は StructuredOutput を強制できない** → 本文末尾の ```json フェンスで返させ、
  パース失敗は再起動1回→縮退をカバレッジに明記。
- **確定 findings は /tmp に永続化**(承認待ち+E2E でコンテキスト劣化 → detail 消失を防ぐ。再レビュー突合元にも)。
- **Phase 5 投稿前に head SHA 陳腐化チェック**(承認待ち中の push で行ズレ)、**既存コメント重複ガード**
  (再レビューの二重投稿)、**start_line も許可集合突合**(複数行範囲は start_line が外れると全体 422)。
- **観点は固定 failure-mode リストをクローズドにしない**(is_novel_pattern で新種バグの出口を残す=確証バイアス回避)。
- **意図整合性(PR body/Issue vs diff)・依存追加・後方互換破壊(F9)** を観点に追加。
- **gh api は git repo 内で {owner}/{repo} を自動展開する**(「補完できない」は事実誤り)。

**worktree 隔離フロー(並行セッション保護):** Phase 1 静的レビューは専用 worktree(`../kozoka-v2r-prreview-<PR>`)に
隔離しメインレポを占有しない。1-0 で作成(作成前に `git worktree list` で取り残し掃除)→ 1-1〜1-3 を中で実行 →
1-4 で撤去。worktree 内 `pnpm install` 禁止、node_modules はメインから symlink 借用([[feedback_worktree_node_modules_symlink]])
+ **借用前に依存系ファイルのブランチ一致を確認**(別ブランチの node_modules を掴むと型解決汚染)。
**撤去トリガは単一条件に固定**(複数箇所で言い換えると矛盾する。検証で実際に「Phase 2 の前/最初/承認後」が散らばり
事故導線になった): E2E 非進行=Phase 4/5 後、E2E 進行=**切替承認後・`git checkout` 直前**(承認前に撤去しない)。
E2E はメインレポ checkout を要求し worktree と二重チェックアウト衝突するため、この順序が必須。`rm -f symlink` は
リンク先を巻き込まない。完了報告前に `git worktree list` でクリーン確認。
