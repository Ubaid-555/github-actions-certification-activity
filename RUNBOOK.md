# GitHub Actions Activity Runbook

This kit covers the seven modules in the Student Activity Guidelines. The main repository contains four workflows. A separate source folder contains the JavaScript action that must be published as its own public repository before the optional invocation step can pass.

## Before publishing

The main repository and the custom action repository must be public per the handout. The source folders intentionally contain no SAP ID. Keep your student name and SAP ID in the local PoC report. Do not put a real credential in a workflow or a screenshot. For the secret-masking demo, configure a disposable dummy value only.

The handout asks the screenshots to show your GitHub identity and repository URL. Capture the GitHub page with the browser address bar visible, your signed-in avatar visible, and the relevant successful run logs expanded.

## 1. Switch the GitHub CLI to Ubaid-555

The stored CLI entries on this PC were both invalid when checked. To remove the old local CLI sign-in and add the intended one, run these in PowerShell:

```powershell
gh auth logout --hostname github.com --user shahaaqi127-debug
gh auth login --hostname github.com --web --git-protocol https
gh auth status
gh api user --jq .login
```

When GitHub opens, sign in as `Ubaid-555`. Enter the password and any two-factor code yourself. If the browser selects another account, choose the option to use a different account. Confirm the final command prints `Ubaid-555` before creating a repository. Never paste a password, two-factor code, or personal access token into chat.

## 2. Create and publish the main public repository

Open PowerShell and run:

```powershell
Set-Location 'C:\Users\HCC\Documents\Codex\2026-11-20\so\outputs\devops-github-actions-activity'
git config --local user.name 'Ubaidullah Zubair'
# Set this to the GitHub noreply address shown in Settings > Emails.
git config --local user.email 'YOUR_GITHUB_NOREPLY_EMAIL'
git init -b main
git add .
git commit -m 'Add GitHub Actions course activity'
gh repo create Ubaid-555/github-actions-certification-activity --public --source . --remote origin --push
```

The email line is a placeholder: replace it with your own GitHub noreply email before committing. Confirm the repository is public and the workflow files appear under `.github/workflows/`.

## 3. Module 1 - Inspect repository Actions settings

Open the repository on GitHub, then **Settings > Actions > General**. Review the action policy and workflow permissions. Record what the page shows. This activity uses explicit `permissions: contents: read` in its workflows and does not need broad write permissions. Keep the configuration least-privileged unless the instructor requires a different setting.

## 4. Module 2 - Trigger workflows

The `activity.yml` workflow includes push, pull request, schedule, `workflow_dispatch`, and `repository_dispatch` triggers. The schedule example is every five minutes so it can produce evidence promptly. It only runs from the default branch; capture one successful scheduled run, then change the schedule to a less frequent interval if you want to reduce repeated runs.

### Push and pull request

Create a `dev` branch and open a pull request to `main`:

```powershell
git switch -c dev
Add-Content README.md "`nDev branch trigger demonstration."
git add README.md
git commit -m 'Demonstrate pull request trigger'
git push -u origin dev
gh pr create --repo Ubaid-555/github-actions-certification-activity --base main --head dev --title 'Demonstrate GitHub Actions triggers' --body 'Course activity: verify the pull_request workflow.'
```

The initial push to `main` triggers the push event. The `dev` push triggers push, and opening the PR triggers pull_request. Expand a successful run for both event types in Actions.

### Manual trigger in the browser

In the repository, open **Actions > Activity - events, contexts, and actions > Run workflow**. Fill in `name`, `greeting`, and `data`, leave `run_self_hosted` off until the runner is registered, then start the run.

### Manual trigger from GitHub CLI

```powershell
gh workflow run activity.yml --repo Ubaid-555/github-actions-certification-activity --ref main -f name='Ubaidullah Zubair' -f greeting='Hello' -f data='Manual CLI demonstration' -f run_self_hosted=false -f run_custom_action=false
gh run list --repo Ubaid-555/github-actions-certification-activity --workflow activity.yml --limit 5
```

For a run ID shown by `gh run list`, inspect it with:

```powershell
gh run watch RUN_ID --repo Ubaid-555/github-actions-certification-activity
```

### External repository_dispatch using a fine-grained PAT

Create a fine-grained PAT in GitHub Settings with access only to this repository and **Contents: Read and write**. Use it only for the curl demo, and revoke it after the screenshot. The request must be POST, with GitHub's JSON Accept header, a Bearer authorization header, an API version header, and a body containing `event_type` plus optional `client_payload`.

Run this PowerShell block. The token is entered in a masked prompt and is not stored in the repository:

```powershell
$owner = 'Ubaid-555'
$repo = 'github-actions-certification-activity'
$securePat = Read-Host 'Enter the temporary fine-grained PAT' -AsSecureString
$pointer = [Runtime.InteropServices.Marshal]::SecureStringToBSTR($securePat)
$pat = $null
try {
  $pat = [Runtime.InteropServices.Marshal]::PtrToStringBSTR($pointer)
  $headers = @{
    Accept = 'application/vnd.github+json'
    Authorization = "Bearer $pat"
    'X-GitHub-Api-Version' = '2022-11-28'
  }
  $body = '{"event_type":"custom-webhook","client_payload":{"greeting":"Hello","data":"External dispatch demonstration"}}'
  $response = Invoke-WebRequest -Method Post -Uri "https://api.github.com/repos/$owner/$repo/dispatches" -Headers $headers -ContentType 'application/json' -Body $body
  "HTTP status: $([int]$response.StatusCode)"
}
finally {
  if ($pointer -ne [IntPtr]::Zero) { [Runtime.InteropServices.Marshal]::ZeroFreeBSTR($pointer) }
  $pat = $null
  $headers = $null
  Remove-Variable securePat,pointer,pat,headers -ErrorAction SilentlyContinue
}
```

The expected response is HTTP 204. The workflow file must already be present on the default branch. Do not include the PAT in a screenshot; capture the terminal after the command, with the token prompt closed and HTTP status visible.

## 5. Module 3 - Contexts and conditionals

The main workflow prints `github.actor`, `github.event_name`, `runner.os`, the `vars.LAB_NOTE` variable, and an environment value. It includes `contains()`, `success()`, `failure()`, and `always()` steps. On a successful run the `failure()` diagnostic is expected to be skipped; the condition is visible in the workflow file.

## 6. Module 4 - Variables, dummy secret, and GITHUB_TOKEN

In **Settings > Secrets and variables > Actions**, create a repository variable named `LAB_NOTE`, such as `Class activity configuration`. Create a repository secret named `MY_SECRET` with a disposable value such as `demo-secret-mask-only-2026`. Never use your password, PAT, or any real account secret for this demonstration.

The workflow displays the dummy secret in one step to demonstrate GitHub masking, so the log should display `***`. The workflow also calls the public repository metadata endpoint using the automatically generated `GITHUB_TOKEN`; it prints the repository name, never the token itself.

## 7. Module 5 - Multi-OS and self-hosted runners

`platform-checks.yml` runs on Windows and macOS. The matrix workflow also runs on Ubuntu, Windows, and macOS. Windows checks PowerShell and Chocolatey; macOS prints the Swift version and compiles/runs `hello.swift`.

For the self-hosted runner, open **Settings > Actions > Runners > New self-hosted runner**, select Windows x64, and follow the current download and configuration commands shown by GitHub. Add the custom label `devops-lab` and keep the runner process open. Do not copy the one-time registration token into chat or commit it. Confirm the runner is **Online/Idle** in Settings.

Then dispatch `activity.yml` with `run_self_hosted=true`. That job is restricted to a manually dispatched run and the custom label; public pull requests cannot send jobs to the local machine.

## 8. Module 6 - Matrix, cache, and PostgreSQL service

`matrix-cache.yml` expands six jobs: three OS images by two Node versions. `actions/setup-node` restores npm's dependency cache using `package-lock.json`. Run the workflow twice; the later successful run should show an exact cache hit. `postgres-service.yml` starts a PostgreSQL container on an Ubuntu runner and runs a query inside that container. GitHub's initialization log should show the service container and health check.

## 9. Module 7 - Publish the custom JavaScript action

Use the separate `devops-custom-action` folder. In PowerShell:

```powershell
Set-Location 'C:\Users\HCC\Documents\Codex\2026-11-20\so\outputs\devops-custom-action'
pnpm install
pnpm exec ncc build index.js -o dist
Test-Path .\dist\index.js
```

Create the second public repository, commit `action.yml`, `index.js`, `package.json`, the generated `pnpm-lock.yaml`, and `dist/index.js`, then publish semantic and major tags:

```powershell
git init -b main
git config --local user.name 'Ubaidullah Zubair'
# Use the same verified GitHub noreply address as above.
git config --local user.email 'YOUR_GITHUB_NOREPLY_EMAIL'
git add action.yml index.js package.json pnpm-lock.yaml .gitignore dist/index.js
git commit -m 'Publish DevOps greeting action v1'
gh repo create Ubaid-555/devops-greeting-action --public --source . --remote origin --push
git tag -a v1.0.0 -m 'Release v1.0.0'
git tag v1
git push origin v1.0.0 v1
```

The main repository's custom-action step uses `Ubaid-555/devops-greeting-action@v1`. Trigger the main workflow with `run_custom_action=true` and verify the greeting output in its logs. The generated `dist/index.js` must be committed; do not commit `node_modules`.

## 10. Evidence and verification

After each module, capture the relevant full-window screenshot with your signed-in GitHub identity and the repository URL visible. Expand the successful run logs before capturing. Use the checklist in `SCREENSHOT_CHECKLIST.md`; keep screenshots in the PoC report, not in the public source repository. The technical answers are in `TECHNICAL_ANSWERS.md`.

A repository with code but without successful GitHub runs and screenshots is not a complete PoC. After all evidence is captured, revoke the temporary PAT and remove or disable the self-hosted runner if you no longer need it.
