---
name: feedback-backend-restart-on-branch-switch
description: ブランチ切替後の e2e 検証では devcontainer 側 make run-dev を必ず再起動する。さもないと古いバイナリの挙動を新ブランチの挙動と誤認する
metadata: 
  node_type: memory
  type: feedback
  originSessionId: e3c21c1b-1628-4525-b1a7-9f52925e5111
---

メインリポを別ブランチに `git checkout` しても、devcontainer の backend サーバー (`make run-dev` で起動した Go プロセス) は **古いバイナリのまま動き続ける**。

**Why:** Go はビルド済みバイナリを起動するため、ソースを変更しただけではプロセスに反映されない。kozoka-v2r では `make run-dev` がホットリロード非搭載で、build と run が一発実行のため、コード変更後は手動再起動が必須。e2e で「修正後の挙動が出ない」と勘違いして余計な原因調査をする時間を防ぐ。

**How to apply:**
- ブランチ切替直後の e2e 検証では、必ずユーザーに「devcontainer で `make run-dev` を Ctrl+C → 再実行したか」を確認する
- 「実機で期待挙動と違う」観測を得たら、まず backend バイナリの古さを疑う(ブランチ別デプロイ経路の方針と組み合わせて経路全体を点検)
- 関連事象: [[feedback_devcontainer_env_baking]] (env が baking されているのと同様、binary も baking されている)

実例: spec-033-template-1llm K の e2e で 1 回目「sentiment 16→29 件 (走った)」、2 回目「sentiment 29→29 件 (温存)」と挙動が変わったのは、1 回目は backend が古い fix/1118 ブランチのバイナリで動いていたため。再起動後は新バイナリで Issue #1065 のスキップ経路が正しく機能した。
