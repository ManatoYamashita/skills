---
name: 刷新 revert 時に bug fix コミットを巻き戻さない
description: Phase X revert で特定ファイルを古い時点に戻す際、そのファイルに後続の bug fix が入っていないか git log で必ず確認する。
type: feedback
originSessionId: df59d2a9-a6a2-46d1-907e-46702eb59210
---
**ルール: 大規模 revert (Phase X 戻し / タイムマシン checkout) を行う際、対象ファイルの直近 commit 履歴を git log で確認し、刷新と無関係な bug fix が混入していないか個別に検証する**

## Why
2026-04-23 の `/` ページ revert で、`hooks/useReportFilters.ts` と `contexts/TeamContext.tsx` を「Phase 0 完了直後 (`b91f09ad0`)」時点に戻した結果、**後続の PR #884 (`db8e8986d` — RSC 無限連打 bug fix)** まで一緒に巻き戻ってしまい、dev server で GET / が 10〜30ms 間隔で連打される現象が再発した。

PR #884 は Issue #859 デザイン刷新とは別系統の Next.js 16 / React 19 移行由来の bug fix。revert スコープに含める理由が無いのに、ファイル単位で時点 checkout したため巻き込まれた。

発見: ユーザーから「UseEffect の依存配列に間違ったものが入っていて RSC 要求が 10〜30ms 間隔で無限発火してる?」とスクリーンショット付きで指摘された。Initiator カラムの `useReportFilters.t...` が決定的証拠。

## How to apply
revert 計画を立てる段階で:

1. **対象ファイル毎に `git log --oneline <file>` を実行**し、revert 起点より後のコミットが刷新由来か bug fix / リファクタリング由来かを分類
2. bug fix コミットが見つかったら:
   - そのコミットだけ cherry-pick で上書き復元する手順を計画に明記
   - または revert 起点をそのコミット直後に変更する
3. **Explore agent に「revert 対象ファイルに刷新と無関係な修正が入っていないか、git log でチェックせよ」と明示**して調査させる。単純な「現在 vs 刷新前の diff」だけでは bug fix が埋もれる

## 具体的な手順例
```bash
# 1. revert 対象ファイルの最近 10 コミットを確認
for f in <target_files>; do
  echo "=== $f ==="
  git log --oneline -10 "$f"
done

# 2. bug fix が見つかったら、revert 起点より後なら個別 cherry-pick で救う
git checkout <pre-renewal-commit> -- "$f"    # revert
git checkout <bugfix-commit> -- "$f"          # bug fix を上書き復元
```

## 関連
- Next.js RSC 無限ループの検出・修正は `feedback_rsc_infinite_loop.md` に詳細記載
- 今回の発現は AuthGuard / TeamContext / useReportFilters 3 ファイルが同一 PR (#884) で修正されていた。AuthGuard は revert リストから漏れていたので無傷、TeamContext / useReportFilters は revert 対象に含めてしまい巻き戻り
