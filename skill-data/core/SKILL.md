---
name: core
description: Core excalidraw-room usage guide. Read this before running excalidraw-room commands. Covers creating or reading a shared room, applying transactional add/update/delete commands via heredoc or file input, restoring snapshots, sending raw elements, and exporting PNG or SVG from scene JSON.
allowed-tools: Bash(excalidraw-room:*), Bash(npx -y excalidraw-room-cli:*), Bash(bunx excalidraw-room-cli:*)
---

# excalidraw-room core

JSON-first CLI for shared Excalidraw rooms.

Use this tool when the user wants to create, inspect, or modify a shared Excalidraw room through its room URL, not by driving the browser UI manually.

## The core loop

```bash
excalidraw-room version                     # 0. check installed vs latest version
excalidraw-room create-room --json          # 1. create a room if the user did not provide one
excalidraw-room status '<roomUrl>'          # 2. inspect current state
excalidraw-room dump '<roomUrl>'            # 3. read full scene if ids/layout matter
excalidraw-room apply-json '<roomUrl>'      # 4. apply one JSON payload via stdin
excalidraw-room export-image '<roomUrl>' /tmp/room.png   # 5. verify visually when needed
```

Prefer one `apply-json` payload per logical change. Use a `commands` transaction when a logical change needs multiple steps.

If `excalidraw-room skill` prints an update notice, tell the user a newer CLI exists before relying on version-sensitive behavior. Do not update the global CLI without explicit user approval.

## Self-hosted Excalidraw

If the user needs a self-hosted Excalidraw domain, configure it once:

```bash
excalidraw-room config set-app-url https://excalidraw.example.com
```

After that, `create-room --json` returns room URLs on that domain. A one-off command also works and saves the domain:

```bash
excalidraw-room create-room --json --app-url https://excalidraw.example.com
```

If the deployment requires HTTP Basic Auth, configure credentials locally. Prefer environment variables when the password should not be visible in shell history:

```bash
EXCALIDRAW_ROOM_BASIC_AUTH_PASSWORD='...' excalidraw-room config set-basic-auth "$EXCALIDRAW_USER"
```

If the self-hosted app uses a custom Socket.IO collab server:

```bash
excalidraw-room config set-ws-server-url https://collab.example.com
```

`config show` redacts the stored password. Do not put domains, room URLs, usernames, or passwords into committed files unless the user explicitly asks and the values are public.

## ExcaliDash backend

If the user wants project/folder storage through ExcaliDash, switch the CLI backend:

```bash
excalidraw-room config set-backend excalidash
excalidraw-room config set-app-url https://hub.excalidraw.example.com
EXCALIDRAW_ROOM_EXCALIDASH_PASSWORD='...' excalidraw-room config set-excalidash-auth "$EXCALIDASH_USER"
```

When running on the same host as ExcaliDash, set a local API URL while keeping public drawing URLs:

```bash
excalidraw-room config set-excalidash-api-url http://127.0.0.1:6767
```

ExcaliDash mode uses `/editor/<drawingId>` URLs. Use the same read/write commands as classic rooms.

## Create path

If the user asks you to make a new room or does not provide a room URL, use:

```bash
excalidraw-room create-room --json
```

Use the returned `roomUrl` for all later commands. `roomId` alone is not enough because the room key is required to decrypt and write the scene.

## Write path

Use `apply-json` as the main mutation path:

```bash
excalidraw-room apply-json '<roomUrl>' [spec.json|-]
```

Accepted inputs:

- file path: `apply-json <roomUrl> spec.json`
- stdin: `apply-json <roomUrl>`
- explicit stdin: `apply-json <roomUrl> -`

Use command payloads for new work. Legacy `mode` payloads are still accepted for compatibility.

## JSON formats

### Command: add

```json
{
  "command": "elements.add",
  "ops": [
    {
      "type": "addRect",
      "x": 80,
      "y": 80,
      "width": 260,
      "height": 140,
      "backgroundColor": "#dbe4ff",
      "label": "Agent\nentrypoint"
    }
  ]
}
```

### Command: update

```json
{
  "command": "elements.update",
  "updates": [
    {
      "id": "element-id",
      "set": {
        "text": "Updated label",
        "fontFamily": 2
      }
    }
  ]
}
```

### Command: delete

```json
{
  "command": "elements.delete",
  "ids": ["id-1", "id-2"]
}
```

Delete every live element:

```json
{
  "command": "elements.delete",
  "all": true
}
```

Move decorative backgrounds behind content:

```json
{
  "command": "elements.reorder",
  "ids": ["background-id"],
  "position": "back"
}
```

### Transaction

Use one transaction for full redraws so deletion and creation are applied as one write:

```json
{
  "commands": [
    { "command": "elements.delete", "all": true },
    {
      "command": "elements.add",
      "ops": [
        { "type": "addText", "x": 100, "y": 100, "text": "New scene" }
      ]
    }
  ]
}
```

Transactions are simulated in order and written once. If any command is invalid, no room update is written.

### Legacy array of operations

```json
[
  {
    "type": "addRect",
    "x": 80,
    "y": 80,
    "width": 260,
    "height": 140,
    "backgroundColor": "#dbe4ff",
    "label": "Agent\nentrypoint"
  },
  {
    "type": "addArrow",
    "x1": 340,
    "y1": 150,
    "x2": 420,
    "y2": 150
  }
]
```

### Legacy object with mode and ops

```json
{
  "mode": "append",
  "ops": [
    {
      "type": "addRect",
      "x": 80,
      "y": 80,
      "width": 260,
      "height": 140,
      "backgroundColor": "#dbe4ff",
      "label": "Agent\nentrypoint"
    }
  ]
}
```

### Legacy raw elements payload

```json
{
  "mode": "append",
  "elements": [
    {
      "id": "custom-id",
      "type": "rectangle"
    }
  ]
}
```

## Supported ops

- `addRect`
- `addText`
- `addArrow`
- `move` (legacy op)
- `delete` (legacy op)

## Rules

- Read before writing when existing ids or layout matter.
- For multiline text in JSON, use `\n`, not `\\n`.
- For labeled boxes, prefer `addRect` with `label`; by default the label is a bound container text and shares a group with the box.
- For arrows between blocks, prefer `addArrow` with `fromId` and `toId`; this writes native Excalidraw bindings. Use `fromSide`/`toSide` (`left`, `right`, `top`, `bottom`, `center`) and `fromOffset`/`toOffset` (`0` to `1`) when the attachment point matters.
- `move` includes grouped elements by default, so moving a labeled box also moves its label. Use `includeGroup: false` only for a single-element move.
- `elements.delete` creates tombstones so open clients see deletions immediately.
- Full redraw is `elements.delete all` plus `elements.add` in one `commands` transaction.
- `elements.update` fails if an id does not exist.
- `elements.add` with raw elements fails if an id already exists live.
- Do not re-add an id deleted earlier in the same transaction.
- Legacy `replace` and `restore` create tombstones for old live elements.

## Examples

### Heredoc patch

```bash
excalidraw-room apply-json '<roomUrl>' <<'JSON'
{
  "command": "elements.add",
  "ops": [
    { "type": "addRect", "x": 80, "y": 80, "width": 260, "height": 140, "backgroundColor": "#dbe4ff", "label": "Agent\nentrypoint" },
    { "type": "addArrow", "x1": 340, "y1": 150, "x2": 420, "y2": 150 }
  ]
}
JSON
```

### Snapshot and restore

```bash
excalidraw-room snapshot '<roomUrl>'
excalidraw-room restore '<roomUrl>' ~/.excalidraw-room-cli/snapshots/example.json
```

### Export

```bash
excalidraw-room export-image '<roomUrl>' /tmp/room.png
excalidraw-room export-image '<roomUrl>' /tmp/crop.png --crop 80,360,1900,980
excalidraw-room export-image '<roomUrl>' /tmp/one.png --crop-element <id> --padding 40
```

## Skill commands

```bash
excalidraw-room version
excalidraw-room config show
excalidraw-room create-room --json
excalidraw-room skills list
excalidraw-room skills get core
excalidraw-room setup --all-agents
```

`setup` writes the discovery stub into standard local skill directories so agent tooling can discover this CLI as a skill.

`version` checks the installed package version against npm latest and prints update commands when a newer version is available.
