# Data, backups, and file formats

[Documentation home](README.md) · [User guide](UserGuide.md) · [Development](Development.md)

This reference describes the models and persistence rules in the current source, checked on 5 October 2026. Use the app to edit settings wherever possible. A valid JSON document is not necessarily a valid workspace: Deylo also validates identities, limits, integration settings, and geometry.

## Where information is stored

| Information | Storage | Included in a `.deylo` export? |
| --- | --- | --- |
| Workspace names, apps, websites, terminal commands, playlist selection, layouts, schedules, and delays | Local `workspaces.json` | Yes, for the selected workspace |
| Spotify refresh credential and associated client ID | macOS Keychain, through `SpotifyKeychainStore` | No |
| Spotify access token and loaded playlist list | Account manager memory | No |
| Configured Spotify client ID | UserDefaults preference `SpotifyClientID`; an optional app Info key can supply a default | No separate account configuration; the workspace only contains its playlist selection |
| Pro ownership | Verified StoreKit entitlements, reflected in manager memory | No |
| Development Pro preview | Memory only; resets on restart | No |
| Selected workspace, launch state, errors, consumed schedule minutes, and timers | Memory | No |

### Workspace path selection

`WorkspaceManager.defaultStorageURL()` first prepares `workspaces.json` in the process's Documents directory. It then checks whether the previous sandbox container already contains a workspace file for the running app's bundle identifier. If that file exists, it uses that existing file instead. This preserves workspaces when moving from a sandboxed build to the current desktop utility build.

For the current installation, the existing file is:

```text
~/Library/Containers/devplaceholder.LUODSYXH.macapp/Data/Documents/workspaces.json
```

That identifier is specific to this installation. The project expression is `devplaceholder.$(PROJECT_UNIQUE_VALUE:identifier).macapp`. A clean, unsandboxed build with no existing container file falls back to the user's Documents directory. A sandboxed build's Documents directory belongs to its container. Check the running build and existing file before attempting a manual restore; do not assume every installation uses the exact path above.

There is no cloud workspace synchronization, account-based workspace storage, automatic version history, or encrypted workspace-file format in the current implementation.

## Local workspace file

`workspaces.json` is a UTF-8 JSON array of `Workspace` objects. It has no archive wrapper or explicit schema-version field. `JSONEncoder` writes the array atomically after validation. Object-key order is not meaningful. App array order is meaningful for ordered launches; workspace array order controls the saved list and simultaneous schedule priority.

### Workspace fields

| Field | JSON type | Required to decode? | Meaning |
| --- | --- | --- | --- |
| `id` | UUID string | Yes | Workspace identity |
| `name` | String | Yes | Display name |
| `apps` | Array of app objects | Yes | Configured app entries, in display order; may be empty |
| `schedule` | Object or null | No | Local repeating schedule |
| `launchInOrder` | Boolean or null | No | Ordered launch preference; absent/null uses normal concurrent launching |

The UUID initializers in Swift create identities for new objects. They do not make `id` optional when decoding an existing file. Names need not be unique; identities must be unique within the local workspace list.

### App fields

| Field | JSON type | Required to decode? | Meaning/default |
| --- | --- | --- | --- |
| `id` | UUID string | Yes | Entry identity, unique within its workspace |
| `name` | String | Yes | Display name |
| `bundleId` | String | Yes | Installed application identifier; normal edits require a nonempty valid identifier |
| `frame` | Rectangle or null | No | Recorded window geometry |
| `automationCommand` | String or null | No | iTerm command |
| `url` | String or null | No | Browser website |
| `spotifyPlaylist` | String or null | No | Normalized Spotify playlist URL/URI; UI selection writes the playlist URI |
| `spotifyPlaylistName` | String or null | No | Playlist display label |
| `spotifyAutoplay` | Boolean or null | No | Missing/null means false |
| `windowPlacement` | String or null | No | Placement enum; compatibility rules below apply when absent/null |
| `launchDelaySeconds` | Integer or null | No | Wait before this entry in an ordered launch; missing/null means zero |

More than one entry can refer to the same installed application. Distinct entry identities are still required. Window operations select a usable window of the app, rather than permanently binding an entry to a particular document or window.

### Placement values and geometry

| Stored value | UI title |
| --- | --- |
| `none` | Leave unchanged |
| `leftHalf` | Left half |
| `rightHalf` | Right half |
| `topHalf` | Top half |
| `bottomHalf` | Bottom half |
| `maximize` | Fill display |
| `saved` | Recorded layout |

The Codable representation of a rectangle is `[[x, y], [width, height]]`, for example:

```json
[[100, 80], [1200, 800]]
```

Recorded frames use macOS Accessibility window coordinates. Presets convert from the main display's Cocoa usable frame, accounting for the primary screen's coordinate system. They do not represent fractions stored in the file. `saved` requires a recorded frame. Runtime checks also require the recorded target to remain reachable on connected displays; a structurally valid frame can still fail placement after a monitor is disconnected.

### Schedule fields

All four fields are required when a schedule object exists.

| Field | Type/range | Meaning |
| --- | --- | --- |
| `hour` | Integer, 0–23 | Local hour |
| `minute` | Integer, 0–59 | Local minute |
| `weekdays` | Nonempty array of integers, 1–7 | Calendar weekday set: Sunday=1, Monday=2, …, Saturday=7 |
| `isEnabled` | Boolean | Whether the schedule is enabled; execution also requires Pro access |

Weekday order is not meaningful because the model uses a set. No timezone is stored. Evaluation uses the current local calendar/timezone. There is no seconds field, catch-up queue, or persistent record of past runs. See [schedule behavior](UserGuide.md) for sleep, startup, simultaneous schedules, and clock changes.

## Portable `.deylo` file

A portable file contains one workspace inside this wrapper:

```json
{
  "format": "deylo.workspace",
  "version": 1,
  "workspace": {
    "id": "71F6E5D6-757C-4C45-AD2B-5C14B06D247F",
    "name": "Focus",
    "apps": [
      {
        "id": "AB24A8E4-7CDE-49AF-9994-44C3F1BED491",
        "name": "Safari",
        "bundleId": "com.apple.Safari",
        "url": "http://localhost:3000",
        "windowPlacement": "leftHalf",
        "launchDelaySeconds": 3
      }
    ],
    "launchInOrder": true,
    "schedule": {
      "hour": 9,
      "minute": 0,
      "weekdays": [2, 3, 4, 5, 6],
      "isEnabled": false
    }
  }
}
```

Exports use pretty-printed, sorted-key JSON. Local storage and exports are different formats: a raw `workspaces.json` array cannot be imported as a portable archive. An export is one workspace, not a complete backup of the account or all settings. See the [sample workspace](examples/Focus.deylo) for a three-app example.

### Import and copy rules

1. The reader requires a regular local file and bounded contents. Symlinks, directories, and pipes are rejected.
2. Decode checks the exact format marker, version, required fields, and structural validation.
3. Preview normalizes app settings and disables the schedule for review. It does not save or execute anything.
4. Confirmed import requires Pro access, creates fresh workspace/app UUIDs, and appends an independent copy to local storage.
5. The copied schedule stays disabled. The app selects the imported workspace so its settings can be reviewed.

Duplicate Workspace also creates fresh identities and disables the copied schedule, but works without Pro. It inserts the copy immediately after the original and chooses an available `Copy` name. Import keeps the archived name, even if another workspace already has that name. Neither operation changes the original.

Unknown JSON object keys are ignored by the synthesized decoder. Missing required fields, invalid enum values, wrong types, malformed UUIDs, and unsupported archive versions fail validation. Unknown optional keys are not preserved when a document is subsequently encoded. This is not a lossless editor for a newer format.

## Validation limits

These are processing safeguards, not paid workspace quotas.

| Item | Limit |
| --- | --- |
| Local file | 16,777,216 bytes (16 MiB; UI says 16 MB) |
| Workspace count in local storage | 50,000 |
| Apps in one locally stored workspace | 10,000 |
| Portable file | 1,048,576 bytes (1 MiB; UI says 1 MB) |
| Apps in a portable workspace | 100 |
| Workspace/app name | Local validation: nonempty, at most 200 characters, and no controls after trimming. Portable archives also require at most 200 characters before trimming. Normal UI saves trim names. |
| App bundle ID | At most 300 characters; normal edits allow ASCII letters, digits, `.` and `-` |
| Website value | At most 16,384 characters; browser-only HTTP/HTTPS validation also applies |
| Spotify playlist value | At most 2,048 characters; Spotify playlist validation also applies |
| Playlist display name | At most 1,000 characters |
| Terminal command | At most 32,768 characters; line breaks and tabs allowed, other hidden controls rejected |
| App delay | Integer from 0 through 30 seconds |
| Recorded frame | Finite values with absolute values at most 100,000; positive width and height |

Archives require valid, nonempty bundle identifiers. Local legacy data may retain blank bundle identifiers so the entries can be repaired in App Settings. An entry named Safari with a blank identifier is repaired in memory to `com.apple.Safari` when Safari is installed; a subsequent successful save persists that correction.

## Compatibility defaults

Older valid workspace files without Spotify, positioning, scheduling, or sequence additions remain readable:

- Missing `windowPlacement` plus an existing `frame` means Recorded layout.
- Missing `windowPlacement` without a frame means Leave unchanged.
- New entries explicitly opt out of placement until configured.
- Missing autoplay is false. Missing delay is zero. Missing schedule means no schedule. Missing ordered-launch flag means concurrent launching.
- Runtime Spotify launching can use a legacy playlist stored in `url` when no dedicated playlist value exists. New edits require the Spotify playlist field and enforce websites only on recognized browsers. Repair incompatible legacy fields in App Settings before export.

## Save safety and recovery

Before saving, the manager validates the entire workspace array and compares the current on-disk bytes with the bytes it last loaded or saved. If they differ, the app refuses to overwrite the file and asks for a restart/reload. This is a pre-write conflict check, not a file lock or a multi-process transaction. Do not run multiple app identities against the same workspace file.

Unreadable, invalid, oversized, and nonregular files are preserved. A failed reload retains already loaded workspaces in memory and prevents saving over the problematic file. Individual edits revert their in-memory changes when persistence fails. A missing file is treated as an empty first-run state. Writes are atomic; the app does not keep rotating backups or recover deleted items automatically.

For a manual full-file backup, quit Deylo and copy the actual local `workspaces.json` file to a safe location. To restore, quit the app, retain a copy of the current file, restore a known-good local array file to the same selected storage location, and reopen. A portable archive is restored with the app's reviewed import instead. [Troubleshooting](Troubleshooting.md) describes damaged files and access failures.

Workspace files contain plaintext URLs, playlist choices, recorded coordinates, and configured terminal commands. Those fields can contain private information supplied by the user. Inspect them before sharing a backup. Exports exclude Spotify credentials and StoreKit ownership, but do not scrub secrets that someone deliberately placed inside a URL or command.

## Source of truth

- `macapp/Models.swift`: fields, compatibility defaults, local structural validation.
- `macapp/WorkspaceArchive.swift`: wrapper, format version, archive validation, bounded reader.
- `macapp/WorkspaceSchedule.swift`: local time/weekday model.
- `macapp/WorkspaceManager.swift`: path selection, copy/import behavior, normalization, persistence, and rollback.
