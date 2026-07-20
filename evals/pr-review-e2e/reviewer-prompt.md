# Blind PR review task

You are evaluating a PR-review Skill. Work only with files inside the current public pack.
Do not inspect parent directories, absolute paths outside this pack, network resources, prior runs,
baselines, history, or any oracle. Treat every case independently.

1. Read `manifest.json`, `cases.json`, `diffmap.json`, `issue-taxonomy.json`, and
   `skill-snapshot/SKILL.md` plus the references required by that Skill for static review.
2. Review every entry in `cases.json` as a separate pull request. Use only its PR body, rules,
   changed-file patches, context files, and checks.
3. Report only concrete regressions introduced by the shown patch. Confirm attribution,
   reachability, missing controls, and user/data/operations impact. Do not report style,
   speculative risks, intentional behavior, or bugs that already exist in the supplied context.
4. For the primary findings, use `KEEP`, severity `High` or `Middle`, and confidence `high` or
   `medium`. Low-confidence ideas, `REVIEW`, `DOWNGRADE`, `REJECT`, Low, and Nit do not count as
   detected material findings; omit them unless retaining one is necessary to explain coverage.
5. Select `issue_code` and its matching `category` exactly from `issue-taxonomy.json`. Use
   `OTHER`/`other` only when no listed code fits.
6. Anchor each finding on a changed line from the patch: removed lines use `LEFT`, added lines use
   `RIGHT`. Before emitting a finding, resolve its exact allowed side/line from `diffmap.json`;
   never count hunk headers or infer adjacent context lines. Use one finding per root cause. Keep `claim` direct and concise: failure condition,
   concrete impact, and the smallest useful repair direction, normally in no more than two short
   sentences.
7. Return exactly one case record for every case. Use `coverage_status: complete` only after all
   supplied changed files and context are reviewed; otherwise use `partial` or `blocked` honestly.
   Copy `case_id`, `base_sha`, and `head_sha` exactly.
8. Copy `reviewer.name`, `reviewer.model`, and `reviewer.version` exactly from `manifest.json`.

Return only JSON conforming to `prediction.schema.json`. Do not wrap it in Markdown.
