---
name: deploy-to-deployvera
description: Deploy the current project to DeployVera, create the project if it does not exist yet, follow the build, diagnose and fix failures, and connect a custom domain. Use when the user asks to deploy, ship, redeploy, create a DeployVera project, check a build, read build logs, roll back, manage environment variables, or connect a domain on DeployVera.
---

# Deploy to DeployVera

Use the `deployvera` MCP tools. The user signs in to DeployVera in the browser when the server is first used; the connection needs the `read` permission, and `deploy` / `env` for the matching tools.

## Workflow

1. **Find the project.** Call `list_projects`, then `get_project` for the one that matches this repository (repo URL and branch). If none matches, create it: call `list_services` to find one of the user's plans with a free app slot and `list_repositories` to get the exact repository URL, then `create_project` (it always needs the user's browser approval). If no plan has a free slot, stop and tell the user to order a plan in the DeployVera panel (deployvera.com/clientarea); plans, payments, billing and account settings are never changed through these tools.
2. **Check the state.** `get_project` shows the latest build and whether the live commit is behind the branch. If it is not behind and nothing changed, say so instead of redeploying.
3. **Commit and push.** Make sure the working tree is committed and pushed to the project's branch. Do not commit secrets or `.env` files.
4. **Deploy.** Call `deploy` (optionally with `commit_sha`). Keep the returned build id.
5. **Wait.** Call `get_build` with `wait_seconds=120`. Repeat until the build reaches a final status.
6. **On failure.** Call `explain_build_failure`, read the cause and fix steps, and use `get_build_logs` only if more detail is needed. Fix the code or config, commit, push, and `deploy` again. If a variable is missing, add it with `set_env_vars` (see rules). Stop and ask the user after two failed attempts with the same cause.
7. **Custom domain (only if asked).** Call `start_domain_verification` and show the user the DNS records to create. After they say the records are in place, call `set_domain` (browser approval required); if DNS is not ready yet, wait and retry. `check_domain` shows the status; `remove_domain` detaches it (approval required).
8. **Report.** Give the final status and the live URL from `get_project`. If the build failed, say what was tried and what is left.

## Rules

- Never ask the user to paste secrets into chat when an environment variable can hold them. Ask them to set the value in the DeployVera panel, or use `set_env_vars` only with a value they have already given for that purpose.
- Never print, log or echo secret values. `list_env_vars` returns keys only; values are write-only.
- Confirm with the user before `create_project`, `set_domain`, `remove_domain`, `rollback` and `delete_env_var`. State what will change (build or key name) and wait for a yes.
- If a tool answers that approval is required, show the user the approval link, wait for them to approve it in the browser, then repeat the same call (pass `approval_id` if the tool accepts it). Never try to work around an approval.
- Use `cancel_build` only when the user asks or a build is clearly stuck on an outdated commit.
- Use `set_env_vars` with `apply` only when a restart is intended; mention that it restarts the app.
- Do not invent project ids, build ids or URLs. Take them from tool output.
- Rate limit is 120 requests per minute per connection: prefer one long `get_build` wait over many short polls.
