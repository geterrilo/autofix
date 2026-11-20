[← Errilo Auto-fix](../README.md)

# Inputs and outputs

## Inputs

| Input             | Steps                 | Default                                   | Description                                                                 |
| ----------------- | --------------------- | ----------------------------------------- | --------------------------------------------------------------------------- |
| `step`            | all                   | required                                  | `check`, `prepare`, `export` or `pull-request`.                              |
| `errilo-token`    | check, pull-request   |                                           | Errilo project token, scopes `issues:read` and `issues:write`.               |
| `errilo-org`      | check, pull-request   |                                           | Errilo organization slug.                                                    |
| `errilo-url`      | all                   | `https://app.errilo.com`                  | Errilo origin, without a path.                                               |
| `issue-id`        | all                   | the alert's, or `inputs.issue_id`         | For `prepare`, `export` and `pull-request`: the `issue-id` output of `check`. |
| `max-prs-per-day` | check, pull-request   | `3`                                       | Cap on auto-fix pull requests opened in 24 hours.                            |
| `github-token`    | check, pull-request   | `github.token`                            | Reads the repository (`check`), pushes and opens the PR (`pull-request`).    |

## Outputs

| Output             | Step         | Description                                                                         |
| ------------------ | ------------ | ----------------------------------------------------------------------------------- |
| `proceed`          | check        | `true` when the issue should be fixed.                                              |
| `issue-id`         | check        | The validated issue id.                                                             |
| `prompt`           | prepare      | The prompt for the agent: the `fix_issue` prompt of the Errilo MCP server.          |
| `claude-args`      | prepare      | Arguments for `anthropics/claude-code-action`: MCP config, instructions, tools.     |
| `mcp-config`       | prepare      | Path of the MCP configuration file, for another agent.                              |
| `instructions`     | prepare      | Path of the CI instructions to append to the agent's system prompt.                 |
| `has-fix`          | export       | `true` when the agent committed a fix.                                              |
| `pull-request-url` | pull-request | URL of the pull request opened.                                                     |

## Another agent

`check`, `export` and `pull-request` do not depend on the agent. To use another agent that supports MCP, give it the files from the `mcp-config` and `instructions` outputs of `prepare` (the agent reads `ERRILO_TOKEN` from its environment), the `prompt` output, and keep the read-only token. Choose the Claude model with `--model` in `claude_args`.

## Versions

`@v1` follows the latest `v1.x.y` release: a workflow moves the `v1` tag each time a release is published. Pin a full commit SHA if your policy requires it.
