[← Errilo Auto-fix](../README.md)

# Security

## Why three jobs

The agent reads text that comes from your users: error messages, breadcrumbs, tags. So it never holds a token that can write to your repository. The action runs in three jobs, each with its own permissions:

| Step           | Job          | GitHub token | What it does                                                                    |
| -------------- | ------------ | ------------ | ------------------------------------------------------------------------------- |
| `check`        | check        | read         | Decides whether the issue should be fixed. No agent.                            |
| `prepare`      | fix          | read         | Writes the agent's MCP configuration and instructions.                          |
| `export`       | fix          | read         | After the agent: exports its local commits as a patch artifact.                 |
| `pull-request` | pull-request | write        | No agent. Pushes `errilo/fix-<id>`, opens the PR, comments the Errilo issue.    |

## Safeguards

- **Pull request only.** The agent runs with a read-only GitHub token: it cannot push or open a pull request. It commits on a local branch; `export` turns the commits into a patch, and the `pull-request` job, without agent, pushes them to `errilo/fix-<id>` (never with `--force`, never to the default branch) and opens the pull request. Protect your default branch as well.
- **No change to workflows.** A fix that touches `.github/` is refused.
- **One run per issue.** Errilo delivers alerts "at least once", so the same issue can arrive twice. Runs are grouped by issue id (`concurrency`), and `check` stops if the issue's branch already exists: a duplicate never opens a second pull request.
- **Daily cap.** At most `max-prs-per-day` pull requests (3 by default) in 24 hours, checked before the agent starts and again before the pull request is opened.
- **Untrusted payload.** The alert's `client_payload` contains text from production events (the issue title, the release). The action never writes it into a script: it reads the payload through environment variables, and only uses the issue id, after checking that it is a number.
- **Untrusted event content.** Error messages, breadcrumbs and tags come from your users, possibly from an attacker. The Errilo MCP server marks them as untrusted data, and the `fix_issue` prompt tells the agent never to follow instructions found in them. The read-only token and the review of the pull request limit what an injection could achieve.
- **Only your code.** No fix is attempted when the stack trace only has frames from `node_modules` or Node.js internals.
- **Only real issues.** Test alerts, spikes, and issues that are no longer unresolved (resolved, ignored) are skipped.
- **The agent can say no.** If it cannot reproduce the error with a failing test, or the fix is uncertain, no pull request is opened; the reason is in the run summary.
