# Deylo user guide

[Documentation home](README.md)

Deylo saves the applications, websites, Spotify playlist, and window positions you use together as a workspace. Choose a workspace and click **Launch Workspace**, or launch it from Deylo’s menu bar menu.

This guide describes the interface implemented in this repository. For building and running the app, see [Development](Development.md). For integration setup and platform requirements, see [Integrations](Integrations.md).

The current Xcode target is configured for macOS 27.0. Use the build requirements in [Development](Development.md) when running this checkout.

## Start with a workspace

1. Open Deylo.
2. Click **New Workspace** in the sidebar or toolbar. The initial empty screen also offers **Create Workspace**.
3. Enter a **Workspace name**, then click **Create Workspace**.
4. Click **Add App**. In **Add Application**, use **Choose…** to select an application, configure any options, and click **Add App**.
5. Add the remaining applications you want in this setup.
6. Click **Launch Workspace**.

New application entries leave window positions unchanged. Opening applications, websites, and Spotify playlists does not require window-control access. Automatic placement and **Record Layout** require macOS Accessibility access. Optional Spotify autoplay and iTerm commands use separate macOS Automation permissions.

Install the applications you intend to launch. The app chooser combines running applications with a list of common installed applications; it does not list every installed app. If an app is missing, open it and try the chooser again, or enter its details manually.

You can use ordinary workspaces without connecting Spotify or purchasing Pro. An empty workspace cannot be launched.

## Find your way around

The sidebar lists saved workspaces and their application counts. Select a workspace to see its setup on the right. A clock icon marks a workspace with an enabled schedule; without Pro access, its schedule is paused.

The selected workspace contains:

- **Launch Workspace** and an ellipsis menu for workspace actions.
- A **Schedule** card and a **Launch sequence** card. Their labels reflect saved settings and whether Pro access is available.
- An **Applications** section with **Add App** and a card for each application entry.
- A footer with the current status and **Record Layout**.

The toolbar’s **Permissions** button opens the **Window access** instructions. The sidebar’s **Deylo Pro** card opens purchase and restore controls; in development preview it is labeled **Pro preview**.

During a launch or placement test, editing controls are disabled. Wait for the operation to finish, or use **Stop Launch** when a workspace launch is in progress.

### Search

**Find a workspace** appears once there are at least six workspaces, and remains visible while a search is entered. It searches workspace names and the names of their applications, ignoring letter case. It does not search website addresses or commands.

Use the clear icon or **Clear Search** to show the whole list. Filtering the list does not delete workspaces or change the saved setup.

## Manage workspaces

Open the ellipsis beside **Launch Workspace**, or right-click a workspace in the sidebar, to access these actions:

| Action | Result |
| --- | --- |
| **Rename Workspace…** | Enter a new name and click **Save Name**. |
| **Duplicate Workspace** | Create and select an independent copy immediately after the original. |
| **Schedule…** | Configure a recurring launch, or remove an existing schedule. |
| **Launch Sequence…** | Configure whether applications open in order. |
| **Export Workspace…** | Save a portable `.deylo` backup with Pro. |
| **Delete Workspace…** | Confirm **Delete** to remove the workspace and its saved setup. |

Workspace names must contain 1–200 characters without line breaks. Leading and trailing whitespace is removed.

A duplicate is named with **Copy**, adding a number if necessary. It preserves the original’s application settings, layout, launch sequence, and delays. Its schedule is copied but disabled. Editing the copy does not change the original.

Deleting a workspace leaves your installed applications and files in place. It does not quit applications already opened.

New and imported workspaces are appended to the saved list. The current interface has no workspace reorder control. Application entries can be reordered within a workspace.

## Add, change, and remove applications

### Choose an application

Click **Add App**, then **Choose…**. Once an application has been chosen, the same menu is labeled **Change…**.

For an application outside the chooser, select **Other Application…**, or expand **Enter app details manually**. Fill in **Application name** and **Bundle identifier, e.g. com.apple.Safari**. A bundle identifier identifies the application; a file path or website address does not belong in this field.

Application names follow the same 1–200 character rule as workspace names. The save button remains disabled until both a name and bundle identifier are present. Invalid values produce an explanation in the sheet.

### Edit an entry

Click the sliders button on an application card, or choose **App Settings…** from its ellipsis menu. Click **Save Changes** when finished; **Cancel** discards the draft settings.

Changing the application clears its previously recorded window frame. Saving also removes options that do not apply to the new application: websites belong to recognized browsers, playlists and autoplay belong to Spotify, and commands belong to iTerm. Check the settings before saving the replacement.

Each card summarizes its launch action and placement. **Application unavailable · check App Settings** means Deylo cannot currently resolve that entry to an installed application. Use **Change…** or correct its bundle identifier.

### Reorder or remove an entry

Open the application card’s ellipsis menu:

- **Move Up** or **Move Down** moves the entry one position. The unavailable direction is disabled for the first or last entry.
- **Remove App…** opens a confirmation. Click **Remove** to delete this workspace entry, including its saved layout and launch options.

Removing an entry does not uninstall the application. Reordering is free and changes the displayed order; launch order follows that order when a Pro launch sequence is enabled.

## Open websites in a browser

For a recognized browser, **App Settings** includes an optional **Website** field. The address opens in that browser when the workspace launches. Leave it blank to launch the browser without opening a configured website.

Use a complete address such as `https://example.com` or `http://localhost:3000`. Deylo also normalizes host-only addresses: `example.com` uses HTTPS, while a loopback address such as `localhost:3000` uses HTTP unless you explicitly supplied a scheme. Only HTTP and HTTPS addresses are accepted. Addresses containing a username or password are rejected; sign in through the browser instead.

The source recognizes Safari, Safari Technology Preview, Dia, Arc, Chrome, Chrome Canary, Chromium, Edge, Firefox, Firefox Nightly, Brave, Vivaldi, Opera, and Opera GX by their bundle identifiers. A different application registering web links does not automatically gain the **Website** field. See [Integrations](Integrations.md) for the supported identifiers and URL handling.

There is one optional website per application entry. Deylo does not save or restore a browser’s complete tab session.

## Run an iTerm command

An iTerm entry includes an optional **Terminal command** field. Enter the shell text you want sent when that entry launches, then save it. The command runs in a new tab in iTerm’s current window; if iTerm has no window, Deylo creates one using its default profile.

macOS may ask for permission for Deylo to control iTerm. Commands are supported for iTerm only; Apple Terminal can be launched as an application but has no command field.

Deylo sends the configured text to the session. It does not monitor the command’s output, wait for a development server to become ready, or check that a long-running command finishes successfully. If a later application needs time before opening, use a Pro launch delay and see the sequence limitations below.

## Choose a Spotify playlist

Add or edit the Spotify desktop application to show its **Spotify** settings.

### Connect for playlist selection

1. Click **Connect Spotify**. If no Client ID is configured, Deylo opens the **Connect Spotify** setup sheet.
2. Use **Open Spotify Developer Dashboard** and follow the sheet’s instructions to create a developer app with the Web API enabled.
3. Add the exact redirect URI shown in Deylo: `macapp-spotify-auth://callback`.
4. Enter the app’s public, 32-character **Spotify Client ID**. A client secret is not used.
5. Click **Save & Connect** and complete Spotify sign-in.
6. In **Playlist**, choose a playlist and click **Add App** or **Save Changes**.

The setup sheet includes a note about Spotify developer eligibility and allowed users. Consult [Integrations](Integrations.md) for the external requirements and their sources, and [Troubleshooting](Troubleshooting.md) if sign-in or playlist loading fails.

When connected, the Spotify ellipsis menu offers **Refresh Playlists**, **Connection Settings…**, and **Disconnect**. If more than ten playlists are loaded, **Find a playlist** appears and searches playlist names. A selected playlist that is absent from the current list can remain visible as the previously selected playlist.

Choose **No playlist** to remove the selected playlist; this also turns autoplay off. Without a playlist, the Spotify entry simply launches the application.

### Open only or autoplay

**Automatically play this playlist** is off by default and is disabled when no playlist is selected.

- With autoplay off, Deylo opens the selected playlist without sending a playback command.
- With autoplay on, Deylo opens the playlist and asks the Spotify desktop app to start it. macOS Automation permission is required to control Spotify.

The Spotify account connection loads playlists; playback uses the local desktop application. Keep Spotify installed and signed in, with access to the chosen playlist. A playlist may open even if the separate autoplay request fails.

**Disconnect** removes Deylo’s local Spotify connection and loaded playlist list. It does not stop Spotify playback or clear the playlist saved in a workspace. An already saved playlist can still be opened without reconnecting; reconnect to browse and choose account playlists again. Connection credentials are kept separately from workspace backups. See [Data reference](DataReference.md) for storage details.

## Position windows

### Allow window control

Click the toolbar’s **Permissions** button, or **Set Up Access** in an application’s placement settings. In System Settings, enable Deylo under **Privacy & Security → Device Control and Data Access**; the interface also names **Accessibility** on earlier macOS and **macapp** for older builds.

Return to Deylo after granting access. It checks access when its window becomes active. If an access recovery message is still visible, click **Check Again**, then launch again.

A workspace containing any automatic placement will not begin launching until window access is available. To launch without positioning, turn **Position window automatically** off for each affected entry and save its settings.

### Choose a preset

In **App Settings**, enable **Position window automatically** and choose **Placement**:

| Placement | Behavior |
| --- | --- |
| **Left half** / **Right half** | Use the corresponding half of the main display’s usable area. |
| **Top half** / **Bottom half** | Use the corresponding half of the main display’s usable area. |
| **Fill display** | Fill the main display’s usable area. |
| **Recorded layout** | Restore the frame captured with **Record Layout**. Available when this entry has a recorded frame for the same application. |

With automatic placement off, the card shows **Leave unchanged** and Deylo keeps the current position. **Fill display** is a window-size placement, not a request to enter macOS full screen.

The usable area accounts for the menu bar and Dock. An application’s minimum window size can extend beyond a half-display preset. Window control targets a usable application window rather than recreating every open window. Open ordinary, non-minimized windows and leave full screen before testing or recording.

### Test a placement

With access enabled, click **Test Placement** in **App Settings**. The test positions the application’s currently open window and reports success or a recovery message. It does not launch the application or save the draft settings. Click **Save Changes** afterward if you want to keep the selected placement.

The test moves the actual window immediately. Choosing **Cancel** afterward discards the draft settings but does not undo that window movement.

### Record your layout

1. Open the applications in the workspace and arrange their normal windows.
2. Select the workspace in Deylo.
3. Click **Record Layout** in the footer.
4. Check the status, such as **Saved 3 of 4 window positions.**

Recording stores one usable window frame for each application entry it can find and changes those entries to **Recorded layout**. It does not open missing applications. Entries without a usable window keep their previous settings; if none can be recorded, Deylo explains how to retry.

Recorded layouts store window geometry rather than application documents, browser tabs, full-screen state, or a named monitor assignment. After changing or disconnecting displays, a recorded position may need to be captured again. See [Troubleshooting](Troubleshooting.md) for window and display failures.

## Launch a workspace

Click **Launch Workspace** in the main window, or **Launch [workspace name]** in the menu bar menu. Only one launch or placement test runs at a time.

By default, applications launch together. Each entry then performs its optional website, playlist, or iTerm action and applies its configured window placement. A problem with one application does not prevent the other entries from being attempted. If an optional integration fails, Deylo still attempts that entry’s window placement.

The footer reports **Launching…**, success, or the number of issues. Error details identify the affected entries. Use **Show Details** for longer messages and **Copy Details** to copy the explanation. Fix the relevant **App Settings** or access permission, then retry.

### Pro launch sequences

1. Open the **Launch sequence** card or **Launch Sequence…** from workspace options.
2. Enable **Launch apps in order** and click **Save**.
3. Reorder application cards with **Move Up** and **Move Down**.
4. In each application’s **App Settings → Launch timing**, choose a wait from 0–30 seconds and save.

During an ordered launch, Deylo waits before each entry, including the first entry if it has a delay. It completes that entry’s launch actions and placement attempt before moving to the next. Failures are reported and the sequence continues.

A delay is a fixed wait, not a readiness check. For example, launching iTerm with a server command and delaying a browser does not prove the server is listening. Deylo does not inspect that command’s output or health.

With **Launch apps in order** off, all applications launch together and saved delays have no effect. Cards show saved delays as paused when the sequence is inactive.

### Stop a launch

While a workspace launch is in progress, **Stop Launch** appears under the launch controls. It stops the remaining sequence entries and pending waits. Applications already opened stay open. It does not terminate running commands, stop music, close browser tabs, restore previous positions, or undo actions already requested from macOS.

In a concurrent launch, application launch requests may already have been sent before you click **Stop Launch**. Those applications can still open. The menu bar menu has no separate stop command; use **Open Deylo** to return to the main window.

## Schedule a workspace with Pro

1. Open the workspace’s **Schedule** card or **Schedule…** menu item.
2. In **Schedule workspace**, enable **Launch on a schedule**.
3. Set **Launch time** and select at least one day under **Repeat on**.
4. Click **Save Schedule**.

The initial schedule draft uses 09:00 on weekdays. Time and weekday labels use your Mac’s local calendar and time. A schedule repeats on its selected days; there is no one-off date field.

Turn **Launch on a schedule** off and save to keep an inactive schedule, or click **Remove Schedule** to remove it. Duplicated and imported schedules begin disabled; enable them explicitly after reviewing the new workspace.

Scheduled launches have these limits:

- Deylo must already be running and the Mac awake. Deylo does not wake the Mac or register an external background launch.
- Missed times are skipped. Opening Deylo or waking the Mac during a scheduled minute also skips that minute rather than catching up.
- A due schedule is skipped while another workspace is launching or a placement test is running. It is not queued for later.
- If several workspaces are due in the same minute, only the first in the saved list is attempted; the others are skipped. Choose distinct times, allowing enough time for each setup to finish.
- Scheduled launches use the same saved application settings, permission checks, and launch-sequence behavior as manual launches.
- Without Pro access, saved schedules do not run.

The scheduler checks periodically, so a scheduled launch need not start at the exact first second of the chosen minute. There is no catch-up queue or run-history screen. See [Testing](Testing.md) for automated scheduling checks and verification limits.

## Export and import a workspace with Pro

### Export

Choose **Export Workspace…** from workspace options, choose a location in **Export Workspace**, and save the suggested `.deylo` file. The suggested filename is based on the workspace name.

A backup contains one workspace’s application entries, websites, commands, playlists, window frames, schedule, launch sequence, and delays. It does not include installed applications, documents, browser sessions, Spotify sign-in credentials, or purchase records. Portable files support up to 100 application entries and 1 MB of data. See [Data reference](DataReference.md) for the format and validation rules.

URLs and commands can contain private details. Review the configuration before sharing a backup.

### Import and review

1. Click **Import Workspace…** in the sidebar.
2. Choose a regular workspace backup file in **Import a Deylo Workspace**.
3. Inspect **Review workspace import** before proceeding. The review shows each application’s name and bundle identifier, website, terminal command, Spotify playlist and autoplay, placement, recorded position, and any launch delay.
4. Click **Import Workspace** to add the workspace, or **Cancel** to leave your saved setup unchanged.

Import creates and selects an independent new workspace without replacing existing workspaces. Its schedule is disabled. Importing does not launch applications, open URLs, play music, or run commands.

Review commands and URLs before the first launch. Imported bundle identifiers must correspond to applications installed on the destination Mac. Recorded positions may need adjustment for its displays; Spotify connection and macOS permissions must be set up there separately. Deylo does not install missing applications or transfer accounts.

The file picker accepts files without depending on their extension; Deylo validates the actual archive contents and supported format version. A saved `workspaces.json` list is not a portable single-workspace archive. For an import error, **Choose Another File** reopens the picker; for an export error, **Choose a Destination** lets you retry saving.

## Menu bar and keyboard

Deylo’s menu bar menu offers **Open Deylo**, **Launch [workspace name]** for every saved workspace, a Pro entry, and **Quit Deylo**. Empty workspaces and all workspace launch items during an active launch or placement test are disabled.

The Pro entry is **Explore Deylo Pro…** until a purchase is verified, then **Deylo Pro — Unlocked**. The application’s **Workspace** menu also offers **Deylo Pro…**.

| Key | Where it applies |
| --- | --- |
| **Return** | Submits the enabled default action in workspace-name, application-settings, schedule, launch-options, Spotify-setup, and import-review sheets. |
| **Escape** | Cancels those sheets; also closes the Pro sheet through **Done** when enabled. |
| **Command-Q** | **Quit Deylo**. Quitting stops Deylo’s scheduling. |

The current app defines no dedicated keyboard shortcut for creating a workspace, launching a workspace, recording a layout, or stopping a launch.

## Free, Pro, and development preview

| Free | Deylo Pro |
| --- | --- |
| Unlimited workspaces; rename and duplicate | Scheduled launches |
| Add, edit, remove, and reorder application entries | Ordered launch sequences and 0–30 second per-entry delays |
| Application and website launching | Portable `.deylo` workspace export and import |
| Spotify playlist selection and optional autoplay | |
| Automatic window placement, placement tests, and recorded layouts | |

Open **Deylo Pro** to see the localized App Store price. **Unlock Pro · [price]** starts the one-time purchase. **Restore Purchases** checks for a purchase associated with the Apple Account. If the purchase is unavailable, use **Try Again** and read the displayed explanation. See [Commerce](Commerce.md) for development StoreKit configuration and distribution setup; a local test price is not a guaranteed live price.

When Pro ownership is not verified, the existing workspace data remains saved. Manual launches use the normal concurrent behavior, schedules pause, and saved delays do not apply. You can remove an existing schedule, turn an existing launch order off, and use **Clear Delay** in **App Settings**, then save. The current schedule sheet requires Pro to edit or save a schedule; **Remove Schedule** remains available without it.

In a Debug build, **Workspace → Preview Pro Features (Development)** enables a memory-only Pro preview. The sidebar displays **Pro preview**, and the Pro sheet states **Development preview. No purchase has been recorded.** Preview access resets when Deylo restarts and the menu item is absent in Release builds. Workspace settings and backups created during preview are real saved data; after preview is disabled, paid behavior follows the access rules above. Use [Development](Development.md), [Testing](Testing.md), and [Commerce](Commerce.md) for development and purchase-testing workflows.

## When something needs attention

The main window’s recovery panel shows the actionable error. **Show Details**, **Copy Details**, and the dismiss control help inspect or clear it. Window-access failures offer **Set Up Access** and **Check Again**; file-operation failures offer the retry actions described above.

Settings are saved as you confirm each operation; there is no global Save button. If saving fails, Deylo reports the problem and keeps the previous saved setup. If saved data cannot be read, Deylo protects it from being overwritten. Follow [Troubleshooting](Troubleshooting.md) before editing or replacing that file, and see [Data reference](DataReference.md) for its location and backup guidance.
