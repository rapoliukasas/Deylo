# Integrations

[Documentation home](README.md)

This page describes the integrations implemented in the current source. Follow the [User Guide](UserGuide.md) for normal setup, [Troubleshooting](Troubleshooting.md) for recovery, and [Development](Development.md) for build and signing details. [Data Reference](DataReference.md) covers stored settings and portable files; [Commerce](Commerce.md) covers the App Store integration.

## How a workspace uses integrations

Each app entry identifies an installed macOS application by its bundle identifier. Deylo asks `NSWorkspace` to launch that application, waits for the launch callback, performs its optional integration action, then applies its window placement. A failed optional integration does not suppress a requested window placement. If both fail, the message includes both explanations. Other app entries continue after a failure, including entries in an ordered launch.

| Action | Mechanism | Required setup |
| --- | --- | --- |
| Launch an app | `NSWorkspace.openApplication` | An installed application matching the configured bundle identifier |
| Open a browser website | `NSWorkspace.open(_:withApplicationAt:)` | A recognized browser and valid HTTP/HTTPS URL |
| List Spotify playlists | Spotify OAuth Authorization Code with PKCE and Web API | A public Client ID, Spotify authorization, working network and login Keychain |
| Open a Spotify playlist | A validated `spotify:playlist:` URI delivered to the Spotify desktop app | Installed Spotify desktop app; a saved playlist selection |
| Automatically play a Spotify playlist | AppleScript sent to Spotify | Spotify desktop app, available playlist, macOS Automation permission and a build allowed to send the event |
| Run an iTerm command | AppleScript sent to iTerm | Installed iTerm, a saved command and macOS Automation permission |
| Record, test or apply a window layout | macOS Accessibility APIs | Window-control permission and an eligible normal app window |

Opening apps, websites and playlist links does not itself ask for Accessibility permission. A workspace containing any placement other than **Leave unchanged** checks window-control permission before opening any of its apps. If access is missing, launch stops at that check and the recovery panel offers setup and rechecking.

An ordered launch waits for an entry's launch, integration and placement callbacks before advancing. Its per-app delay is a fixed wait before opening that entry, including the first entry. It does not probe a website, wait for a server to listen, inspect a command's output or confirm that Spotify is audibly playing. Without Pro access, manual launches use the normal concurrent behavior and saved delays are ignored. **Stop Launch** prevents remaining sequence entries from starting; apps already opened and actions already delivered remain in effect.

Source: `macapp/WorkspaceManager.swift`, `macapp/AutomationManager.swift`.

## Application selection and browser routing

The application chooser combines currently running apps with a short list of commonly used installed apps. Running apps must have a regular activation policy and a bundle identifier, and Deylo excludes itself. The common list contains Safari, Dia, Chrome, Edge, Firefox, Brave, Arc, iTerm, Apple Terminal, Xcode, Visual Studio Code, Slack and Spotify when macOS can resolve them. Choices are deduplicated by bundle identifier and sorted by name.

This is not a complete scan of every installed application. Open an app so it appears in the running list, or use **Other Application…** and supply its name and bundle identifier. Launching still requires macOS to resolve that identifier to an installed app. Saving an identifier does not prove the app is installed.

Browser capability is an explicit bundle-identifier allowlist. Registering an HTTP handler is not enough to qualify an application as a browser. The current list is:

| Browser or variant | Bundle identifier |
| --- | --- |
| Safari | `com.apple.Safari` |
| Safari Technology Preview | `com.apple.SafariTechnologyPreview` |
| Dia | `company.thebrowser.dia` |
| Arc | `company.thebrowser.Browser` |
| Chrome | `com.google.Chrome` |
| Chrome Canary | `com.google.Chrome.canary` |
| Chromium | `org.chromium.Chromium` |
| Microsoft Edge | `com.microsoft.edgemac` |
| Firefox | `org.mozilla.firefox` |
| Firefox Nightly | `org.mozilla.nightly` |
| Brave | `com.brave.Browser` |
| Vivaldi | `com.vivaldi.Vivaldi` |
| Opera | `com.operasoftware.Opera` |
| Opera GX | `com.operasoftware.OperaGX` |

The website setting appears for recognized browsers. Deylo delivers the website to the application named in that entry, even when a different browser is the Mac's default. It does not choose a profile, private-browsing mode, browser window or tab; the destination browser controls those details. Deylo does not use browser AppleScript or read page content.

Source: `ApplicationCapabilities` in `macapp/Models.swift`, `availableApplications` and `saveApplication` in `macapp/ContentView.swift`, and `openBrowserURL` in `macapp/AutomationManager.swift`.

### Website normalization and validation

Deylo trims surrounding whitespace and normalizes a website when saving settings and again when opening it. A public address with no scheme defaults to HTTPS. Recognized loopback addresses with no explicit HTTP/HTTPS scheme default to HTTP:

| Input | Normalized behavior |
| --- | --- |
| `example.com` | `https://example.com` |
| `example.com:8443/path` | HTTPS, preserving the host, port and path |
| `//example.com/path` | HTTPS |
| `localhost:3000` | `http://localhost:3000` |
| `project.localhost:3000/path` | HTTP |
| `127.0.0.1:3000` | HTTP |
| `[::1]:3000` | HTTP |
| `https://localhost:3000` | Explicit HTTPS is retained |
| `http://example.com/path` | Explicit HTTP is retained |

Loopback recognition includes `localhost`, names ending in `.localhost`, valid four-part IPv4 addresses in `127.*.*.*`, and IPv6 `::1`. Other local network names and addresses default to HTTPS when the scheme is omitted. Use an explicit `http://` address for a local server that needs HTTP.

The parser rejects empty addresses, hidden control characters, more than 16,384 characters, schemes other than HTTP/HTTPS, incomplete `http:`/`https:` forms without `//`, missing or invalid hosts, ports outside 1–65,535, and embedded usernames or passwords. A host with a numeric port is distinguished from a URI scheme. These checks validate syntax and allowed URL shape; they do not check whether a host exists, whether a certificate is trusted or whether a server is ready.

Website URLs and terminal commands are stored as workspace configuration and included in portable backups. Treat query parameters, paths and commands as potentially private when sharing a backup. Credentials embedded as URL user information are rejected; sign in through the browser.

Source: `websiteURL(from:)` in `macapp/AutomationManager.swift`.

## Spotify

### Account connection and desktop playback are separate

Connecting a Spotify account enables the playlist picker. The account integration reads playlist metadata through the Web API. It does not play audio, modify playlists or control a playback device through that API.

Launching a saved playlist uses the installed Spotify desktop application (`com.spotify.client`). Deylo first launches Spotify, then opens the playlist URI in that app. **Automatically play this playlist** is off by default. When off, Deylo sends no playback command; Spotify's own existing playback state can continue. When enabled, Deylo sends `play track` through AppleScript after the playlist URI has been opened, with a 15-second script timeout.

These desktop actions do not consult the OAuth connection state. A previously selected playlist can still be opened after disconnecting the account. The UI retains a previously selected playlist even if the current account's list or search results do not contain it. Removing the selection turns autoplay off.

The current settings UI chooses playlists from the connected account rather than offering a general playlist paste field. The backend also accepts valid public playlist links and Spotify playlist URIs in stored settings and imported files. It normalizes them to `spotify:playlist:<id>`; the ID must contain exactly 22 ASCII letters or digits. Web links must have host `open.spotify.com`, no credentials or explicit port, and a `/playlist/<id>` path, optionally prefixed with `/intl-<two letters>/` and ending in `/`. Link query and fragment data do not become part of the playlist URI. Track, album, shortened and other Spotify links are rejected. Old Spotify entries using the general `url` field remain supported by the launch path; resaving them uses the Spotify-specific setting.

Source: `spotifySettings` in `macapp/ContentView.swift`, `runAutomation` in `macapp/WorkspaceManager.swift`, and `spotifyPlaylistURL`, `openSpotifyPlaylist` and `spotifyPlaylistScript` in `macapp/AutomationManager.swift`.

### One-time developer app setup

The committed project does not supply a built-in Spotify Client ID. Use the setup sheet for the local development app:

1. Open the [Spotify Developer Dashboard](https://developer.spotify.com/dashboard), create or select a developer app and enable the Web API as directed by the setup sheet.
2. Add the exact redirect URI `macapp-spotify-auth://callback` in that Spotify app's settings. Keep the spelling and trailing-slash choice exact. Spotify requires the authorization redirect to match a registered URI. [Spotify's PKCE guide](https://developer.spotify.com/documentation/web-api/tutorials/code-pkce-flow).
3. Add the Spotify account that will sign in to the developer app's allowed users when required by its quota mode. Current development-mode rules require the app owner to have Premium and limit new apps to five authenticated users. A user can complete sign-in yet receive HTTP 403 for Web API requests when not allowlisted. These are Spotify account restrictions. [Spotify quota modes](https://developer.spotify.com/documentation/web-api/concepts/quota-modes).
4. In Deylo's Spotify app settings, open **Connection Settings…** or **Connect Spotify**, enter the public 32-character hexadecimal Client ID, and choose **Save & Connect**.
5. Complete the browser sign-in and consent, then choose a playlist and save the app settings.

Use only a public Client ID. No client secret is requested, stored or sent by this flow. The code resolves the Client ID in this order: an explicitly injected initialization value, the `SpotifyClientID` user preference, the bundle's `SpotifyClientID` Info value, then an empty string. If this resolved value is empty, a valid stored Keychain connection can supply its Client ID. The repository's Xcode settings contain no `SpotifyClientID` Info value. A runtime user preference or saved connection can therefore persist an earlier setup without placing an ID in source.

Changing the Client ID cancels an active sign-in or playlist load and clears the current local connection when one exists. Connect again for the new ID. The Client ID preference itself remains after **Disconnect**.

The app uses `ASWebAuthenticationSession` with the custom callback scheme `macapp-spotify-auth`; it does not start a local callback web server. The committed project does not register a general `CFBundleURLTypes` handler. The sign-in session receives its callback through AuthenticationServices. Callback validation accepts the expected scheme and host `callback`, an empty or literal `/` path, no credentials, port or fragment, exactly one matching `state`, and exactly one nonempty authorization code. A changed path, duplicate state/code or mismatched state fails verification and requires a new connection attempt.

Source: `SpotifyConnectionSetupSheet` in `macapp/ContentView.swift`, `init`, `setClientID`, `connect` and `authorizationCode` in `macapp/SpotifyAccountManager.swift`, and `macapp.xcodeproj/project.pbxproj`.

### OAuth, scopes and playlist loading

The implemented flow is Authorization Code with PKCE using a cryptographically random verifier, SHA-256 (`S256`) challenge, random state and a browser consent request. It asks for precisely:

| Scope | Purpose |
| --- | --- |
| `playlist-read-private` | Include private playlists |
| `playlist-read-collaborative` | Include collaborative playlists |

Those meanings are defined by [Spotify's scope reference](https://developer.spotify.com/documentation/web-api/concepts/scopes). Deylo requires both scopes on the initial token response; it also checks any scope field returned during a refresh. No email, profile, playlist-write, streaming or Web API playback-control scope is requested by the source.

Authorization uses `https://accounts.spotify.com/authorize`; code exchange and refresh use `https://accounts.spotify.com/api/token`. Playlist loading starts at `https://api.spotify.com/v1/me/playlists?limit=50`, the endpoint for the signed-in user's owned or followed playlists. [Spotify's current-user playlist reference](https://developer.spotify.com/documentation/web-api/reference/get-a-list-of-current-users-playlists).

The loader follows validated pagination URLs and retains the response order. It skips null entries and deduplicates by playlist ID. It checks that each playlist's URI agrees with its ID. Pagination must stay on HTTPS `api.spotify.com`, at `/v1/me/playlists`, without credentials, port or fragment; repeated page URLs and more than 2,001 pages are rejected. Playlist display names and optional owner display names are used for the picker. Search appears when more than ten playlists are loaded and filters playlist names locally, without another API search request.

An access token is reused while more than 30 seconds remain before expiry. Otherwise Deylo refreshes it. A playlist HTTP 401 triggers one forced refresh and one retry; a second 401 disconnects the local account. Expired/revoked refresh grants and lost playlist scopes also invalidate the local connection. A stored credential makes the UI initially appear connected, but its validity is established only when a network request succeeds. The account list is not persisted across app launches.

### Credentials, network and privacy boundaries

| Data or operation | Current source behavior |
| --- | --- |
| Public Client ID | Stored in `UserDefaults` under `SpotifyClientID` |
| Refresh credential | Stored as a generic-password item in the login Keychain, using service `<app bundle identifier>.spotify-oauth`, account `refresh-token`, with Keychain synchronization disabled |
| Access token | Held in memory with its expiry; not written into workspaces or preferences |
| Playlist list | Held in memory; the selected playlist URI, optional name and autoplay choice become workspace configuration |
| Web sign-in | Requests an ephemeral browser session and uses a current app window as its presentation anchor |
| API requests | Use an ephemeral `URLSession`, no URL cache or cookie storage, 30-second request timeout and 45-second resource timeout |
| Redirected API/token response | HTTP redirects are refused, so bearer headers and exchange bodies are not forwarded to another destination |
| Response sizes | Network reads are capped at 4 MiB; token responses are additionally capped at 64 KiB |
| **Disconnect** | Cancels pending sign-in, clears memory and playlist list, and attempts to delete this app's local Keychain credential |

Disconnect does not revoke Spotify's server-side consent or change Spotify's playback. If local credential deletion fails, Deylo shows that failure and asks you to try **Disconnect** again. A token invalidation uses the same local deletion boundary. Keychain reads avoid presenting an interactive authentication prompt during initialization; a locked or inaccessible login Keychain produces an actionable error. Missing signing entitlement errors are identified separately from normal Keychain-access failures.

Workspace JSON and `.deylo` backups contain no OAuth credential fields, access tokens, refresh tokens or purchase records. They can still contain private playlist names, browser URLs and executable terminal commands. Deylo's error formatting avoids raw network response bodies, token values, callback query values and OS error dumps. Structural callback errors report only the shape of the unexpected callback. Do not include authorization callback URLs or Keychain contents in a bug report.

Source: `macapp/SpotifyAccountManager.swift`, `macapp/AppErrorMessage.swift`, `macapp/WorkspaceArchive.swift`.

### Spotify permissions and build limits

Spotify consent to read playlists is independent of macOS permission to control the Spotify app. Autoplay may prompt for permission under **System Settings → Privacy & Security → Automation**. Allow Deylo to control Spotify there. Apple's Automation settings govern permission for one app to control another. [Apple's Automation guide](https://support.apple.com/guide/mac-help/allow-apps-to-automate-and-control-other-apps-mchl108e1718/mac).

The entitlement file declares Apple-event automation and the Spotify `com.spotify.playback` scripting target. macOS can still deny an event because of the running build's signing/permission state. The code distinguishes user permission denial (`-1743`), a build without Spotify playback access (`-10004`), and a timed-out Spotify response (`-1712`). Turning off autoplay keeps playlist opening available in all three cases. See [Troubleshooting](Troubleshooting.md#spotify-playlist-opening-and-autoplay) for recovery.

The source does not check a Spotify subscription, an audible output device or playlist availability before sending the desktop action. A successful URI or AppleScript delivery is not proof that the account can play the requested playlist. Development-mode Web API owner restrictions do not establish a subscription requirement for every desktop playback user.

## iTerm commands

The terminal-command integration is available only for iTerm's bundle identifier `com.googlecode.iterm2`. Apple Terminal can be launched and placed as an ordinary app, but Deylo does not execute commands in it. The current AppleScript addresses the application named `iTerm`.

On workspace launch, Deylo first opens iTerm. The script then creates a window with the default profile when iTerm has no windows. Otherwise it creates a new tab with the default profile in iTerm's current window. It writes the saved command to that window's current session. There is no choice of profile, window, working directory or environment in Deylo. Include a `cd` or environment assignment in the command itself when needed.

The settings UI trims leading and trailing whitespace and treats a blank command as absent. Validation accepts at most 32,768 characters and rejects hidden control characters except newline, carriage return and tab. Strings are escaped for AppleScript, including quotes and backslashes, so the command remains text in the script. iTerm's shell then interprets that text as a command; it can have all effects available to that shell. Portable imports display commands for review and do not run them until a later workspace launch.

Deylo does not capture output, inspect a process exit code, wait for completion or stop a command already sent to iTerm. A successful action means the AppleScript returned without an error. Ordered-launch timing does not wait for a development server started by the command to become ready. Repeated workspace launches can open additional tabs and execute the command again.

iTerm control uses macOS Automation permission, separate from window-control permission. The entitlement file declares an Apple-event temporary exception for `com.googlecode.iterm2`. A denied event (`-1743`) produces an iTerm-specific Automation recovery message; other script errors ask you to check that iTerm is open and retry. [Apple's Automation guide](https://support.apple.com/guide/mac-help/allow-apps-to-automate-and-control-other-apps-mchl108e1718/mac).

Source: `validateITermCommand`, `iTermScript` and `executeAppleScript` in `macapp/AutomationManager.swift`, `runAutomation` in `macapp/WorkspaceManager.swift`, and `macapp/macapp.entitlements`.

## Window controls and displays

### Permission and eligible windows

Deylo reads and sets window positions and sizes with macOS Accessibility (`AX`) APIs. The UI calls the permission pane **Device Control and Data Access**, with **Accessibility** given as the name on earlier macOS versions. Follow the pane available on the running Mac; Apple's published guide describes enabling the app under **Privacy & Security → Accessibility** and adding it if absent. [Apple's Accessibility permission guide](https://support.apple.com/guide/mac-help/allow-accessibility-apps-to-access-your-mac-mh43185/mac).

**Set Up Access** requests the macOS trust prompt and opens the privacy settings deep link. **Check Again**, returning to the active main app and entering App Settings recheck trust without prompting. **Record Layout** can request the prompt. **Leave unchanged** bypasses window access entirely. Screen-recording permission is not requested by this implementation.

For each bundle identifier, the native adapter looks at the first matching running application. It considers the focused window first, then the app's remaining exposed windows. It chooses the first eligible normal window. A window must have Accessibility role `AXWindow`, must not be modal, and must have no subrole or the standard-window subrole. This accepts normal Electron windows that omit a subrole. Minimized, full-screen, dialog and explicitly nonstandard windows are not eligible. If another normal window exists, that window can be selected instead.

Only one window is managed per app entry. Deylo does not track a window's title, document, tab or stable identity, recreate closed documents, restore every window of a multi-window app, unminimize windows or leave full screen automatically.

### Presets and recorded frames

**Left half**, **Right half**, **Top half**, **Bottom half** and **Fill display** use the first screen in `NSScreen.screens`, consistently targeting the primary display instead of the display following keyboard focus. The usable area comes from that screen's `visibleFrame`, which excludes reserved desktop areas. **Fill display** resizes a normal window to that area; it does not enter macOS full screen.

Geometry uses points, including Retina displays. AppKit's bottom-left coordinates are converted to Accessibility's top-left coordinates relative to the primary display. Presets are recomputed for every request rather than saved as fixed screen rectangles.

**Record Layout** records eligible currently open windows without launching apps. Each captured app receives a frame and switches to **Recorded layout**. Apps with no usable window are skipped, so a successful operation reports how many positions were saved out of the workspace's app count. If none can be recorded, it reports that no usable app windows were found. A persistence failure restores the previous app settings.

Recorded layouts retain absolute coordinates and sizes. They can restore to a connected secondary display, but contain no display identifier or relative layout specification. They are not automatically scaled or moved to a substitute screen when the arrangement changes. Before and during restoration, Deylo checks that enough of the saved window's top edge overlaps a connected display's usable area. A wholly unreachable layout is rejected with instructions to reconnect its display, select another placement or record it again. This is a reachability check, not a requirement that the entire saved rectangle fit on one display.

**Test Placement** acts on the selected application's currently open eligible window; it does not launch the app or run a website, Spotify or iTerm action. The test can use unsaved placement choices, so it moves a real window before the settings are saved. Save the settings to use that choice on later launches.

### Placement reliability and limitations

Placement is asynchronous: Deylo sets the target size, moves the window, resizes again after moving, aligns it and verifies that the reported result becomes stable. It retries transient window/AX failures for up to five seconds, with 0.12-second scheduling intervals and short native AX messaging timeouts. Permission is rechecked during the operation. A display change during a preset aborts with a retry message.

Presets accommodate an app's minimum size if the resulting rectangle fits the display's usable area and remains aligned. A right-half window stays aligned with the right edge; a bottom-half window stays aligned with the bottom edge. An app's minimum size can make a half-placement wider or taller than half, so side-by-side windows may overlap. A result larger than the usable display or smaller than the requested preset does not count as success. Recorded layouts require the saved rectangle to match within one point, so a changed app minimum size can prevent exact restoration.

Some apps refuse settable position or size attributes, defer changes, impose size constraints or expose no normal Accessibility window. Those limitations produce explicit errors rather than a claimed success. Open a normal window, leave full screen, restore it from the Dock, try another placement or use **Leave unchanged**. The complete error-to-recovery table is in [Troubleshooting](Troubleshooting.md#window-control-and-layouts).

Source: `macapp/AccessibilityManager.swift`, `recordLayout`, `testWindowPlacement` and permission helpers in `macapp/WorkspaceManager.swift`, and the placement settings in `macapp/ContentView.swift`.

## Build identity and integration scope

The app is displayed as Deylo; older permission entries or builds may use `macapp`. The current Debug and Release target settings disable App Sandbox, enable Hardened Runtime and use the existing generated bundle identifier. The entitlement file declares outbound networking, Apple-event automation, the Spotify playback scripting target and the iTerm Apple-event exception. These declarations describe the build's configuration; they do not grant user-controlled macOS permissions or prove a distribution build is ready for the App Store.

Permission grants and the Spotify Keychain service are associated with the running app's identity. A different bundle identifier chooses a different Keychain service and legacy workspace-container lookup. Build/signing changes can require checking the permission entry for the actual installed build. Preserve the existing identity when maintaining a user's setup. See [Development](Development.md) for configuration and [Commerce](Commerce.md) for release requirements.

The app includes no cloud workspace sync, account server, command-output collection, browser-content scraping or Web API playback integration. The source has specific adapters for safe mocked regression tests; passing those tests establishes behavior at those adapter boundaries. Native app versions, account permissions, signing and the connected displays remain relevant when validating a real installation. See [Testing](Testing.md) for verification coverage and limits.
