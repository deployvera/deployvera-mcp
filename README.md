# DeployVera for AI agents

<img src="assets/logo.png" alt="DeployVera" width="96" align="right">

Connect coding agents (Claude Code, Cursor, VS Code, Codex and any other MCP client) to your existing [DeployVera](https://deployvera.com) projects. The agent can list projects, deploy, follow builds, read logs, get a plain-language explanation of a failed build, and manage environment variables. It uses DeployVera's hosted MCP server at `https://mcp.deployvera.com/mcp` (Streamable HTTP), listed in the official MCP Registry as `com.deployvera/mcp`. This repository also contains a Claude Code plugin with a deploy workflow skill.

## Get a token

1. Sign in to the [DeployVera panel](https://panel.deployvera.com).
2. Open **Settings > AI agent access** and create a token.
3. Pick the scopes (`read`, `deploy`, `env`) and a lifetime (30, 90 or 365 days).
4. Copy the token (`dvp_...`). It is shown once.

Use the smallest scope set that works. A `read` token cannot change anything.

## Install

### Claude Code

Plugin (MCP server plus the deploy skill):

```sh
export DEPLOYVERA_TOKEN=dvp_...
```

```
/plugin marketplace add yusufcelalcelik/deployvera-agent
/plugin install deployvera@deployvera
```

Set `DEPLOYVERA_TOKEN` in the environment Claude Code is started from. Run `/mcp` to check the connection.

MCP server only:

```sh
claude mcp add --transport http deployvera https://mcp.deployvera.com/mcp --header "Authorization: Bearer dvp_..."
```

### Cursor

[Add to Cursor](cursor://anysphere.cursor-deeplink/mcp/install?name=deployvera&config=eyJ1cmwiOiJodHRwczovL21jcC5kZXBsb3l2ZXJhLmNvbS9tY3AiLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgWU9VUl9ERVBMT1lWRVJBX1RPS0VOIn19)

The link installs a placeholder token (`YOUR_DEPLOYVERA_TOKEN`); replace it in Cursor's MCP settings after installing. Or add this to `~/.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "deployvera": {
      "url": "https://mcp.deployvera.com/mcp",
      "headers": { "Authorization": "Bearer dvp_..." }
    }
  }
}
```

### VS Code

Add to `.vscode/mcp.json` or the user `mcp.json`. VS Code asks for the token once and stores it securely.

```json
{
  "inputs": [
    { "type": "promptString", "id": "deployvera-token", "description": "DeployVera token (dvp_...)", "password": true }
  ],
  "servers": {
    "deployvera": {
      "type": "http",
      "url": "https://mcp.deployvera.com/mcp",
      "headers": { "Authorization": "Bearer ${input:deployvera-token}" }
    }
  }
}
```

### Codex

Add to `~/.codex/config.toml` and export the token in your shell:

```toml
[mcp_servers.deployvera]
url = "https://mcp.deployvera.com/mcp"
bearer_token_env_var = "DEPLOYVERA_TOKEN"
```

### Windsurf, Gemini CLI and other clients

Any client that supports remote MCP over Streamable HTTP works. Use:

- URL: `https://mcp.deployvera.com/mcp`
- Header: `Authorization: Bearer dvp_...`

Check your client's documentation for where to enter a remote server URL and headers. For clients that only support stdio, use `npx -y @deployvera/mcp` with the `DEPLOYVERA_TOKEN` environment variable.

## Tools

| Tool | Scope | What it does |
|------|-------|--------------|
| `list_projects` | read | Projects with status, live URL, repo/branch, last build |
| `get_project` | read | Details, latest build, and whether the live commit is behind the branch |
| `get_build` | read | Build status (latest by default); `wait_seconds` (0-120) polls until it ends |
| `get_build_logs` | read | Tail (default 200, max 1000) of the sanitized build log, or runtime log |
| `explain_build_failure` | read | Reason code, plain-English cause, fix steps and the last log lines |
| `list_env_vars` | env | Keys and flags only. Values are never returned |
| `set_env_vars` | env | Bulk set `{KEY: value}`; optional `build_time` and `apply` (restart) |
| `delete_env_var` | env | Delete a variable by key |
| `deploy` | deploy | Deploy the branch head, or a specific `commit_sha`; returns the build id |
| `rollback` | deploy | Redeploy the retained image of an earlier build |
| `cancel_build` | deploy | Cancel a queued or running build |

## Example prompts

- "Deploy my latest commit and tell me if it fails."
- "Why did my last build fail? Fix it and redeploy."
- "Which of my projects is behind its branch?"
- "Add DATABASE_URL to the api project and restart it." (give the value only if you are comfortable sharing it with the agent)
- "Show the last 100 runtime log lines of my site."

## Security

- Scopes: `read`, `deploy`, `env`. A token only does what its scopes allow.
- Environment variable values are write-only: masked in `list_env_vars`, never echoed by `set_env_vars`, scrubbed from error text. A value you type into a prompt is visible to that agent.
- Tokens cannot create projects, change billing, domains or account settings, or reach admin routes. Use the panel for those.
- Tokens expire after 30, 90 or 365 days and can be revoked at any time in **Settings > AI agent access**.
- Rate limit: 120 requests per minute.
- The hosted server is stateless and does not store tokens. Keep tokens out of repositories; prefer an environment variable or your client's secret prompt over a literal value in a config file.

## Links

- Website: <https://deployvera.com>
- Panel: <https://panel.deployvera.com>
- Documentation: <https://deployvera.com/docs/en/quickstart>
- Privacy policy: <https://deployvera.com/legal/privacy-policy>
- Terms of service: <https://deployvera.com/legal/terms-of-service>
- Support: <support@deployvera.com>

## License

MIT. See [LICENSE](LICENSE). Logo: DeployVera mark, all rights reserved by DeployVera.
