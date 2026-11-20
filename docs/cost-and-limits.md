[← Errilo Auto-fix](../README.md)

# Cost and limits

## Cost

With an API key, each run is billed by Anthropic according to the tokens used; follow it in the [Anthropic Console](https://console.anthropic.com/usage). With a subscription token, runs count toward your plan's usage limits. For scale: on a small demo bug (an unknown discount code that crashes a cart function), a run took 7 agent turns, 30 seconds, and $0.08. A real code base costs more: the agent reads more code and runs a longer test suite.

Three settings bound the cost: `max-prs-per-day`, `--max-turns` in `claude_args` (80 in the example), and the `timeout-minutes` of the `fix` job. Runs also use GitHub Actions minutes.

## Limits

- **Review every pull request.** The agent writes a test that reproduces the error, but a test that passes does not prove the fix is right.
- **Bots cannot trigger Claude Code.** `claude-code-action` only runs for a human GitHub user: the dispatch token must belong to a person, not a GitHub App.
- **One issue per run.** An alert that groups several issues sends one dispatch per issue (10 at most); each one starts its own run.
- **GitHub.com only.** Errilo sends `repository_dispatch` to `api.github.com`; GitHub Enterprise Server is not supported.
