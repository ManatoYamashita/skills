---
name: devcontainer ターミナルで複数行コマンドが破綻する
description: VSCode devcontainer の zsh が改行付きコマンドを空白連結して誤解釈する。`&&` 連鎖や heredoc が壊れる。シェルスクリプト化して `bash <path>` 1 行で起動する運用に切り替えるべし
type: feedback
originSessionId: 3ce48262-d226-409b-90b0-e48b5d66fc16
---
# 症状

devcontainer (VSCode + zsh) のターミナルで、複数行コマンド (`&&` 連鎖や `<<EOF` heredoc) が改行を空白に置換されて貼られる。結果:

- `git checkout && cd frontend && pnpm vitest run __tests__/...` のように `\` 改行ありで貼ると、`pnpm vitest run` の引数が次行に流れて `zsh: no such file or directory: __tests__/...` となる
- heredoc は `heredoc>` プロンプトに張り付いたまま閉じられず、ターミナルが詰まる

## 何が起きているか

VSCode integrated terminal や Cursor の zsh 補完設定 (auto-suggest など) が、改行付きペーストを「multi-line と認識せず空白連結」する挙動を取ることがある。同じコマンドを Ghostty の zsh / 普通の terminal に貼ると問題なく動く。user の環境固有の癖。

## 対処 (確立した手順)

**Ghostty 側からシェルスクリプトをファイル化して devcontainer に渡す**:

1. project root 配下の `tmp/` (`.gitignore` 済) に `.sh` を Write tool で書く
2. `chmod +x` する
3. user に `bash /workspace/tmp/<name>.sh` の **1 行コマンド** を渡す
4. devcontainer は `/workspace` で project root をマウントしているので Mac 側の `tmp/` がそのまま見える

複雑な heredoc / 連鎖コマンド / SQL 投入はすべてこの方式に揃える。`&&` で繋いだ短い 2-3 step の指示でも、devcontainer 側で動かすなら script 化する方が確実。

# Why

過去 2 セッション連続で同じ症状が起きており、毎回 user の時間を 1〜2 分浪費している (heredoc 詰まり時はターミナル復旧に Ctrl+C が必要)。symptom-fix で `1 行ずつ貼ってください` と案内するより、Ghostty 側で script 化する方が user の作業を確実に成功に導く。

# How to apply

devcontainer 側で実行が必要なコマンドが以下のいずれかに該当するなら、script 化する:
- 2 行以上にわたる
- heredoc を含む (`<<EOF` `<<SQL` など)
- 環境変数を `&&` チェーンで複数コマンドに渡す (`X=$(uuidgen) && do_something_with $X && do_more_with $X`)
- パイプや複雑なクォートを含む

単純な 1 行コマンド (`pnpm vitest run`、`git status` など) は script 化不要、そのまま user に伝える。

`tmp/` ディレクトリは project root に作成済 (`.gitignore` 登録済)。スクリプトは検証用の使い捨てなので commit しない方針。
