# Baselines

承認済みbaselineだけを`approved.json`として置きます。`score`、`aggregate`、`record`はbaselineを更新しません。

初回measurementがabsolute floorを満たさない場合は`measurements/`と`history.jsonl`へ観測値を残し、
このdirectoryへ昇格しません。baseline昇格時は人手承認、measurement ID、Skill digest、model profile、
tool-policy digest、scorer digestを確認してください。
