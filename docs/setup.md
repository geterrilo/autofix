[← Errilo Auto-fix](../README.md)

# Setup

You need an Errilo project that receives errors from your app, admin access to the GitHub repository, and either an [Anthropic API key](https://console.anthropic.com/settings/keys) or a Claude subscription (Pro or Max).

## Requirements

- A runner with Actions Runner **v2.327.1 or later**. The action itself is a composite action and runs no Node.js code, but it uses `actions/checkout@v6`, `actions/upload-artifact@v7` and `actions/download-artifact@v8`, which run on Node.js 24. GitHub-hosted runners are always up to date; check the version of self-hosted runners.
- On self-hosted runners: `bash`, `git`, `curl`, `jq` and the GitHub CLI (`gh`), all present on GitHub-hosted Ubuntu runners.
- A working test setup in the `fix` job: the agent needs to install your dependencies and run your tests, otherwise it cannot reproduce the error and stops.

## 1. Add the workflow

Copy [`examples/errilo-autofix.yml`](../examples/errilo-autofix.yml) to `.github/workflows/errilo-autofix.yml` on your **default branch** (GitHub only starts `repository_dispatch` workflows from there). Replace the setup steps of the `fix` job (`setup-node`, `npm ci`) with your project's.

## 2. Send releases with their commit

The agent fixes the code at the version that crashed. Errilo knows that version when your SDK sends a `release` and the release has a commit:

- a release that is a commit SHA (`release: process.env.GIT_SHA`) is enough;
- otherwise, run `sentry-cli releases set-commits <release> --auto` against Errilo when you deploy.

Without a release, the agent works from the default branch, and Errilo cannot mark the issue "resolved in the next release" (the pull request is still opened).

## 3. Create an Errilo token for the project

In Errilo, **Settings → API tokens → New token**: limited to the project of this repository, scopes `issues:read` and `issues:write`. The token only sees this project.

## 4. Secrets and variables

In the repository, **Settings → Secrets and variables → Actions**:

| Name                          | Kind     | Value                                                                                             |
| ----------------------------- | -------- | ------------------------------------------------------------------------------------------------- |
| `ANTHROPIC_API_KEY`           | Secret   | Your Anthropic API key. Or `CLAUDE_CODE_OAUTH_TOKEN`, created with `claude setup-token`, passed as `claude_code_oauth_token`. |
| `ERRILO_TOKEN`                | Secret   | The token from step 2.                                                                            |
| `ERRILO_ORG`                  | Variable | Your organization slug: the first path segment of your Errilo URLs (`app.errilo.com/<slug>/...`). |
| `ERRILO_AUTOFIX_GITHUB_TOKEN` | Secret   | Recommended. See "Pull request token" below.                                                      |

Also turn on **Settings → General → Automatically delete head branches**. The action skips an issue while its `errilo/fix-<id>` branch exists, so a branch left after a merge would block the next attempt on a regression.

### Pull request token

By default, the `pull-request` step uses the workflow's `GITHUB_TOKEN`. It has two limits:

- GitHub refuses to let it open pull requests ("GitHub Actions is not permitted to create or approve pull requests") unless **Settings → Actions → General → Allow GitHub Actions to create and approve pull requests** is on, in the repository and, for an organization, in the organization's settings first;
- GitHub does not start workflows for a pull request it opened, so your CI does not run on the fix.

Pass an `ERRILO_AUTOFIX_GITHUB_TOKEN` secret as `github-token` to avoid both: a fine-grained token on this repository only, with **Contents** and **Pull requests** in read and write, or a GitHub App token. Only the `pull-request` job receives it, never the agent.

## 5. Connect the Errilo alert

Create a fine-grained personal access token on GitHub: **only this repository**, permission **Contents: read and write** (required by GitHub to send a `repository_dispatch`), short expiry. For a repository of an organization, choose the organization as **Resource owner**, and check that an organization admin approved the token if the organization requires it. Otherwise GitHub answers 404 and Errilo shows "The service responded 404".

In Errilo, open the project, then **Alerts → Create a rule**, channel **GitHub**: the repository (`owner/name`), the token, and the triggers **New issue** and **Regression**. Spikes are ignored.

Click **Send a test alert**: a run appears in the Actions tab and stops at once with "test alert received". The wiring works.

## 6. Try it on one issue

**Actions → Errilo auto-fix → Run workflow**, with the id of an issue (the number at the end of its Errilo URL). The run opens a pull request named `fix: Errilo issue <id>`, and the issue shows a comment with the link and the status "resolved in the next release".
