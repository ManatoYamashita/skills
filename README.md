# skills

Codex向けの自作Skillsと、運用で蓄積した汎用メモリを収録するworkspace。

## 構成

```
.
├── skills/            # Codex Skills (SKILL.md + references/)
│   └── pr-review-e2e/
├── evals/             # Skillのblind評価corpus、oracle、policy
│   └── pr-review-e2e/
├── memories/          # 汎用 feedback メモリ(作業の進め方の教訓)
│   └── MEMORY.md      # 索引
└── README.md
```

## skills/

### pr-review-e2e

GitHub PRをリポジトリ固有のルール、CI、技術スタックへ適応して多観点レビューし、必要に応じて
ビルド、テスト、ブラウザE2Eまで検証する汎用Skill。結果とmerge判断を先にチャットへ提示し、
ユーザーの明示承認後にだけGitHubへ投稿する。

精度を支える設計:

- **repository adaptation** — PRとlocal repositoryのidentityを照合し、base snapshotの`AGENTS.md`、CI、manifestからreview profileを作る。
- **高シグナルrubric** — PR帰属、到達可能性、具体的影響、証拠、修正可能性を満たす候補だけをfinding化。
- **独立した反証** — `KEEP / DOWNGRADE / REJECT / REVIEW`で誤検知と低確度推測を分離。
- **完全性を検証するdiffmap** — GitHub patchからLEFT削除行とRIGHT追加・context行を正規化し、API件数との不一致や部分patchをfail-closedで検出。
- **coverage ledgerと証拠段階** — 全変更ファイルの担当・risk・未取得を可視化し、支持証拠、反証、base/head差分で精度を上げる。
- **簡潔なコメント生成** — 内部証拠を保持したまま、1根本原因・原則2文・240文字以内のGitHub本文へ中央変換し、diffmap位置も機械検証。
- **payload単位の承認** — 最終event・位置・本文を`head SHA + payload hash`へ束縛し、内容変化時は再承認。
- **安全な実行検証** — detached worktree、信頼できないforkの隔離、変更リスクに応じた段階的テスト。
- **承認制E2E** — UIやruntime riskがある場合だけ実施し、devcontainerでメインrepo切替が必要なら状態・影響・復元手順を示して明示承認を得る。
- **blind継続評価** — oracleをreviewerから隔離し、16ケースをfresh sessionで3回評価。High/Middle別precision・recall、severity一致率、negative FP、baseline差分を決定論的に記録する。

`SKILL.md`、`references/`、3つの検証script、unit test、`agents/openai.yaml`で構成する。評価資産と運用手順は`evals/pr-review-e2e/`に置く。

## memories/

Claude Code の運用で蓄積した **feedback 系メモリ**(作業の進め方に関する教訓)を収録。
`MEMORY.md` が索引で、各ファイルは1つの教訓を frontmatter 付きで保持する。

プロジェクト固有の機密(アカウント・チーム ID・インフラ構成・個人情報を含む `project_*` /
`user_profile` など)は**意図的に除外**している。
