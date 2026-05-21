# Excalidraw Room CLI

`excalidraw-room` is a JSON-first CLI for reading, editing, and exporting shared Excalidraw rooms.

It is useful when you want a script or an AI agent to make precise changes to a live Excalidraw room without clicking around in the browser UI.

This fork includes self-hosted Excalidraw, ExcaliDash, fitted backgrounds, bindings, images, embeddables, and native frame support beyond the upstream project. See [FORK_CHANGES.md](FORK_CHANGES.md) for the full fork comparison.

## Requirements

- Bun 1.1+
- Node.js, used by the export runner
- Google Chrome or Chromium for PNG/SVG export

If Chrome is not in a standard location, set:

```bash
export EXCALIDRAW_ROOM_CHROME_BIN="/path/to/chrome"
```

## Install

From npm:

```bash
npm install -g excalidraw-room-cli@latest
```

From GitHub with Bun:

```bash
bun add -g excalidraw-room-cli
```

Check the install:

```bash
excalidraw-room help
excalidraw-room version
```

## Self-hosted Excalidraw

Configure a custom Excalidraw app URL once:

```bash
excalidraw-room config set-app-url https://excalidraw.example.com
```

Then `create-room` will return room URLs on that domain. You can also set it while creating a room; the URL is saved to the global CLI config:

```bash
excalidraw-room create-room --json --app-url https://excalidraw.example.com
```

If your deployment or collab endpoint is behind HTTP Basic Auth, configure credentials locally:

```bash
excalidraw-room config set-basic-auth "$EXCALIDRAW_USER" "$EXCALIDRAW_PASSWORD"
```

The password is stored only in `~/.excalidraw-room-cli/config.json` with file mode `0600`, and `config show` redacts it. To avoid shell history, set `EXCALIDRAW_ROOM_BASIC_AUTH_PASSWORD` and omit the password argument:

```bash
EXCALIDRAW_ROOM_BASIC_AUTH_PASSWORD='...' excalidraw-room config set-basic-auth "$EXCALIDRAW_USER"
```

If your self-hosted app uses a custom Socket.IO collaboration server, set it too:

```bash
excalidraw-room config set-ws-server-url https://collab.example.com
```

Per-command overrides are also supported and saved to the global config when provided:

```bash
excalidraw-room create-room --json --domain https://excalidraw.example.com
excalidraw-room apply-json "$ROOM_URL" --basic-auth "$EXCALIDRAW_USER:$EXCALIDRAW_PASSWORD" <<'JSON'
{ "command": "elements.add", "ops": [] }
JSON
```

Environment overrides are available for automation:

- `EXCALIDRAW_ROOM_APP_URL`
- `EXCALIDRAW_ROOM_WS_SERVER_URL`
- `EXCALIDRAW_ROOM_BASIC_AUTH=user:password`
- `EXCALIDRAW_ROOM_BASIC_AUTH_USER` and `EXCALIDRAW_ROOM_BASIC_AUTH_PASSWORD`

## ExcaliDash

The CLI can also target an ExcaliDash workspace instead of classic encrypted Excalidraw rooms:

```bash
excalidraw-room config set-backend excalidash
excalidraw-room config set-app-url https://hub.excalidraw.example.com
excalidraw-room config set-excalidash-auth "$EXCALIDASH_USER" "$EXCALIDASH_PASSWORD"
```

On the same host as ExcaliDash, you can keep public room URLs on the external domain while using a local API/socket path:

```bash
excalidraw-room config set-excalidash-api-url http://127.0.0.1:6767
```

In this mode `create-room --json` creates an ExcaliDash drawing and returns `/editor/<drawingId>`. `status`, `dump`, `snapshot`, `restore`, `send-file`, `apply-json`, and `export-image` work against ExcaliDash drawing URLs.

Environment overrides:

- `EXCALIDRAW_ROOM_BACKEND=excalidash`
- `EXCALIDRAW_ROOM_EXCALIDASH_API_URL`
- `EXCALIDRAW_ROOM_EXCALIDASH_AUTH=user:password`
- `EXCALIDRAW_ROOM_EXCALIDASH_USER` and `EXCALIDRAW_ROOM_EXCALIDASH_PASSWORD`

## Quick Start

Create a new shared room:

```bash
ROOM_URL="$(excalidraw-room create-room --json | bun -e 'const data = await Bun.stdin.json(); console.log(data.roomUrl)')"
```

Or use an existing Excalidraw shared room URL:

```bash
ROOM_URL="https://excalidraw.com/#room=...,..."
```

Inspect the room:

```bash
excalidraw-room status "$ROOM_URL"
excalidraw-room dump "$ROOM_URL" room.json
```

Add elements:

```bash
excalidraw-room apply-json "$ROOM_URL" <<'JSON'
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
    },
    {
      "type": "addArrow",
      "x1": 340,
      "y1": 150,
      "x2": 420,
      "y2": 150
    }
  ]
}
JSON
```

Export the room:

```bash
excalidraw-room export-image "$ROOM_URL" room.png
excalidraw-room export-image "$ROOM_URL" room.svg
```

## Commands

```bash
excalidraw-room help
excalidraw-room version
excalidraw-room config show
excalidraw-room create-room [--json] [--app-url <url>] [--ws-server-url <url>] [--basic-auth <user:password>]
excalidraw-room status <roomUrl>
excalidraw-room dump <roomUrl> [out.json]
excalidraw-room watch <roomUrl>
excalidraw-room snapshot <roomUrl> [out.json]
excalidraw-room restore <roomUrl> <snapshot.json>
excalidraw-room apply-json <roomUrl> [spec.json|-]
excalidraw-room send-file <roomUrl> <elements.json> [--mode append|replace]
excalidraw-room export-image <roomUrl> <out.png|out.svg> [options]
```

Agent integration commands:

```bash
excalidraw-room skill
excalidraw-room skills list
excalidraw-room skills get core
excalidraw-room setup --all-agents
```

## Version Check

```bash
excalidraw-room version
```

Prints the installed package version, checks the latest version in npm, and shows update commands when a newer version is available.

## Create Room

```bash
excalidraw-room create-room
```

Creates an empty shared Excalidraw room and prints:

- `roomId`
- `roomKey`
- `roomUrl`

The `roomUrl` uses the configured app URL. Set it globally with `config set-app-url` or for this command with `--app-url` / `--domain`.

Use `--json` when another tool needs to consume the result:

```bash
excalidraw-room create-room --json
```

Example output:

```json
{
  "roomId": "JUP-lk92luW3XjW16k3c",
  "roomKey": "m7D7utg2V8m2ptM-A2kHWg",
  "roomUrl": "https://excalidraw.com/#room=JUP-lk92luW3XjW16k3c,m7D7utg2V8m2ptM-A2kHWg"
}
```

## Apply JSON

`apply-json` is the main write command:

```bash
excalidraw-room apply-json <roomUrl> [spec.json|-]
```

Input can come from:

- a file: `apply-json <roomUrl> spec.json`
- stdin: `apply-json <roomUrl> -`
- stdin without `-`: `apply-json <roomUrl>`

### Format 1: Command

Add elements:

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
      "label": "Planner\nstep"
    }
  ]
}
```

Update existing elements:

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

Delete elements:

```json
{
  "command": "elements.delete",
  "ids": ["id-1", "id-2"]
}
```

Move elements behind or above others:

```json
{
  "command": "elements.reorder",
  "ids": ["background-id"],
  "position": "back"
}
```

Delete all live elements:

```json
{
  "command": "elements.delete",
  "all": true
}
```

### Format 2: Transaction

Full redraw is an explicit transaction. Deletion creates tombstones so open browser clients remove old elements.

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

### Format 3: Legacy Operation Array

```json
[
  {
    "type": "addRect",
    "x": 80,
    "y": 80,
    "width": 260,
    "height": 140,
    "backgroundColor": "#dbe4ff",
    "label": "Planner\nstep"
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

### Format 4: Legacy Mode and Operations

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
      "label": "Planner\nstep"
    }
  ]
}
```

Supported modes:

- `append`: add new elements without removing existing elements
- `replace`: replace the scene
- `patch`: update existing elements by id

Prefer command payloads for new work. Legacy `replace` remains available for compatibility.

### Format 5: Legacy Raw Elements

```json
{
  "mode": "replace",
  "elements": [
    {
      "id": "custom-id",
      "type": "rectangle"
    }
  ]
}
```

Command payloads can also add strict raw Excalidraw elements:

```json
{
  "command": "elements.add",
  "elements": []
}
```

Raw elements must be complete Excalidraw elements. `elements.add` fails when an incoming id already exists live.

## Operations

### `addRect`

```json
{
  "type": "addRect",
  "x": 80,
  "y": 80,
  "width": 260,
  "height": 140,
  "backgroundColor": "#dbe4ff",
  "strokeColor": "#1e1e1e",
  "label": "Agent\nentrypoint",
  "labelFontSize": 24,
  "labelColor": "#1e1e1e",
  "labelPosition": "center",
  "labelGap": 8,
  "bindLabel": true,
  "groupLabel": true,
  "layer": "front"
}
```

When `label` is set, the CLI binds the text to the rectangle as its container and puts both elements into one group by default. Set `bindLabel: false` or `groupLabel: false` only when you intentionally need a free-standing label.

To create a background or section frame around existing elements, use `fitToIds`. The rectangle is computed from the selected elements' bounding box plus `padding`; `layer: "back"` places the rectangle behind existing content while keeping its label visible.

```json
{
  "type": "addRect",
  "fitToIds": ["step-1-id", "step-2-id", "step-3-id"],
  "padding": 32,
  "backgroundColor": "#eef2ff",
  "strokeColor": "#6366f1",
  "label": "Основной поток",
  "labelPosition": "topLeft",
  "labelGap": 8,
  "layer": "back"
}
```

`labelPosition` accepts `center`, `topLeft`, `topCenter`, or `leftCenter`. For `topLeft` and `topCenter` with `fitToIds`, the CLI reserves extra top space for the label so it does not overlap the framed content; `labelGap` controls the extra gap below the label.

### `addText`

```json
{
  "type": "addText",
  "x": 220,
  "y": 240,
  "text": "Free note",
  "fontSize": 24,
  "strokeColor": "#1e1e1e",
  "textAlign": "center"
}
```

### `addImage`

```json
{
  "type": "addImage",
  "id": "logo-image",
  "x": 120,
  "y": 320,
  "width": 320,
  "height": 180,
  "path": "/tmp/logo.png"
}
```

Use either `path` or `dataURL`. In ExcaliDash mode the CLI stores the required Excalidraw `files[fileId]` entry alongside the image element, so `dump`, `snapshot`, `restore`, `send-file`, and `export-image` preserve image data.

### `addEmbeddable`

```json
{
  "type": "addEmbeddable",
  "id": "demo-embed",
  "x": 520,
  "y": 320,
  "width": 420,
  "height": 260,
  "link": "https://example.com"
}
```

Embeddable rendering depends on the Excalidraw host app's `validateEmbeddable` and `renderEmbeddable` configuration. Unsupported URLs still render as a linked embeddable placeholder.

### `addFrame`

```json
{
  "type": "addFrame",
  "id": "main-frame",
  "fitToIds": ["step-1-id", "step-2-id"],
  "padding": 40,
  "name": "Main flow"
}
```

`addFrame` creates a native Excalidraw frame and assigns `frameId` to the elements listed in `fitToIds`.

### `addArrow`

By coordinates:

```json
{
  "type": "addArrow",
  "x1": 340,
  "y1": 150,
  "x2": 420,
  "y2": 150,
  "strokeColor": "#1e1e1e"
}
```

Between existing elements:

```json
{
  "type": "addArrow",
  "fromId": "left-card-id",
  "toId": "right-card-id",
  "fromSide": "right",
  "toSide": "left",
  "fromOffset": 0.5,
  "toOffset": 0.5,
  "startGap": 4,
  "endGap": 4,
  "strokeColor": "#1e1e1e"
}
```

When `fromId` or `toId` is used, the CLI writes native Excalidraw element bindings so the arrow stays attached in the editor. `fromSide` and `toSide` accept `left`, `right`, `top`, `bottom`, or `center`; if omitted, the side is chosen from the relative element positions. Offsets are normalized from `0` to `1` along the selected side.

### `move`

```json
{
  "type": "move",
  "id": "element-id",
  "dx": 40,
  "dy": 20,
  "includeGroup": true
}
```

or:

```json
{
  "type": "move",
  "id": "element-id",
  "x": 600,
  "y": 320
}
```

`move` includes grouped elements by default, so a labeled rectangle moves together with its label. Set `includeGroup: false` to move only the target element.

### `delete`

```json
{
  "type": "delete",
  "ids": ["id-1", "id-2", "id-3"]
}
```

For new payloads, prefer the command form:

```json
{
  "command": "elements.delete",
  "ids": ["id-1", "id-2", "id-3"]
}
```

## Export

```bash
excalidraw-room export-image "$ROOM_URL" room.png
excalidraw-room export-image "$ROOM_URL" room.svg
excalidraw-room export-image "$ROOM_URL" crop.png --crop 80,360,1900,980
excalidraw-room export-image "$ROOM_URL" one.png --crop-element <id> --padding 40
excalidraw-room export-image "$ROOM_URL" group.png --crop-elements <id1,id2,id3> --padding 60
```

Export uses Excalidraw scene JSON and `@excalidraw/excalidraw`. It is not a browser screenshot.

Crop coordinates use Excalidraw scene coordinates.

## Agent Setup

Install the local discovery skill for supported agents:

```bash
excalidraw-room setup --all-agents
```

Or install it for one target:

```bash
excalidraw-room setup --claude
excalidraw-room setup --codex
excalidraw-room setup --cursor
excalidraw-room setup --universal
```

This writes a small discovery `SKILL.md` into standard local skill directories. The full agent workflow is available through:

```bash
excalidraw-room skill
```

## Local Files

The CLI stores runtime files under:

- snapshots: `~/.excalidraw-room-cli/snapshots`
- export cache: `~/.excalidraw-room-cli/cache`

## Compatibility

`apply-spec` is still accepted as a compatibility alias for `apply-json`. Prefer `apply-json` for new usage.
