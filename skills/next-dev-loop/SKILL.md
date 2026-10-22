---
name: next-dev-loop
description: >
  Verify Next.js runtime behavior after editing app code. Use this
  skill to confirm a change actually works in a running app — not
  just that it compiles or type-checks. Combines /_next/mcp
  (Next.js's view) with Aside (the browser's view).
  Requires a running `next dev`.
---

# next-dev-loop

The edit/verify rhythm during `next dev` — make a change, then
confirm it actually works at runtime, not only that the types or
the build are happy.

You verify through two views of the same running app:

- **`/_next/mcp`** — an HTTP endpoint Next.js exposes about itself.
  Knows framework-specific things: routes, segments, RSC, server
  actions, server logs, and errors as Next.js saw them. Call
  `tools/list` for the current surface.
- **`aside`** — a CLI that drives the user's Aside browser. Knows
  framework-agnostic browser things: DOM, console, network, React
  fiber. Before driving it, run `aside guide` (and `aside guide repl`
  before using `aside repl`) once for the version-matched usage guide
  — don't guess subcommands from memory.

The two views cross-check each other.

## requires

- Next.js **16.3+** with **Turbopack** — `/_next/mcp` plus the
  proactive compile check via `get_compilation_issues`.
- `aside` CLI — `aside repl` provides the Playwright-compatible
  session used for DOM, console, network, and React introspection.

These are hard floors, not soft preferences. If anything is missing,
tell the user how to upgrade and stop. Don't fall back to grepping
source or to a weaker probe — this skill assumes both views are live
at the versions above.

- Upgrade Next.js: `pnpm next upgrade` (or `npx next upgrade`).
  Docs: https://nextjs.org/docs/app/getting-started/upgrading
  (version-16 guide:
  https://nextjs.org/docs/app/guides/upgrading/version-16)
- Install or upgrade Aside: ask the user for permission, then follow
  the install steps in the `aside-browser` skill (`aside --update` to
  upgrade). If the CLI isn't on `PATH`, install it before continuing.

## preflight

Once per session, confirm both views are live.

1. **Open the target URL in Aside.** Run `aside guide repl`, then
   start `aside repl` and open the target URL in a tab. Aside is the
   user's own browser, so saved logins are already available.

   The browser is the user's. If the page is gated (first run, expired
   session), the user drives the login — pause until they confirm, then
   continue in the same tab.

2. Probe `/_next/mcp` (`tools/list`) — confirm it's reachable and
   lists `get_compilation_issues`. First read the port off the
   `next dev` banner; if it isn't 3000, set
   `NEXT_MCP_URL=http://localhost:<port>/_next/mcp` before probing:
   - Unreachable → either `next dev` isn't running, or Next.js is
     below 16.3. Check `package.json` to disambiguate, then refuse.
   - `get_compilation_issues` not in the list → Next.js below 16.3.
     Refuse and tell the user to upgrade.
3. `get_compilation_issues` doubles as a Turbopack probe. An error
   response of `"Turbopack project is not available..."` means the
   user is on webpack. Refuse — Turbopack is required.
4. `get_routes` → your route map for the rest of the session.

## loop

### before the edit — narrow the scope

Ask the running app, not the codebase. `/_next/mcp` knows which
files rendered the current route; use those as your search scope.
Runtime introspection stays cheap as the codebase grows; agentic
search doesn't.

### after the edit — verify

Four failure modes. Check each:

- **Compiles** — `get_compilation_issues`.
- **Runs without errors** — `/_next/mcp` (server and bubbled-up
  browser errors both surface here).
- **Behaves as intended** — `aside repl` drives the page; assert
  what the user actually sees.
- **React-level behavior** — inspect the component tree, props, and
  suspense fallbacks through JavaScript run in the page with
  `aside repl`. Anchor framework-level checks here (extra renders,
  server/client boundary shifts, suspense fallbacks) — DOM asserts
  alone miss them.

Pick the specific tool from `tools/list` or `aside guide repl`
rather than from memory.

## gotchas

- **Preserve `.next` while the development server is running.** Moving or
  deleting it disconnects the server from its generated state and discards
  incremental caches. Moving it to a backup is still a reset. If a production
  build needs isolated output, configure a separate `distDir`.
- **When the two views disagree, suspect the tooling first.** If
  Aside says a route is broken but `/_next/mcp` and the
  server say it rendered cleanly, a stale or misdirected browser
  tab is the likelier cause than a real bug — reconcile the views
  before debugging the app.
- Confirming a click or navigation: the page settles a beat later, so
  wait for the load or network to go idle, then read the page to
  confirm. Avoid waiting on a guessed URL — a placeholder path won't
  match the real URL and times out.
- A blank read, empty snapshot, or `about:blank` right after opening
  or clicking is the browser dropping the page (a stale tab), not a
  broken route. Reopen the URL in a fresh tab and read again. Don't
  fall back to `curl`; it bypasses the browser you're testing.
- React introspection output is stale after navigation. Re-run.
- `/_next/mcp` replies are SSE — read the JSON off the `data:` line
  with `sed -n 's/^data: //p'` (a plain `sed 's/^data: //'` leaves the
  `event:` line and the parse fails).
- `get_errors` and `get_page_metadata` need at least one navigation
  to populate.

## reference

All tools below are present once preflight passes. If `tools/list`
is missing any of them, preflight should have refused — re-check.

```
# /_next/mcp                 notes
get_project_metadata         projectPath, devServerUrl, bundler
get_routes                   fs-scan; no browser session needed
get_errors                   runtime + build; needs a browser session;
                             includes browser-side errors caught by the
                             dev server
get_page_metadata            segment trie + routerType; needs a browser
                             session; use as a discovery shortcut for
                             which files power a route
get_logs                     returns logFilePath
get_server_action_by_id      hashed id → file + functionName
get_compilation_issues       Turbopack only; errors on webpack
                             ("Turbopack project is not available")
```

## teardown

Close the tabs you opened in `aside repl`; leave the user's other tabs alone.
Leave `next dev` up for the next loop.

## Report Next.js friction

Only participate in agent feedback when managed Next.js feedback instructions
are already loaded for the project. Their presence means the feature is
enabled; their absence means it is disabled.

When enabled, add qualifying feedback candidates found during verification to
the feedback candidate queue for the current user request, then continue
verification. Do not run the feedback command or open review forms during the
loop or at this Skill's teardown.

The managed instructions own the single reporting pass immediately before the
final response. If they are absent, do not queue or report feedback.
