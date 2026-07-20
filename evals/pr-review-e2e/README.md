# pr-review-e2e blind evaluation

このdirectoryは、`pr-review-e2e` SkillのHigh/Middle finding精度を継続測定するcontrol planeです。
reviewerへ渡すのは`public/cases.json`、そこから生成した`diffmap.json`、taxonomy、output schema、prompt、Skill snapshotだけです。
`oracle/`、baseline、history、過去predictionをpublic packへ含めてはいけません。

## Corpus

- 16 PR cases
- High: 5
- Middle: 7
- no-finding: 4
- neutral case IDと非連番SHA
- LEFT削除、RIGHT追加、security、migration、compatibility、concurrency、partial write、negative controlを収録

released corpusの意味を変える編集では`corpus_version`を上げます。publicとoracleのcase集合、diff anchor、
taxonomy、closed output schemaは`validate`で同時検証します。

## Commands

repository rootから実行します。

```bash
EVAL=skills/pr-review-e2e/scripts/review_eval.py
ROOT=evals/pr-review-e2e

python3 "$EVAL" validate \
  --cases "$ROOT/public/cases.json" \
  --truth "$ROOT/oracle/ground-truth.json" \
  --taxonomy "$ROOT/issue-taxonomy.json" \
  --prediction-schema "$ROOT/prediction.schema.json"
```

各measurementは3つのfresh packを別directoryへ作ります。verified blind runではpackをworkspace外へ置き、
reviewer subprocessからworkspace rootをmacOS sandboxでread-denyします。

```bash
python3 "$EVAL" pack \
  --cases "$ROOT/public/cases.json" \
  --taxonomy "$ROOT/issue-taxonomy.json" \
  --prediction-schema "$ROOT/prediction.schema.json" \
  --prompt "$ROOT/reviewer-prompt.md" \
  --skill-root skills/pr-review-e2e \
  --run-id <RUN_ID> --replicate-id <1..3> --shuffle-seed <SEED> \
  --reviewer-name codex-default --model default --model-version <VERSION> \
  --reasoning-effort default --isolation-mode verified --output <FRESH_PACK>

python3 "$EVAL" run --pack <FRESH_PACK> --output <PREDICTIONS> \
  --codex-bin <CODEX_BIN> --deny-read-root "$PWD"

python3 "$EVAL" score --pack <FRESH_PACK> \
  --truth "$ROOT/oracle/ground-truth.json" \
  --predictions <PREDICTIONS> --output <SCORE_REPORT>
```

3 reportをpoolし、承認済みbaselineとのabsolute floorとdeltaを同時に検査します。

```bash
python3 "$EVAL" aggregate \
  --score <SCORE_1> --score <SCORE_2> --score <SCORE_3> \
  --measurement-id <MEASUREMENT_ID> --output <AGGREGATE>

python3 "$EVAL" compare --candidate <AGGREGATE> \
  --baseline "$ROOT/baselines/approved.json" --policy "$ROOT/policy.json" \
  --output <COMPARISON>

python3 "$EVAL" record --aggregate <AGGREGATE> --history "$ROOT/history.jsonl"
```

baselineは自動更新しません。3 replicate、全digest、verified blind、absolute floor、baseline deltaが揃った
measurementだけを人手承認で`baselines/approved.json`へ昇格します。live model credentialはtrusted manualまたは
scheduled jobだけで使用し、fork PRではvalidator、unit test、保存済みprediction replayだけを実行します。
