# Fork Changes

This document summarizes the changes in `ascorblack/excalidraw-room-cli` compared to the upstream repository `toolittlecakes/excalidraw-room-cli`.

Comparison baseline:

- Upstream remote: `upstream/main`
- Upstream commit at comparison time: `60aa183 feat: add transactional element command API`
- Fork commit at comparison time: `45cfc6e feat: support images embeds and frames`
- Fork-only commits: 7

## Fork-Only Commits

- `ab8f349 feat: support self-hosted excalidraw config`
- `54862e7 feat: add ExcaliDash backend support`
- `7d841e5 fix: support ExcaliDash element reordering`
- `822d44c fix: bind labels and arrows in ExcaliDash CLI`
- `ce4bf3f feat: add fitted background rectangles`
- `f1d9483 fix: reserve space for fitted frame labels`
- `45cfc6e feat: support images embeds and frames`

## Main Differences

### Self-hosted Excalidraw support

The fork adds persistent configuration for non-default Excalidraw deployments:

- Custom app URL via `config set-app-url`, `--app-url`, `--domain`, or `EXCALIDRAW_ROOM_APP_URL`.
- Custom Socket.IO collaboration endpoint via `config set-ws-server-url`, `--ws-server-url`, or `EXCALIDRAW_ROOM_WS_SERVER_URL`.
- HTTP Basic Auth support for protected self-hosted deployments.
- Redacted credential display in `config show`.
- File permissions hardened for the local CLI config file.

This lets generated room URLs and websocket traffic target a private/self-hosted Excalidraw installation instead of only the public Excalidraw service.

### ExcaliDash backend

The fork adds a second backend mode, `excalidash`, for working with an ExcaliDash workspace:

- Backend selection through `config set-backend excalidash`, `--backend excalidash`, or `EXCALIDRAW_ROOM_BACKEND=excalidash`.
- ExcaliDash API URL configuration, including separate public app URL and local API URL.
- ExcaliDash login configuration through username/email and password.
- `create-room --json` creates an ExcaliDash drawing and returns an `/editor/<drawingId>` URL.
- `status`, `dump`, `snapshot`, `restore`, `send-file`, `apply-json`, and `export-image` work against ExcaliDash drawing URLs.
- Scene updates are persisted through the ExcaliDash API and broadcast through the ExcaliDash socket where applicable.

Known limitation: `watch` is still not implemented for ExcaliDash. Use `status` or `dump` for inspection.

### Element ordering and scene replacement fixes

The fork improves operations that are sensitive to element order and live clients:

- `elements.reorder` works against ExcaliDash scenes.
- Replace/restore flows create tombstones for old live elements so open clients observe deletions.
- Reorder operations preserve deleted elements while changing the visible ordering of live elements.
- Raw element payloads, command payloads, and legacy operation arrays are normalized more consistently.

### Labels, groups, and arrow bindings

The fork fixes several visual correctness issues around generated diagrams:

- Rect labels can be bound to their containing rectangle.
- Rect labels can be grouped with their containing rectangle so move operations keep the visual unit together.
- Arrow endpoints created with `fromId`/`toId` now include Excalidraw-compatible `startBinding` and `endBinding`.
- Arrow binding supports side selection, offsets, and start/end gaps.
- Legacy `move` can include a grouped label through `includeGroup`.
- Generated bound text uses container metadata expected by modern Excalidraw scenes.

These changes are meant to reduce the need for manual reattachment or regrouping in the Excalidraw UI.

### Fitted background rectangles

The fork adds `fitToIds` support to `addRect` for generating section backgrounds around existing elements:

- The rectangle can be computed from the bounding box of existing elements.
- `padding` controls the surrounding spacing.
- `layer: "back"` places the background behind existing content.
- Label placement supports `center`, `topLeft`, `topCenter`, and `leftCenter`.
- Top labels reserve additional vertical space so the label does not overlap framed content.
- `labelGap` controls spacing between a top label and the framed content.

This is useful for creating visual groups such as "Main flow" backgrounds without manually calculating coordinates.

### Images, embeddables, and native frames

The fork adds support for more Excalidraw element types:

- `addImage` creates an `image` element from a local `path` or `dataURL`.
- Image file payloads are stored in Excalidraw `files[fileId]` data.
- PNG, JPEG, GIF, WebP, and SVG MIME types are detected from file extensions or data URLs.
- PNG, JPEG, and GIF dimensions are detected automatically when possible.
- `addEmbeddable` creates an `embeddable` element with an absolute URL.
- `addFrame` creates a native Excalidraw `frame` element.
- `addFrame` can use explicit coordinates or `fitToIds`.
- Elements listed in `addFrame.fitToIds` receive the generated frame's `frameId`.

Embeddable rendering still depends on the host Excalidraw app's embeddable validation/rendering policy.

### Scene files and export preservation

The upstream CLI mostly worked with element arrays. This fork carries complete scene data where needed:

- `dump` can write `elements`, `appState`, and `files`.
- `snapshot` stores complete scene data.
- `restore` preserves scene files.
- `send-file` preserves incoming scene files in append and replace modes.
- `apply-json` merges files produced by operations such as `addImage`.
- `export-image` passes `files` to Excalidraw export utilities so image elements render in PNG/SVG exports.

This fixes the common failure mode where an `image` element exists but its binary payload is missing, causing broken or placeholder visuals.

### Agent skill and README updates

The fork updates the built-in skill prompt and README to describe the new capabilities:

- Self-hosted Excalidraw configuration.
- ExcaliDash backend configuration.
- Fitted rectangles and frame/background guidance.
- Image, embeddable, and native frame operations.
- Cropped exports.
- Safer visual editing guidance for AI agents.

## Files Changed Versus Upstream

At the comparison point, the fork changes these tracked files relative to `upstream/main`:

- `src/index.ts`
- `scripts/export-runner.mjs`
- `README.md`
- `skill-data/core/SKILL.md`

Summary size:

- 1,942 insertions
- 155 deletions

## Verification Used During Fork Development

The fork changes were checked with:

- `npx tsc --noEmit`
- `git diff --check`
- `excalidraw-room help`
- `excalidraw-room skill`
- `excalidraw-room config show`
- `excalidraw-room create-room --json`
- `excalidraw-room status`
- `excalidraw-room dump`
- `excalidraw-room snapshot`
- `excalidraw-room restore`
- `excalidraw-room apply-json`
- `excalidraw-room send-file`
- `excalidraw-room export-image`

Manual smoke scenes were also used for:

- Bound labels and grouped movement.
- Arrow binding between elements.
- Fitted background rectangles.
- Native frames.
- Real PNG image insertion and export.
- Scene file preservation across dump/snapshot/restore/send-file/export.
