# PoC Screenshot Checklist

All evidence screenshots must show (1) the signed-in GitHub avatar or username and (2) the browser address bar with the repository URL. Expand the relevant successful run logs before capture. Never show passwords, one-time codes, registration tokens, PATs, or real secret values.

- [ ] Module 1: Settings > Actions > General, permissions visible; repository URL and GitHub identity visible.
- [ ] Module 2A: Successful push run and successful pull_request run, with event badges.
- [ ] Module 2B: Scheduled run listed as Scheduled on the default branch.
- [ ] Module 2C: Manual dispatch form with inputs, successful run, and CLI output.
- [ ] Module 2D: Terminal showing repository_dispatch HTTP 204 and the resulting Actions run. Do not show the PAT.
- [ ] Module 3: Expanded context step showing actor, event, runner OS, variable, and environment output.
- [ ] Module 4: Settings > Secrets and variables; workflow log shows only a dummy secret masked as `***` and a GITHUB_TOKEN-authenticated API result.
- [ ] Module 5A: Successful Windows and macOS jobs; the Ubuntu matrix job also appears.
- [ ] Module 5B: Settings > Actions > Runners showing the custom runner online/idle.
- [ ] Module 6A: Matrix grid with OS and Node combinations.
- [ ] Module 6B: Cache step with cache miss/save on first run and cache restored on a later run.
- [ ] Module 6C: PostgreSQL container initialization/health log and successful query.
- [ ] Module 7: Custom action repository showing `action.yml`, `index.js`, `package.json`, `dist/index.js`, and successful greeting output from the main workflow.
