# Installing DeployVera MCP (for Cline)

DeployVera is a hosted MCP server. Nothing needs to be built or installed locally.

## 1. Add the remote server

Add this entry to the MCP settings (`cline_mcp_settings.json`):

```json
{
  "mcpServers": {
    "deployvera": {
      "url": "https://mcp.deployvera.com/mcp",
      "type": "streamableHttp",
      "disabled": false
    }
  }
}
```

The endpoint uses Streamable HTTP. On first use the client opens a browser window for OAuth sign-in (OAuth 2.1 with PKCE). Ask the user to complete the sign-in and approve access.

## 2. If the client cannot do OAuth

Ask the user to create a token in the DeployVera panel (https://deployvera.com/clientarea/settings > AI agent access). Tokens start with `dvp_`. Never invent or guess a token. Then add it as a header:

```json
{
  "mcpServers": {
    "deployvera": {
      "url": "https://mcp.deployvera.com/mcp",
      "type": "streamableHttp",
      "headers": { "Authorization": "Bearer dvp_REPLACE_WITH_USER_TOKEN" },
      "disabled": false
    }
  }
}
```

Alternative (stdio): `npx -y deployvera-mcp` with environment variable `DEPLOYVERA_TOKEN` set to the same token.

## 3. Verify

Call the `list_projects` tool. It should return the user's DeployVera projects (an empty list is valid for a new account).

## Notes

- `rollback` and `delete_env_var` always require the user's approval in the browser. When the tool returns an approval link, show it to the user, wait for approval, then call the same tool again with the returned `approval_id`. Do not try to approve on the user's behalf.
- After pushing code: call `deploy`, then `get_build` with `wait_seconds=120`; on failure call `explain_build_failure`.
- Support: support@deployvera.com
