# メモリ索引(公開版)

このリポジトリには、Claude Code の運用で蓄積した **feedback 系メモリ**(作業の進め方に関する教訓)のみを
収録している。プロジェクト固有の機密(アカウント・インフラ・個人情報を含む `project_*` / `user_profile` 等)は
意図的に除外している。各行は `[タイトル](ファイル) — 要点` の形式。

- [PRレビュー精度の核](feedback_pr_review_precision_design.md) — 並列化でなくdiffmap前処理+反証専任の批判検証(4値KEEP/DOWNGRADE/REJECT/REVIEW)。subagentは親メモリ非継承なので失敗モードを明示注入
- [上長へのメンション催促は避ける](feedback_no_mention_upstream.md) — PR承認待ちでも @mention 催促は提案しない、記録系の打ち手のみ
- [flex truncate は親チェーン全段の min-w-0 が必要](feedback_flex_truncate_minw0_chain.md) — max-w/truncate が効かず見切れる時は中間ラッパーの min-w-0 欠落を疑う
- [pnpm dev / pnpm install は devcontainer 側](feedback_dev_server_ownership.md) — Ghostty で pnpm dev も pnpm install も禁止。前者は OOM、後者は native binding 上書きで dev server を Build Error させる
- [Next.js RSC無限ループのデバッグ](feedback_rsc_infinite_loop.md) — GET /?_rsc=連打時はNetwork Initiator見る。hook返却関数はuseCallback必須、useSearchParamsは.toString()化
- [刷新revert時にbug fixコミットを巻き戻さない](feedback_revert_protect_bugfix.md) — 時点 checkout 前に git log -- <file> で刷新以外の修正を抽出して cherry-pick で救う
- [ハイブリッド route 構造の選好](feedback_hybrid_route_group.md) — Next.js (app) route group はそのまま受け入れず、フラット + 機能側 layout に分解する設計を好む
- [React 19 で useRef は初期値必須](feedback_react19_useref_init.md) — useRef<T>() は TS2554 エラー、useRef<T | undefined>(undefined) で逃がす
- [devcontainer ターミナルで複数行コマンドが破綻](feedback_devcontainer_multiline_break.md) — &&連鎖/heredoc が空白連結で壊れる。tmp/<name>.sh 化して bash 1 行で渡す運用に
- [長寿命ブランチ merge は意味検証必須](feedback_long_lived_branch_merge_semantic_loss.md) — git status conflict 0 件でも auto-merge が依存関係を破壊。go build/vet/test と atlas validate を必ず走らせる
- [merge で復活する dead code は lint v2 まで通さないと露見しない](feedback_merge_resurrects_dead_code.md) — 「片方が消した dead code をもう片方が後から再追加」した場合、build/vet/test では検出不能。CI の golangci-lint unused を信頼
- [単一プロジェクト構成での pnpm filter の罠](feedback_pnpm_filter_noop_exit0.md) — pnpm-workspace.yaml 無し。`pnpm --filter frontend` は exit 0 で素通りする罠。cd frontend && pnpm 直叩き。typecheck は tsc --noEmit
- [worktree の tsc/lint は node_modules を symlink で借りる](feedback_worktree_node_modules_symlink.md) — worktree 側 pnpm install は禁止。メイン repo の frontend/node_modules を ln -s で貸し、検証後 rm で撤去
- [Turbopack stale chunk 対処](feedback_turbopack_stale_chunk.md) — ファイル編集済 & tsc/vitest 緑なのにブラウザ React Fiber が古い関数体を返すなら `rm -rf .next` → pnpm dev 再起動 → Cmd+Shift+R
- [pnpm test の filter は -- なしで渡す](feedback_pnpm_test_filter.md) — `pnpm test -- topics-section` は filter 無視で全件走る。`pnpm test topics-section` か `pnpm exec vitest run <pattern>` で渡す
- [devcontainer は .env を OS env として焼き付ける](feedback_devcontainer_env_baking.md) — compose.yaml env_file 経由で焼き付き、godotenv.Load() は OS env 非上書き。.env 編集後は Rebuild or unset → make run-dev で対処
- [ブランチ切替時はbackend再起動必須](feedback_backend_restart_on_branch_switch.md) — git checkout しても make run-dev のGoバイナリは古いまま動く。e2e で挙動差が出たら一番に疑う
- [useReportSentimentsをcontainerで呼ぶとpolling停止](feedback_useReportSentiments_polling_conflict.md) — audio-detail-container 直接呼びは useReportProgress を妨害。子セクション内で呼ぶ運用に
- [edge検知useEffectで初回prefillは禁忌](feedback_useEffect_edge_detect_no_prefill.md) — 初回観測時の silent prefill は polling race で 0 toast。per-key last-status ref + 真の遷移検知に倒す
- [host vitest は darwin-arm64 binding 3 種を手動配置](feedback_host_vitest_native_binding.md) — pnpm install せず worktree + rsync + rolldown/lightningcss/@tailwindcss-oxide の npm pack 配置で完走
- [batch は小さく刻む / 1ターン1ツール](feedback_batch_small_chunks.md) — browser_batch や複数行 Bash の長いネストは malformed→出力停止を招く。3〜4 アクションに抑える
- [ProseMirror大量テキストで idle timeout](feedback_prosemirror_idle_timeout.md) — エディタに数千バイト投入すると screenshot/get_page_text/javascript_tool が45s timeout。console/network 等 idle非依存ツールで代替
