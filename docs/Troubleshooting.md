# Troubleshooting

[Documentation home](README.md)

Use the explanation shown by Deylo first: it identifies the failed action and usually gives the next step. This guide covers the current source's recovery paths. The [User Guide](UserGuide.md) explains normal workflows, [Integrations](Integrations.md) explains permissions and external apps, and [Data Reference](DataReference.md) identifies files and validation limits. For build failures see [Development](Development.md); for purchase setup see [Commerce](Commerce.md).

## Read and recover from a message

Workspace errors appear in the bottom recovery panel. Longer or multiline errors offer **Show Details** and **Less Detail**; **Copy Details** copies the displayed explanation. The close button dismisses the message. Dismissing does not change permissions, repair a file or repeat the operation.

| Where the problem appears | Available recovery |
| --- | --- |
| Window access blocks workspace launch | **Set Up Access** opens the macOS privacy settings; **Check Again** checks permission. Launch again after access is ready. |
| Choosing or reading an import file fails | **Choose Another File** opens a fresh file picker. |
| Export validation or writing fails | **Choose a Destination** retries export for that workspace. Validation errors still require fixing settings before another destination can help. |
| App or workspace settings cannot save | The sheet keeps its values and shows an inline error. Correct the cause and save again. |
| Spotify list cannot load while connected | **Try Again** or **Refresh Playlists** repeats loading. |
| Spotify is disconnected or sign-in fails | **Connection Settings…** lets you check setup and reconnect. |
| Pro price/product is unavailable | **Try Again** reloads ownership and the product. **Restore Purchases** performs an account sync. |

File pickers, settings sheets and active launches guard against overlapping operations. Some controls are disabled while a picker, launch, placement test, purchase or playlist load is active. Wait for the operation to finish. **Stop Launch** is available for workspace launching, including fixed delays; it is not offered for a placement test. It stops future launch entries without closing apps already opened or undoing commands already sent.

When a workspace finishes with issues, Deylo reports failures by app name and shows up to ten app explanations plus a count of any additional failures. Other apps can still have launched. Correct the affected entry before launching the whole workspace again: a second launch can reopen websites, add iTerm tabs and rerun commands.

The error formatter deliberately avoids raw response bodies, failing URLs, OS dumps and arbitrary system descriptions. A generic contextual message means the source has no more specific safe recovery mapping. Preserve the message and the steps that caused it; do not infer a cause from an omitted underlying error.

Source: recovery panels and file operations in `macapp/ContentView.swift`, launch aggregation in `macapp/WorkspaceManager.swift`, and `macapp/AppErrorMessage.swift`.

## App launching and settings

| Symptom or message | Recovery and interpretation |
| --- | --- |
| No installed app matches the bundle identifier | Open **App Settings**, choose the installed app again or correct its bundle identifier. If the app is absent, install it or remove the entry. |
| An app is missing from the chooser | Open it so the running-app list can include it, or use **Other Application…**. The chooser does not enumerate every installed app. |
| The app could not be opened | Open it manually to check installation and normal startup, then retry. macOS launch completion can succeed before a normal window becomes available. |
| Workspace or app could not be found | Reselect the workspace and reopen its settings. The action's referenced entry may no longer exist. |
| This app entry is already in the workspace | Edit the existing entry, or add a new entry with a new identity. This error concerns duplicate app-entry IDs, not two entries targeting the same application. |
| Name is empty or invalid | Use 1–200 characters after trimming, without control characters or line breaks. |
| Bundle identifier is invalid | Choose the app again. Nonempty identifiers are limited to 300 ASCII letters, digits, periods or hyphens. |
| An app setting is too long | Shorten its website, playlist name or command. Limits are listed in [Data Reference](DataReference.md). |
| Workspace contains too many apps, duplicate identifiers or an invalid schedule | Split the workspace, recover a valid backup, or correct the schedule. These are validation safeguards; the source does not silently repair arbitrary duplicate identities. |
| Wait for launch or placement test before changing settings | Finish the active operation, or use **Stop Launch** when offered. Editing is blocked to keep that operation's workspace snapshot consistent. |
| Already first/last in the workspace | The entry cannot move farther in that direction. Choose the other direction. |

The app permits existing legacy entries with blank bundle identifiers to load so they remain editable. A legacy entry named Safari is repaired in memory to `com.apple.Safari` only when Safari is installed. Other blank identifiers require choosing an app. Loading the legacy entry does not rewrite the stored file; a successful later save persists changes.

Source: `validatedApp`, editing operations and `resolveLegacySafariEntries` in `macapp/WorkspaceManager.swift`, and validation in `macapp/Models.swift`.

## Websites and browser routing

| Problem | Recovery |
| --- | --- |
| Website field is absent for the chosen app | Website capability uses a fixed browser allowlist. Check the supported identifiers in [Integrations](Integrations.md#application-selection-and-browser-routing). A generic HTTP handler or an unknown browser variant is not sufficient. |
| Website links are available for browsers only | Remove the website from that entry or select a supported browser. Use Spotify's separate playlist setting for Spotify. |
| Empty address or hidden control characters | Enter a normal HTTP/HTTPS address and remove pasted hidden characters. |
| Address is too long | Use at most 16,384 characters. |
| Use a complete HTTP or HTTPS address | Include `//` after an explicit `http:` or `https:`, for example `https://example.com`. |
| Website addresses must use HTTP/HTTPS | Replace `file:`, `javascript:`, `mailto:` and other schemes with a website address, or leave the website setting empty. |
| Invalid host or port | Check the hostname and use a numeric port from 1 to 65,535. |
| Address contains username/password | Remove URL credentials and sign in through the browser. |
| Local server opens with HTTPS unexpectedly | Use explicit `http://` unless the host is a recognized loopback address. Private network IPs and ordinary local hostnames do not automatically receive HTTP. |
| Website opens in a browser different from the default | The entry intentionally routes to its configured browser. Change that entry to change the destination. |
| Website opens but page fails to load | Check the address in that browser and confirm the server/network is ready. Deylo validates URL syntax and delivers the URL; it does not fetch the page or wait for a development server. |

The browser decides which window, profile and tab receives the URL. Deylo cannot restore a specific browser tab or choose a profile. See [Integrations](Integrations.md#website-normalization-and-validation) for normalization examples.

## iTerm commands

| Problem | Recovery |
| --- | --- |
| Command setting is absent for Apple Terminal or another app | Commands are supported only for iTerm (`com.googlecode.iterm2`). Choose iTerm, or launch the other terminal without a saved command. |
| Command is empty, too long or contains hidden control characters | Use a nonblank command of at most 32,768 characters. Newlines, carriage returns and tabs are allowed; other control characters are rejected. |
| macOS blocked iTerm automation | Under **System Settings → Privacy & Security → Automation**, allow the running Deylo build to control iTerm, then retry. [Apple's Automation guide](https://support.apple.com/guide/mac-help/allow-apps-to-automate-and-control-other-apps-mchl108e1718/mac). |
| Could not run the iTerm command | Open iTerm manually and check it works, then retry. Check the build's Apple-event configuration if the problem survives normal setup; see [Development](Development.md). |
| Command runs in an unexpected directory/profile | The script uses iTerm's default profile and current window. Put a `cd` and required environment setup in the command. Deylo has no separate working-directory or profile setting. |
| Launch succeeds but shell command fails | Inspect iTerm's output. Deylo does not inspect shell exit status; successful AppleScript delivery does not establish command success. |
| Stop Launch did not stop an already running command | Once the command has been written to iTerm, stop it in that terminal. Stop Launch prevents later entries from opening. |
| Later app opens before a server started by iTerm is ready | Ordered launch completion does not wait for shell output or server readiness. Use a suitable fixed delay or a readiness check within your own command/workflow. |

A new workspace launch can create a new iTerm tab and run the command again. Review imported commands before launching a portable workspace. [Integrations](Integrations.md#iterm-commands) describes the script and validation boundaries.

## Window control and layouts

### Enable access for the running build

1. Choose **Set Up Access** in the recovery panel or App Settings, or **Permissions → Open System Settings** in the main app.
2. Open **Privacy & Security** and the app-control permission pane. Deylo's copy calls this **Device Control and Data Access**, with **Accessibility** on earlier macOS versions. Enable Deylo, or `macapp` for an older build. Apple's guide documents adding the application when it does not appear. [Apple's Accessibility permission guide](https://support.apple.com/guide/mac-help/allow-accessibility-apps-to-access-your-mac-mh43185/mac).
3. Return to Deylo and choose **Check Again** if offered. A successful check reports that window control is ready.
4. Launch again or repeat the placement test. Granting access does not automatically replay the failed action.

Permission entries can refer to a different installed build. If access remains unavailable after rebuilding, verify the actual running application's entry and build identity. Do not change the bundle identifier as a routine permission fix: it also changes the Spotify Keychain service and legacy-container lookup. See [Development](Development.md).

Opening apps and websites alone needs no window-control access. If one app in a workspace has a preset or recorded layout, the entire workspace launch checks access first. To run without this permission, edit every affected entry and turn off **Position window automatically** so its placement is **Leave unchanged**.

### Layout error reference

| Message or failure | Recovery and limitation |
| --- | --- |
| Allow this app to manage window positions | Enable window-control permission for the running build, recheck and retry. |
| No window became available | Open a normal window in the target app and retry. Placement waits up to five seconds; it does not create documents or generic application windows. |
| No normal app window was available | Close a dialog or modal sheet and open a normal app window. Explicitly nonstandard windows are excluded. |
| Window is minimized | Restore it from the Dock, then retry. Deylo does not unminimize it. |
| Window is in full screen | Leave full screen, then retry. Deylo does not exit full screen for you. |
| macOS or the app did not allow the position/size | Recheck access, try another preset or **Leave unchanged**, and consider the app's minimum size. Both position and size must be settable. |
| Saved window position is invalid | Open the app window and record the layout again. Stored frames must be finite and have positive width and height. |
| No layout has been recorded / recorded layout is missing | Use **Record Layout** with an open eligible window, or select a preset. Changing the app in settings clears its former recorded frame. |
| Main display's usable area could not be determined | Reconnect/check the display and repeat the action. |
| Display arrangement changed during placement | Wait until the display arrangement is stable and try again. The preset aborts when the primary display's frame or usable area changes. |
| Recorded layout is outside connected displays | Reconnect the original display, choose a preset on the current primary display, or arrange the window and record it again. |
| No usable app windows were found when recording | Open normal windows in the workspace apps, leave full screen and restore minimized windows, then record again. |
| Only some positions were recorded | The reported count is intentional. Recording skips apps without usable windows and preserves their previous settings. |

**Test Placement** uses a currently open window; it does not launch the target app. It moves that window using the current sheet choice even before you save the setting. Save after a successful test if the placement should be used on future launches.

Presets always use the primary display's usable area, regardless of which display has keyboard focus. **Fill display** fills that area as a normal window, without entering full screen. An app's minimum size can make a half-placement overlap the other half. A preset succeeds only if the resulting size fits the usable area and is at least the requested size. A recorded layout requires a match within one point, so the same app's new minimum size can make an older recording fail.

Recorded coordinates can restore onto a connected secondary display, but no display identity, scaling rule or substitute-display mapping is saved. The source checks whether enough of the window's top edge remains reachable. It does not require the whole rectangle to be visible. Deylo manages one eligible window per app entry, preferring a focused normal window before other windows. It does not restore every document, browser tab or app window. See [Integrations](Integrations.md#window-controls-and-displays).

## Spotify account and playlist loading

Playlist browsing uses the account connection; opening and autoplay use the Spotify desktop app. Fix the failing path before disconnecting an otherwise working connection. [Integrations](Integrations.md#spotify) describes setup, scopes and privacy boundaries.

| Message or symptom | Recovery |
| --- | --- |
| Add/check the Client ID | In **Connection Settings…**, paste the public 32-character hexadecimal Client ID from the correct Spotify Developer app. A client secret is never needed. The committed project has no built-in Client ID. |
| Spotify rejected the Client ID | Check the selected developer app's Client ID and registered settings, then save and connect again. |
| Spotify sign-in could not open | Bring the main Deylo window forward and retry from it. The authentication session needs a presentation context. |
| Sign-in is already in progress | Finish or cancel the current browser sign-in before starting another. |
| Sign-in was cancelled | Start **Connect Spotify** again when ready. Cancellation is not a successful connection. |
| No sign-in code, unexpected callback, unreadable callback or sign-in could not be verified | Check that the developer app has exactly `macapp-spotify-auth://callback` registered, then start a fresh connection. Do not reuse or paste a callback URL. Validation rejects incorrect callback structure, missing/duplicate state or code and mismatched state. |
| Playlist access was not granted / Spotify did not grant playlist access | Connect again and approve private and collaborative playlist access. Both source-requested scopes are required. |
| Client ID changed | Connect again with the current ID. Changing an ID clears a current local connection instead of reusing its credential for another app. |
| Connection expired, revoked or no longer valid | Reconnect. Deylo already tries one refresh/retry after a playlist HTTP 401; repeated 401 or an invalid refresh grant clears the local connection. |
| Spotify no longer grants playlist access | Reconnect and consent to the required scopes. |
| HTTP 403 / denied playlist access | Check the developer app's allowed users and its owner's Premium subscription, then reconnect. A successful browser sign-in does not prove allowlist access. These current development-mode rules are documented by [Spotify quota modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes). |
| HTTP 429 / request limit reached | Wait the number of seconds displayed from a valid `Retry-After` header, or try later when none is available. Deylo does not automatically wait and retry a 429. |
| Other HTTP status | Retry after checking connectivity. The source displays the status number, not Spotify's raw response body. |
| Response could not be read, invalid playlist/page/token or response too large | Refresh the list for a playlist response; reconnect for a sign-in/token response. The parser rejects unsafe or inconsistent data. Persistent failures need a bug report with the safe message. |
| Could not prepare secure sign-in | Retry. Secure random verifier/state creation failed; no weaker sign-in fallback is used. |
| Saved connection could not be read | Use **Disconnect**, then connect again. Preserve workspaces; this error concerns the local account credential. |
| Login Keychain could not be accessed | Check that the login Keychain is unlocked and permits the running app to access its item, then retry. The error includes a Keychain status number. |
| Build signing configuration does not allow Keychain access | Use a build with the correct macOS signing configuration; see [Development](Development.md). This is distinct from a wrong Client ID or a Spotify allowlist denial. |
| Local credential could not be removed | Use **Disconnect** again after correcting Keychain access. Memory is already cleared, but deletion of the saved credential failed. |
| Connected indicator but list has not loaded | The indicator can initially reflect a saved local credential. Refresh performs the network validation. |
| No playlists were found | Save or create a playlist in Spotify, check which account signed in and refresh. An empty successful list is not a transport error. |
| No matching playlists | Choose **Clear Search** or change the local playlist-name search. The list search does not search Spotify's catalog. |
| Previously selected playlist appears instead of a list item | Deylo preserves that selection even when disconnected or when search/account results do not contain it. Reconnect to choose a different playlist, or choose **No playlist** to remove it. |

The API network client refuses redirects, enforces response bounds and uses 30-second request and 45-second resource timeouts. Network errors are mapped to connection, timeout, unreachable service or secure-connection recovery. There is no offline playlist catalog. A failed refresh can leave a prior in-memory list visible; check the error instead of treating the old list as a newly successful load.

**Disconnect** deletes only this app's local credential and clears its list and tokens in memory. It does not revoke Spotify's server consent, stop music or clear saved playlist settings in workspaces. The public Client ID preference remains. Do not post a Keychain item, access/refresh token, authorization code or full callback URL when requesting help.

Source: `macapp/SpotifyAccountManager.swift`, account controls in `macapp/ContentView.swift`.

## Spotify playlist opening and autoplay

| Problem | Recovery |
| --- | --- |
| Spotify is not installed / app could not be found | Install/open the Spotify desktop app, select it again if needed, or remove the playlist entry. Web API connection alone does not install a desktop player. |
| Playlist link or ID is invalid | Use a normal `open.spotify.com/playlist/<id>` link or `spotify:playlist:<id>` URI. Track, album and shortened URLs are not supported. Select a playlist from the connected picker for normal UI setup. |
| Add a playlist before enabling autoplay | Choose a playlist first; the UI disables autoplay for an empty selection. |
| Playlist playback is available for Spotify only | Remove Spotify-specific settings from the other app, or choose Spotify. |
| Could not open the playlist | Open Spotify manually, confirm the playlist works there, then retry. |
| macOS blocked Spotify autoplay (`-1743`) | Enable Deylo → Spotify under **Privacy & Security → Automation**, then retry. Alternatively turn autoplay off to send only the playlist link. [Apple's Automation guide](https://support.apple.com/guide/mac-help/allow-apps-to-automate-and-control-other-apps-mchl108e1718/mac). |
| Build does not have permission to control Spotify playback (`-10004`) | Use a build with Spotify playback access configured, or turn autoplay off. Reconnecting OAuth does not repair this build-level Apple-event restriction. |
| Spotify did not respond in time (`-1712`) | Open Spotify, ensure the playlist is available and retry, or turn autoplay off. The script timeout is 15 seconds. |
| Could not start the playlist | Check Spotify is signed in and can play that playlist, then retry. Turn autoplay off if opening alone is sufficient. |
| Autoplay is off but music continues | Deylo sends no stop/pause command when opening a link. Existing Spotify playback can continue. |
| Account is disconnected but a saved playlist opens | This is expected. Desktop opening and AppleScript playback do not require the OAuth credential used by the picker. |
| Deylo reports success but no audio is heard | Check Spotify's own playback state, account, content availability and output. Deylo confirms URI/AppleScript delivery, without verifying audible playback. |

## Saving, loading and preserving workspaces

Deylo normally uses `workspaces.json` in the user's Documents directory. If an existing legacy file is found at `~/Library/Containers/<bundle identifier>/Data/Documents/workspaces.json`, that file takes priority. Check [Data Reference](DataReference.md) before deciding which file is active. The startup file contains the full workspace array; it is a different format from a portable `.deylo` archive.

Edits save atomically. Ordinary add/edit/remove/rename/reorder/duplicate/import operations restore their previous in-memory state when saving fails. An inline or recovery message indicating an unsaved change should be resolved before relying on it. A failed save does not imply the displayed edit was committed to disk.

| Message or situation | Recovery and safeguard |
| --- | --- |
| No saved file on first launch | Deylo starts with an empty workspace list. This is expected. |
| Saved workspaces contain damaged or unsupported data | Keep a copy of the active file, recover a known-good storage backup and reopen Deylo. The failed startup read leaves the file unchanged and blocks overwriting it. |
| Saved data could not be read, so it was not overwritten | Resolve the original load problem before saving. Dismissing the message or creating a new workspace does not bypass the protection. |
| Saved file changed outside Deylo | Close and reopen Deylo to load the latest file, then make the change again. Deylo detects a byte-level difference from the last loaded/saved bytes and refuses to overwrite it. Concurrent app instances can trigger this safeguard. |
| Saved file exceeds 16 MiB / saved list too large | Preserve it, restore a smaller valid backup and reopen. The load is bounded. The source also limits the list to 50,000 workspaces and each workspace to 10,000 apps. |
| New save would exceed 16 MiB | Shorten large settings or remove unused entries, then retry. The existing file is not changed. |
| Duplicate workspace/app identities in storage | Recover a known-good backup with unique IDs. This is a structural data problem, not a workspace-name collision. |
| Deylo could not read/write its data folder | Correct access to the active data folder and reopen or retry. Account, file-picker and Accessibility permissions do not substitute for filesystem access. |
| File is damaged, unsupported, not regular or cannot be read | Keep a copy and use a valid ordinary file. The bounded reader refuses symlinks and nonregular items such as pipes or directories. |

There is no automatic damaged-file repair, backup rotation, merge of externally changed files or in-app “reset storage” operation. Reopening reloads the active file; it does not fix that file. If recovery requires a portable `.deylo` backup, preserve the damaged startup file and move it aside while Deylo is closed, reopen to an empty readable store, then use the normal Pro import workflow. A legacy file can take priority over the Documents file, so move/restore the actual active file. Never copy the `.deylo` envelope directly over `workspaces.json` or rename it as a storage repair.

Recovery from a startup validation failure may require restoring a known-good file outside the UI because editing/import cannot overwrite an unreadable startup store. Keep the original copy until the restored data is verified. The application does not silently replace it with an empty list.

Source: `defaultStorageURL`, `loadWorkspaces`, `saveWorkspaces` and editing rollback in `macapp/WorkspaceManager.swift`, `macapp/Models.swift`, and `WorkspaceFileReader` in `macapp/WorkspaceArchive.swift`.

## Importing and exporting portable workspaces

Import/export requires Pro access. Import uses a file picker and validates contents rather than trusting the filename extension. Previewing and importing do not launch apps, run commands or enable the copied schedule. Import creates a new workspace with new app identities; existing workspaces are preserved. Its review screen shows browser URLs, commands, playlist/autoplay choice, placement and delay so you can inspect the future actions.

| Problem | Recovery |
| --- | --- |
| Choose a file stored on this Mac / not a regular workspace file | Select an accessible ordinary exported backup through the picker. Copy a symlink target to a regular file if necessary. |
| Not a valid Deylo file / wrong format | Choose a `.deylo` backup exported by Deylo. An internal `workspaces.json`, another app's JSON or a renamed arbitrary file is not a portable archive. |
| Unsupported archive version | Update Deylo or export again using a compatible version. The current archive version is 1. |
| Backup exceeds 1 MiB | Choose or create a smaller backup. The limit is 1,048,576 bytes. |
| Portable workspace contains more than 100 apps | Split it into smaller workspaces before export. The portable limit is smaller than the internal storage limit. |
| Missing/excessively long settings, duplicate IDs, invalid frame or invalid schedule | Correct the original workspace in Deylo and export again, or use a known-good backup. Content validation happens before an import can be committed. |
| Website, playlist or command belongs to the wrong app type | Correct the source settings: websites for recognized browsers, playlists/autoplay for Spotify and commands for iTerm. Changing destination cannot fix validation. |
| Import cannot read the selected file | Choose an accessible local copy with **Choose Another File**. |
| Export cannot write the selected location | Use **Choose a Destination** and select a writable directory, such as Documents. |
| Import review stays open with a save error | Resolve the storage problem and retry **Import Workspace**. No new workspace is committed when saving fails. |
| Imported apps are missing on this Mac | Install them or change their app entries after importing. The archive is configuration, not an application installer. |
| Recorded layout no longer fits the connected displays | Reconnect the display or choose/record a current layout before launching. |
| Imported schedule did not launch | Imported schedules are intentionally disabled. Review and explicitly enable the schedule after import. |

Portable files contain URLs and commands, which can reveal private data and execute later when launched. They contain no Spotify sign-in credentials or purchase records. The recipient needs its own local Spotify connection to browse playlists and its own Pro access to import. See [Data Reference](DataReference.md) for the envelope and compatibility details.

## Shared file and network errors

The same system-error mappings are used across storage, file operations, Spotify and App Store actions. Their introduction names the failing context.

| Error category | Recovery shown by the app |
| --- | --- |
| User cancellation | Retry when ready. A cancelled picker normally closes without creating an error. |
| Offline, lost connection or data disallowed | Check internet connectivity and retry. |
| Timeout | Check connection and repeat after the service responds. |
| Host/DNS/connect failure | Check connection and try the service again later. |
| TLS/certificate/secure-connection failure | Check the Mac's date and network, then retry. |
| Service authentication required | Reconnect Spotify, or sign in to the relevant service. |
| Missing file | Choose an existing file; it may have moved or been removed. |
| Read/write permission | Correct saved-data access or choose an accessible import/writable export location. |
| Disk full | Free disk space and retry. |
| Read-only volume | Choose a writable location. |
| Destination name already exists | Choose another name or destination. |
| Corrupt or unsupported data | Keep a copy and use a valid backup. |
| Too-large file | Choose a smaller backup; observe the separate storage and archive bounds. |
| Invalid filename | Use a simple name and retry. |
| Unknown failure | Follow the contextual recovery; report reproducible steps if it persists. |

## Pro access, purchases and restore

The current product is the non-consumable `com.deylo.pro.lifetime`. Deylo uses StoreKit's localized product price. It unlocks ownership only for an exact, verified, active non-consumable entitlement. There is no preference-based production unlock. Product loading and ownership checks are separate, so a missing catalog product does not itself remove an existing verified entitlement. [Commerce](Commerce.md) explains implementation and release setup.

| Message or symptom | Recovery and interpretation |
| --- | --- |
| Purchase currently unavailable / no App Store price | Choose **Try Again** after checking the App Store connection. In a local development build, no live product may be configured. See [Commerce](Commerce.md) before treating this as an app bug. |
| Purchase cancelled | No new ownership was granted by that cancelled flow. Retry only if you want to purchase. |
| Purchase awaiting approval | Wait for the App Store to confirm. Deylo listens for transaction updates; pending approval does not unlock Pro early. |
| Purchase or update could not be verified | Choose **Restore Purchases**. Deylo deliberately does not grant access for an unverified result. |
| No active purchase after a purchase sheet/update | Use **Restore Purchases** to reconcile the account's current ownership. A newer refund or account change takes precedence over an older purchase result. |
| No purchase found for this Apple Account | Check the App Store is signed in to the account used for the purchase, then restore again. A different account or a local preview does not prove ownership. |
| Purchase window could not open | Bring Deylo to the front and retry. |
| Purchases restricted for this Apple Account/Mac | Review App Store account permissions before retrying. |
| App Store account notices | Open the App Store, review its notices and retry. |
| Product unavailable in current region | Try later; inspect product availability during commerce setup. |
| App Store cannot complete the operation on this Mac | Update macOS and Deylo, then retry as directed by the message. |
| Access removed by the App Store | The current verified entitlement no longer grants access, including revocation. Restore checks the account again; it does not override the store. |
| Pro preview vanished after restart | Expected in Debug. **Preview Pro Features (Development)** is memory-only, records no purchase and is absent in Release. |
| Release build has no development preview | Expected. Use valid StoreKit ownership; see [Commerce](Commerce.md) for local StoreKit test configuration. |
| Schedule/sequence is paused after loss of Pro | Saved settings remain, but schedules do not run and manual launch uses concurrent behavior without paid delays. Use **Remove Schedule**, turn launch sequence off, or clear a delay. The schedule sheet's toggle and Save action require Pro. |

**Restore Purchases** invokes explicit App Store sync and then reads current entitlements. It is not a local flag reset. A restored purchase, a cancelled purchase and an unavailable product have distinct messages. A proposed price in `Deylo.storekit` is for local testing; it does not make a live product available or charge an account. Debug Pro Preview is also separate from StoreKit's local test ownership.

The committed targets are desktop utility builds with App Sandbox disabled. Implemented StoreKit code does not establish App Store release readiness. Distribution signing, product configuration and release requirements remain owner-managed work documented in [Development](Development.md) and [Commerce](Commerce.md).

Source: `macapp/PremiumManager.swift`, `macapp/ProViews.swift`, StoreKit recovery in `macapp/AppErrorMessage.swift`.

## Schedules and launch sequences

| Situation | Expected behavior or recovery |
| --- | --- |
| Schedule did not run while Deylo was closed or Mac slept | Deylo must already be running and the Mac awake. Missed times are skipped; reopening/waking does not catch them up. |
| Schedule coincided with another launch or unavailable Pro | The due minute is consumed instead of queued for later. Run manually or choose another time. |
| Several workspaces share the same time | Only the first due workspace launches; all due workspaces consume that minute. Use distinct times. |
| Schedule does not run after importing or duplicating | Copies intentionally have schedules disabled. Enable after reviewing the copy. |
| Invalid schedule time/weekday | Choose a valid local time and at least one day. |
| Clock moved backward or long gap in evaluation | The scheduler skips catch-up evaluation rather than unexpectedly replaying workspaces. |
| Saved launch delay does nothing | Delays apply only to an ordered launch with Pro access. They are ignored in concurrent launches. |
| Sequence advances after an app failure | Expected. The error is collected and remaining entries continue. |
| Sequence cannot be edited while waiting | Use **Stop Launch**, then edit, or wait for completion. |

The scheduler is an in-process timer, checked about every 15 seconds. It does not create a macOS background agent, start Deylo at login or wake the Mac. See [User Guide](UserGuide.md) for setting up schedules and sequences, and [Data Reference](DataReference.md) for schedule data.

## Report a persistent issue

Include the safe **Copy Details** text, what you attempted, the affected app and settings type, whether the issue survives reopening, the Deylo build/configuration and macOS version, and relevant display arrangement or permission state. Separate “launch delivered” from “website/command/music worked” when describing the result.

For storage issues, preserve the original file and state whether the active location was Documents or the legacy container. Do not post complete workspace files unless you have reviewed their URLs, commands and playlist names. Never send OAuth tokens, Keychain data, client secrets or full sign-in callbacks. The public Client ID is not needed in an ordinary bug report.

Known safeguards and build limits above are expected behavior. Reproducible failures after their documented prerequisites are met may need investigation. [Testing](Testing.md) and [Hardening](Hardening.md) describe the verification coverage and its limits; mocked regression success alone does not establish that every installed third-party app version, account or display will behave the same way.
