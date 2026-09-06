# skills

Codex向けの自作Skillsと、運用で蓄積した汎用メモリを収録するworkspace。

## 構成

```
.
├── skills/            # Codex Skills (SKILL.md + references/)
│   ├── better-*/                   # Claudeから移植したUIレビュー基盤
│   ├── break/                      # コンポーネントの状態・境界値検証
│   ├── explain-interface/          # UI実装の調査・説明
│   ├── frontend-design/            # ブリーフ固有のフロントエンド設計
│   ├── interface-review/          # 変更スコープのUIレビュー
│   ├── review2/                    # pr-review-e2e の後継
│   ├── tabelog-review/
│   ├── variant/                   # UIバリエーション比較
│   └── google-maps-review-writing/
├── evals/             # Skillのblind評価corpus、oracle、policy
│   └── pr-review-e2e/              # review2 の評価資産（旧名のまま）
├── memories/          # 汎用 feedback メモリ(作業の進め方の教訓)
│   └── MEMORY.md      # 索引
└── README.md
```

## skills/

### review2（旧 pr-review-e2e）

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

`SKILL.md`、`references/`、3つの検証script、unit test、`agents/openai.yaml`で構成する。評価資産と運用手順は`evals/pr-review-e2e/`に置く(スキル名を`pr-review-e2e`から`review2`へ改名したが、評価資産は旧名のまま)。

### UI系Skill（Claudeから移植）

以下の12本をCodex Skill形式へ移植している。`better-interface` は各ドメインSkillを束ね、`better-writing` はその依存Skillとして含めている。

- `better-accessibility` / `better-colors` / `better-interface`
- `better-layout` / `better-typography` / `better-ui` / `better-writing`
- `break` / `explain-interface` / `frontend-design` / `interface-review` / `variant`

リポジトリ版は `skills/` に、現在のCodexが自動検出する実行版は `/Users/manatoy_mba/.agents/skills/` に同期している。`explain-interface`、`interface-review`、`break`、`variant` は明示呼び出し専用として登録している。

### 全Skill一覧（2026-09-06）

```text
agent-browser
ai-elements
alloydb-basics
better-accessibility
better-colors
better-interface
better-layout
better-typography
better-ui
better-writing
bigquery-basics
break
building-components
cloud-run-basics
cloud-sql-basics
deploy-to-vercel
explain-interface
find-skills
firebase-basics
frontend-design
gemini-api
gke-basics
google-cloud-recipe-auth
google-cloud-recipe-networking-observability
google-cloud-recipe-onboarding
google-cloud-waf-cost-optimization
google-cloud-waf-reliability
google-cloud-waf-security
google-maps-review-writing
grill-me
grill-with-docs
gsap-core
gsap-frameworks
gsap-performance
gsap-plugins
gsap-react
gsap-scrolltrigger
gsap-timeline
gsap-utils
interface-review
migrate-radix-to-base
modern-web-guidance
next-best-practices
next-cache-components
next-upgrade
review2
shadcn
tabelog-review
variant
vercel-cli-with-tokens
vercel-composition-patterns
vercel-react-best-practices
vercel-react-native-skills
vercel-react-view-transitions
web-design-guidelines
```

同期時は、まず `skills/` を正本としてSkillディレクトリを追加・更新し、各Skillの `SKILL.md` と必要な `agents/openai.yaml`・参照ファイルを `/Users/manatoy_mba/.agents/skills/` へ同一内容で反映する。同期後はSkill数、`SKILL.md` の存在、frontmatter、内部リンク、両ディレクトリの差分を確認する。

### tabelog-review

グルメインフルエンサー風の食べログ(Tabelog)レビュー投稿のタイトルと本文を作成するSkill。
情報収集で裏付けの取れた事実のみを使い、人間らしいプレーンテキストのレビューを生成する。

### google-maps-review-writing

Googleマップのクチコミ(特に日本語300文字以上)を作成・推敲するSkill。ブラウザでレビュー画面を
直接操作できる場合、店舗名・評価・入力済みテキストをページから読み取り、公式サイトなど信頼できる
情報源で裏付けを取ってから下書きを入力する。投稿ボタンはユーザーが明示的に指示しない限り押さない。

## memories/

Claude Code の運用で蓄積した **feedback 系メモリ**(作業の進め方に関する教訓)を収録。
`MEMORY.md` が索引で、各ファイルは1つの教訓を frontmatter 付きで保持する。

プロジェクト固有の機密(アカウント・チーム ID・インフラ構成・個人情報を含む `project_*` /
`user_profile` など)は**意図的に除外**している。
