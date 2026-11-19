# DeployVera for AI agents

[![mcptry](https://mcptry.com/badge/com.deployvera/mcp)](https://mcptry.com/servers/com.deployvera/mcp)

<img src="assets/logo.png" alt="DeployVera" width="96" align="right">

Connect coding agents (Claude Code, Cursor, VS Code, Codex and any other MCP client) to your existing [DeployVera](https://deployvera.com) projects. The agent can list projects, deploy, follow builds, read logs, get a plain-language explanation of a failed build, check managed databases, manage environment variables, create new projects and connect custom domains (both with browser approval). It uses DeployVera's hosted MCP server at `https://mcp.deployvera.com/mcp` (Streamable HTTP), listed in the official MCP Registry as `com.deployvera/mcp`. This repository also contains a Claude Code plugin with a deploy workflow skill, plus Cursor and Gemini CLI manifests.

## Get a token (only for token-based setups)

Browser sign-in needs no token. For clients without OAuth support, or for CI, create one:

1. Sign in to the [DeployVera panel](https://deployvera.com/clientarea).
2. Open **Settings > AI agent access** and create a token.
3. Pick the scopes (`read`, `deploy`, `env`, `support`) and a lifetime (30, 90 or 365 days).
4. Copy the token (`dvp_...`). It is shown once.

Use the smallest scope set that works. A `read` token cannot change anything.

## Install

### Sign in with browser (recommended)

Add the server URL without any token. The client opens your browser, you sign in to DeployVera and choose the scopes and duration, and the client stores the login. Nothing to copy or paste. Connected apps are listed, and can be disconnected, in **Settings > AI agent access**.

**Claude Code**: plugin (MCP server plus the deploy skill):

```
/plugin marketplace add deployvera/deployvera-mcp
/plugin install deployvera@deployvera
```

Then run `/mcp`, select `deployvera` and authenticate. Or add the server only:

```sh
claude mcp add --transport http deployvera https://mcp.deployvera.com/mcp
```

**Cursor**: [![Add to Cursor](https://cursor.com/deeplink/mcp-install-dark.svg)](https://cursor.com/install-mcp?name=deployvera&config=eyJ1cmwiOiJodHRwczovL21jcC5kZXBsb3l2ZXJhLmNvbS9tY3AifQ%3D%3D)

One click adds the server; Cursor then opens the DeployVera sign-in. Or add it to `~/.cursor/mcp.json` (the repository also works as a Cursor plugin with a root `mcp.json`):

```json
{
  "mcpServers": {
    "deployvera": { "url": "https://mcp.deployvera.com/mcp" }
  }
}
```

**VS Code** (`.vscode/mcp.json` or the user `mcp.json`):

```json
{
  "servers": {
    "deployvera": { "type": "http", "url": "https://mcp.deployvera.com/mcp" }
  }
}
```

**Gemini CLI**: the repository root is a Gemini CLI extension (`gemini-extension.json`) with the same URL, no token. Gemini CLI starts the browser login on the first 401 response.

### With a token

Use this when the client has no OAuth login, or for scripts. Create a token first (see above).

**Claude Code**

```sh
claude mcp add --transport http deployvera https://mcp.deployvera.com/mcp --header "Authorization: Bearer dvp_..."
```

**Cursor**: [Add to Cursor](https://cursor.com/install-mcp?name=deployvera&config=eyJ1cmwiOiJodHRwczovL21jcC5kZXBsb3l2ZXJhLmNvbS9tY3AiLCJoZWFkZXJzIjp7IkF1dGhvcml6YXRpb24iOiJCZWFyZXIgWU9VUl9ERVBMT1lWRVJBX1RPS0VOIn19) installs a placeholder token (`YOUR_DEPLOYVERA_TOKEN`); replace it in Cursor's MCP settings. Or in `~/.cursor/mcp.json`:

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

**VS Code**: VS Code asks for the token once and stores it securely.

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

**Codex** (`~/.codex/config.toml`; export the token in your shell):

```toml
[mcp_servers.deployvera]
url = "https://mcp.deployvera.com/mcp"
bearer_token_env_var = "DEPLOYVERA_TOKEN"
```

**Windsurf and other clients**: any client that supports remote MCP over Streamable HTTP works. Use the URL `https://mcp.deployvera.com/mcp` and, for tokens, the header `Authorization: Bearer dvp_...`. For stdio-only clients use `npx -y deployvera-mcp` with the `DEPLOYVERA_TOKEN` environment variable.

## Tools

| Tool | Scope | What it does |
|------|-------|--------------|
| `list_services` | read | Your own services with plan, status, app quota, apps used and free slots. Shared services are not included |
| `list_repositories` | read | Repositories you can deploy from connected GitHub/GitLab (and Gitea where enabled): provider, full name, URL, default branch, private flag. No tokens |
| `list_projects` | read | Projects with status, live URL, repo/branch, last build |
| `get_project` | read | Details, latest build, and whether the live commit is behind the branch |
| `check_domain` | read | Whether a custom domain is attached and whether each name currently points to the project (live DNS lookup) |
| `get_build` | read | Build status (latest by default); `wait_seconds` (0-120) polls every 5 s until it ends |
| `list_databases` | read | Managed databases (id, name, engine, status, project). Never connection details or credentials |
| `get_database_logs` | read | Tail (default 100, max 500) of a managed database's sanitized operational log |
| `get_build_logs` | read | Tail (default 200, max 1000) of the sanitized build log, or `source: "runtime"` |
| `explain_build_failure` | read | Reason code, plain-English cause, concrete fix steps and the last ~40 log lines |
| `list_env_vars` | env | Keys and flags only (build-time, multiline). Values are never returned |
| `set_env_vars` | env | Bulk set `{KEY: value}`; optional `build_time`, `apply` (restart). Values are never echoed |
| `delete_env_var` | env | Delete by key; always needs approval, optional `approval_id` |
| `deploy` | deploy | Deploy the branch head, or a specific `commit_sha`; returns the build id; optional `approval_id` |
| `rollback` | deploy | Redeploy the retained image of an earlier build; always needs approval, optional `approval_id` |
| `cancel_build` | deploy | Cancel a queued or running build |
| `create_project` | deploy | Create a project in one of your own services from a Git repository (`service_id`, `repo_url`, `source`, `branch`; optional `name`, `port`, `build_method`, `root_directory`); always needs approval. No env values: use `set_env_vars` afterwards |
| `start_domain_verification` | deploy | Returns the DNS records (TXT ownership challenge plus CNAME/A) the user must create for a custom domain; changes no routing, no approval |
| `set_domain` | deploy | Attaches the domain once DNS is verified (otherwise an error to retry after propagation); always needs approval |
| `remove_domain` | deploy | Detaches the custom domain; always needs approval |
| `check_approval` | read | Status of an approval request (pending, approved, denied, expired, used) |
| `report_to_deployvera` | support | Send a detailed issue or product suggestion to the DeployVera support inbox, optionally with a project and related links |

New project and domain: `list_services` (a service with a free slot; ordering a plan stays in the panel) -> `list_repositories` -> `create_project` (approval) -> `set_env_vars` -> `deploy` / `get_build` -> `start_domain_verification` -> create the DNS records -> `set_domain` (approval) -> `check_domain`.

Typical flow after pushing code: `deploy`, then `get_build` with `wait_seconds=120`. If it fails: `explain_build_failure`, fix, push, repeat.

## Example prompts

- "Deploy my latest commit and tell me if it fails."
- "Why did my last build fail? Fix it and redeploy."
- "Which of my projects is behind its branch?"
- "Add DATABASE_URL to the api project and restart it." (give the value only if you are comfortable sharing it with the agent)
- "Show the last 100 runtime log lines of my site."
- "Are my databases running? Show the Postgres log if one is not."

## Approvals

Some actions need your approval in the browser, whatever the token or login scopes:

- `create_project`, `set_domain`, `remove_domain`, `rollback` and `delete_env_var` always require it.
- Optionally, **approve every deploy** can be switched on for a token or login, so `deploy` requires it too.

When approval is needed, the agent shows you a link that opens on `https://deployvera.com/approve/…`. Check that the address bar shows `deployvera.com` before you enter your password. Then check what will happen and confirm with your password and second factor. Requests expire after 10 minutes, and each approval works for one action only.

## Security

- Scopes: `read`, `deploy`, `env`, `support`. A token only does what its scopes allow. Reporting an issue or suggestion requires the separate `support` scope.
- Environment variable values are write-only: masked in `list_env_vars`, never echoed by `set_env_vars`, scrubbed from error text. A value you type into a prompt is visible to that agent.
- Tokens and browser logins cannot buy or change plans, make payments, change billing or account settings, or reach admin routes. Creating a project and connecting a domain are possible but always need browser approval.
- Tokens expire after 30, 90 or 365 days and can be revoked at any time in **Settings > AI agent access**.
- Rate limit: 120 requests per minute.
- The hosted server is stateless and does not store tokens. Keep tokens out of repositories; prefer an environment variable or your client's secret prompt over a literal value in a config file.

## Links

- Website: <https://deployvera.com>
- Panel: <https://deployvera.com/clientarea>
- Documentation: <https://deployvera.com/docs/en/quickstart>
- Privacy policy: <https://deployvera.com/legal/privacy-policy>
- Terms of service: <https://deployvera.com/legal/terms-of-service>
- Support: <support@deployvera.com>

## License

MIT. See [LICENSE](LICENSE). Logo: DeployVera mark, all rights reserved by DeployVera.
