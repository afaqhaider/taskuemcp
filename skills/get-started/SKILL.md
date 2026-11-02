---
name: get-started
description: Set up or use Taskue, including installing the bundled MCP connector, signing in with OAuth, troubleshooting connection problems, and managing tasks, shifts, staff, schedules or reports.
---

# Taskue

Taskue plans shifts, assigns tasks to staff and collects shift reports. This
connector is for Taskue administrators and owners only.

## Connection setup

The plugin bundles the Taskue remote MCP server. Installing the plugin should
register the connector automatically, but OAuth always requires the user to
approve access in their browser.

Before calling a Taskue tool, check whether Taskue tools are available:

- If `get_connection_status` is available, call it first.
- If Taskue tools are unavailable, explain that the connector has not loaded.
  Ask the user to update or reinstall the Taskue plugin, restart Claude, and run
  `/mcp`. As a Claude Code fallback, provide:

  ```text
  claude mcp add --transport http --scope user taskue https://taskue-mcp-lmo7mkw3jq-uc.a.run.app/mcp
  ```

- If the connector exists but is not authenticated, tell the user to open
  `/mcp`, select Taskue, choose **Connect**, sign in using an active Taskue admin
  or owner account, and authorize access. Then retry their original request.
- Never ask the user to paste their Taskue password, OAuth token, Firebase
  credential, or API key into the conversation.
- Staff accounts cannot authorize this connector. If Taskue refuses the login,
  explain that an active admin or owner account is required.

Use these user-facing recovery messages:

- Missing tools: "Taskue is installed, but its connector has not loaded. Update
  or reinstall the Taskue plugin, restart Claude, then run /mcp and connect
  Taskue."
- Not authenticated or `401`: "Taskue needs to be connected. Run /mcp, select
  Taskue, choose Connect, sign in, and authorize access."
- Wrong role: "Taskue MCP access is available to active administrators and
  owners. Staff accounts cannot authorize the connector."
- Rate limited or `429`: "Taskue is temporarily busy. Wait a moment and retry."

## Workflow

1. Follow **Connection setup**, then call `get_connection_status` (or
   `get_profile`) to confirm who is connected. Never assume access to any
   company other than the one returned.
2. Read before writing: `get_dashboard`, `list_staff`, `list_tasks`,
   `list_shifts`, `get_schedule`, `list_reports`. Dates are `YYYY-MM-DD`.
3. For any change (`create_task`, `update_task`, `set_task_active`,
   `create_shift`, `update_shift`, `set_shift_active`):
   - Call the tool WITHOUT `confirm` first. It returns a preview and writes nothing.
   - Show the user exactly what will change and wait for explicit approval.
   - Only then call again with `confirm: true`.
4. Report what was applied, including the returned id.

## Switching account

If the user wants a different Taskue account or says the wrong account is connected, call
`disconnect_connection` (preview first, then `confirm: true` after they agree). The next
Taskue request then asks them to sign in again and they can choose another account.

## Rules

- There is no delete. Use `set_task_active` / `set_shift_active` with `isActive: false` to archive. If asked to delete, explain this and offer to archive.
- Recurring and master shifts, closed shifts, staff accounts, billing and submitted reports cannot be changed here; direct the user to the Taskue app.
- Assign only staff returned by `list_staff`; never guess staff ids.
- Do not batch many writes without the user approving the list first.
