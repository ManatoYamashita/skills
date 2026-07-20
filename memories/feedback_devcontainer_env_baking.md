---
name: devcontainer-env-baking
description: "kozoka-v2r devcontainer は compose.yaml で env_file: ../backend/app/.env を指定しており、.env の値が OS env として焼き付けられる。.env 編集後に backend だけ再起動しても godotenv.Load() は OS env を上書きしない仕様のため反映されない"
metadata: 
  node_type: memory
  type: feedback
  originSessionId: ef6d648c-d22d-4b8d-be06-1701e3869f83
---

kozoka-v2r で backend/app/.env を編集してもモデル設定 (GEMINI_TRANSCRIPTION_MODEL 等) が反映されないケースの対処。

**Why:** `.devcontainer/compose.yaml:22-23` が `env_file: ../backend/app/.env` で .env を OS env として焼き付ける。`backend/app/internal/config/config.go:63` の `godotenv.Load()` は **OS env を上書きしない仕様** なので、.env の新値は永遠に反映されない。`make run-dev` で backend を再起動しても無駄。実測で `docker inspect kozoka-v2r-devcontainer --format '{{range .Config.Env}}...'` に焼き付きが確認された。

**How to apply:**
- .env 編集後にモデル/フィーチャーフラグ等を切り替えたい時は以下のいずれか:
  1. **VSCode で `Dev Containers: Rebuild Container`** (本番想定に最も近い、確実)
  2. **同じ shell で `unset <KEY>` → `make run-dev` 再起動** (実験向き、最速)
  3. **`export <KEY>=<新値>` → 同じ shell で `make run-dev`** (手動上書き)
- 検証は **`cat /proc/$(pgrep -f cmd/server/main)/environ | tr '\0' '\n' | grep <KEY>`** で実プロセスの env を直視
- 関連: DB 接続/環境変数まわりの運用方針(プロジェクトメモリ)、設定経路 trace は `bootstrap/initialize.go:296,322-323`
- 根治案: `config.go:63` を `godotenv.Overload()` に変える / compose.yaml の env_file を削除する / 起動時ログに `cfg.Gemini*Model` を fmt.Printf で吐く debug 改修
