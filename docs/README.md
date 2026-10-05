# Deylo documentation

**Your setup. One click.**

Deylo is a native macOS workspace launcher. A workspace groups installed applications with optional browser websites, Spotify playlists, iTerm commands, and window positions. Launch it from the main window or menu bar. Deylo Pro adds scheduled launches, ordered launches with per-app delays, and reviewed portable workspace files through a single non-consumable purchase.

This documentation covers the source and interface checked on **5 October 2026**. The current project targets **macOS 27.0**, uses the **macapp** Xcode scheme, and builds **Deylo.app**. Read the development reference before distributing a build: the current desktop target has App Sandbox disabled and live commerce still needs owner setup.

## Repository contents

This repository currently contains the documentation, `Deylo.storekit`, and a zipped Xcode project. The archive contains project metadata; the Swift application and regression-test source trees are still in the development workspace. Source references in these guides name paths in that full workspace. Build and test commands require those source trees in addition to the project.

## Choose a guide

| Guide | What it covers |
| --- | --- |
| [User guide](UserGuide.md) | First workspace, every current UI workflow, apps/websites/playlists/commands, placements, recording/testing, schedules, sequences/Stop, menu bar, keyboard, free/Pro |
| [Integrations and permissions](Integrations.md) | Supported browser identifiers, URL normalization, Spotify developer/OAuth setup, playlist access/autoplay, iTerm, Accessibility and display behavior |
| [Troubleshooting](Troubleshooting.md) | Recovery for launch, website, Spotify callback/account, permissions, displays, schedules, Pro, import/export, and saved-data failures |
| [Data, backups, and file formats](DataReference.md) | Actual storage paths, privacy boundaries, JSON schema/defaults, `.deylo` format/version, limits, import/copy behavior, migration, backup/restore |
| [Development and architecture](Development.md) | Toolchain/build commands, source map, state ownership, launch/schedule lifecycle, internal APIs, dependency injection, signing/identity, release workflow |
| [Testing and verification](Testing.md) | Regression commands, suite map, native acceptance checks, fixture isolation, artifact checks, documented evidence and limits |
| [Pro commerce setup](Commerce.md) | Product ID, verified ownership, one-time billing behavior, local StoreKit configuration, owner setup and distribution prerequisites |
| [Hardening report](Hardening.md) | Dated reproduced bugs, fixes, regression totals, and completed native checks |
| [Example portable workspace](examples/Focus.deylo) | Valid three-app archive for Visual Studio Code, Safari localhost, and Spotify; copied schedule starts off |

## Quick start

1. Open Deylo and choose **New Workspace**.
2. Name the setup and choose **Add App** for each installed application.
3. Configure **Website** for a recognized browser, a connected account's **Playlist** for Spotify, or **Terminal command** for iTerm as needed.
4. If you want positioning, enable **Position window automatically**, choose a placement, and allow Deylo's window-control permission in System Settings.
5. Save the settings and choose **Launch Workspace**.

The [user guide](UserGuide.md) walks through those steps and every additional control. The sample workspace imports without executing anything; starting its localhost URL still requires a server already listening, and choosing a Spotify playlist still requires configuring that entry.

## Feature availability

| Capability | Free | Pro |
| --- | --- | --- |
| Workspaces; rename/duplicate; app editing/reordering | Yes | Yes |
| App and selected-browser website launching | Yes | Yes |
| Spotify account playlist picker and optional local autoplay | Yes | Yes |
| iTerm commands | Yes | Yes |
| Window presets, placement test, and recorded layouts | Yes | Yes |
| Local recurring schedules | No | Yes |
| Ordered launches and 0–30 second per-entry delays | No | Yes |
| Portable `.deylo` export/import | No | Yes |

Ordinary workspace behavior remains available without Pro. Existing paid settings are retained but inactive when access is not verified. Clearing paid configuration remains possible, as detailed in the user guide. The Debug-only Pro preview is labeled, memory-only, and absent from Release. The local €24.99 test price is proposed; live prices come from StoreKit.

## Current operating limits

Schedules require Deylo running and the Mac awake; missed or overlapping times are skipped. Ordered launch delays are fixed waits. Deylo does not check server readiness, restore an entire browser session, install applications, reopen named project documents, synchronize configurations through a cloud account, or provide a background system scheduler.

Window control works with usable application windows and the connected display topology. Recorded layouts contain geometry, not named monitor assignments or every app window. Spotify account access is for playlist selection; autoplay is a separate desktop-app automation request.

Local workspace files are plaintext configuration. Portable files omit account credentials and purchases but include configured URLs and commands. See [DataReference](DataReference.md) before sharing or manually restoring files.

## Documentation maintenance

The Swift source and Xcode settings are the source of truth for shipped behavior. Provider requirements are linked to official Apple and Spotify documentation in the relevant chapters. Historical test results are dated; they are not a promise that every environment or future build has been verified. When code changes, update the affected workflow, validation, data-format, integration, and test sections together.

Return to the [project overview](../README.md).

