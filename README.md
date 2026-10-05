# Deylo

**Your setup. One click.**

Deylo is a native macOS workspace launcher. Save the apps, websites, Spotify playlist, and window positions you use together, then launch the setup from the app or menu bar.

## Free and Pro

The existing app remains free: unlimited workspaces, app and website launching, connected Spotify playlist selection, optional autoplay, automatic window placement, recorded layouts, workspace renaming and duplication, and app reordering.

Deylo Pro adds three features through a single non-consumable purchase:

- **Scheduled launches:** choose a local time and repeat days. Deylo must already be open and the Mac awake. Missed times are skipped; use distinct times for different workspaces.
- **Launch sequences:** open apps in their displayed order, with an optional 0–30 second wait before each app. Stop cancels the remaining launches; apps already opened stay open. A delay is a fixed wait, not a check that a development server is ready.
- **Portable workspace backups:** export a `.deylo` file, review its launch actions, and import an independent copy. Import and duplication disable copied schedules. Importing never opens apps or runs commands. Backups include configured URLs and commands, but never Spotify sign-in credentials or purchase records.

Removing a schedule, turning an ordered launch off, and clearing a delay remain available without Pro. Without verified ownership, saved Pro schedules do not run and manual workspace launches use the normal concurrent behavior.

## Development

Open `macapp.xcodeproj` and run the `macapp` scheme. The built application is named Deylo. Its existing bundle identifier and Spotify callback identity are preserved so current workspace files and account connections remain associated with the same app.

Run `bash Tests/run-backend-tests.sh` for Debug regression checks, or `bash Tests/run-backend-tests.sh release` for Release checks. Both treat compiler warnings as failures and use temporary data and mocked system integrations. The tests never charge an account, open real apps, or run commands.

The refreshed interface adds workspace and app cards, placement previews, searchable workspace lists, focused settings, and contextual recovery messages. Read the [hardening report](docs/Hardening.md) for reproduced failures, fixes, and verification limits.

In a Debug build, **Workspace → Preview Pro Features (Development)** enables an explicitly labeled, memory-only preview of the Pro features. It resets on restart and is unavailable in Release builds.

## Commerce

The proposed local test price is **€24.99 once**. The live interface uses StoreKit's localized price. Read [commerce setup](docs/Commerce.md) for local StoreKit testing and the owner-managed App Store Connect steps required before real purchases are available.
