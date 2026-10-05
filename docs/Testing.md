# Testing and verification

[Documentation home](README.md) · [Development](Development.md) · [Hardening report](Hardening.md)

The automated suite exercises backend behavior through injected system services. Native UI checks cover the macOS integration and interactions that mocks cannot prove. Keep those two kinds of evidence separate when describing a release.

## Run the regression suite

From the project root on a Mac with the active Xcode toolchain:

```sh
bash Tests/run-backend-tests.sh
bash Tests/run-backend-tests.sh release
```

The first command defaults to `debug`; `bash Tests/run-backend-tests.sh debug` is equivalent. Any other mode is rejected with exit code 2. A failing assertion or compiler warning returns a nonzero exit status. The runner compiles directly with `xcrun swiftc`; it is not an XCTest target and does not invoke `xcodebuild test`.

The script uses Swift 5 language mode, default MainActor isolation, and warnings as errors. Debug adds `-D DEBUG`; Release omits it so the development-preview boundary is checked. The runner's Release mode is a conditional-compilation check, not an optimized production app build. Build the actual Release app separately as described in [Development](Development.md).

A temporary directory contains the compiled binary and module cache, and is removed on exit. Suites create isolated fixture files, use mock launchers and script executors, fake Spotify HTTP/authentication/credential stores, injected StoreKit transactions, and controllable clocks/display adapters/schedulers. They do not launch installed applications, open websites, execute configured shell commands, access the user's Spotify credentials, or charge an account.

The backend executable reports one total assertion count, not a count of independent test cases. On 5 October 2026 the current source passed **2,243 Debug assertions** and **2,242 Release assertions**. The one-assertion difference comes from development-preview behavior. This is dated verification; rerun after changing behavior. See [Hardening](Hardening.md) for the completed native checks and reproduced failures.

## Suite map

| File | Coverage |
| --- | --- |
| `Tests/BackendRegressionTests.swift` | Main runner, browser normalization and delivery, independent app failures, layout ordering, CRUD/reordering/copying, rollback, persistence, legacy data, permission preflight, busy mutations, duplicate/stale callbacks |
| `Tests/SpotifyRegressionTests.swift` | Playlist URL/URI validation, direct delivery to Spotify, autoplay scripts, Apple-event failures, duplicate delivery boundaries |
| `Tests/SpotifyAccountRegressionTests.swift` | PKCE/state/callback validation, sign-in cancellation, credential validation, token refresh, malformed responses, pagination, rate limits, auth expiry, disconnect/client-change races |
| `Tests/WindowPlacementRegressionTests.swift` | Window candidate selection, coordinate conversion, presets/saved layouts, permissions, retries/verification, minimum-size behavior, screen changes, disconnected screens, geometry overflow/underflow |
| `Tests/PremiumRegressionTests.swift` | Verified exact-product ownership, unavailable catalog, purchase/restore/pending/cancellation, refunds, invalid transactions, query races, late results, Debug/Release preview boundary |
| `Tests/ProWorkspaceRegressionTests.swift` | Schedule model and evaluation, startup/wake/DST/clock changes, simultaneous/busy schedules, Pro gates, archive round trips and limits, launch ordering/delays/Stop, callback/stack/lifetime protection |
| `Tests/ErrorMessageRegressionTests.swift` | Contextual recovery copy, Cocoa/POSIX/network/StoreKit errors, unknown-error fallback, control-character cleanup, bounded details, raw-diagnostic redaction, hostile wrapper/code values |
| `Tests/FileReaderRegressionTests.swift` | Size boundaries, missing/nonfile URLs, invalid limits, directory/symlink/pipe rejection, prompt nonblocking behavior |
| `Tests/AdversarialStressTests.swift` | One deterministic history of 240 mixed editing steps, disk round trips, identity independence, disabled copied schedules, corrupt/external changes, 64 binary import attempts, absence of integration side effects |

## Add a meaningful regression

1. Reproduce the behavior with an isolated fixture and injected dependencies.
2. Assert the user-visible outcome and prohibited side effect, such as a second website open, a stale schedule firing, or an overwritten file.
3. Apply the fix and run the relevant suite through the runner.
4. Run both configurations if the change affects compilation gates, ownership, or shared backend code.
5. Reproduce through native UI when the issue depends on macOS permissions, application windows, authentication presentation, or file panels.

Use controllable fake schedulers rather than waiting real seconds in a regression. Do not make a test depend on the user's installed apps, playlists, network, App Store account, or saved workspaces. Existing suite helpers are intentionally local to each file; follow the file's assertions and fixture pattern.

## Native acceptance checks

Run destructive editing checks with a separate QA bundle identity and isolated data. Keep the established production identity intact for read-only restart checks and the specifically intended live integration tests. A QA identity should not inherit permissions or credentials as a shortcut.

| Area | Native acceptance check | Expected outcome |
| --- | --- | --- |
| First run | Open with no saved file | Empty-workspace UI; no error or spontaneous launch |
| Workspaces | Create, rename, duplicate, delete disposable entries; restart | Saved state survives; duplicate has independent settings and schedule off |
| App settings | Add/change app; submit unsupported website; cancel settings | Actionable validation; failed/cancelled edits do not corrupt saved settings |
| Browsers | Launch a configured HTTP/HTTPS site with a selected browser | Browser receives that site; errors identify the failed entry |
| Spotify | Connect, open playlist menu, choose playlist; refresh and restart | Playlist list loads and connection persists; choosing a playlist is not playback |
| Autoplay | Launch once with autoplay off, then on | Off delivers the playlist only; on requests playback through Spotify automation |
| Window control | Deny access, then grant to the actual test build; test placement | Launch preflight reports missing access; granted live test moves a normal window |
| Displays | Test each preset, saved layout, minimum-size app, missing saved monitor | Visible usable result or a recoverable explanation; no reported success for an unreachable target |
| Sequences | Set a long delay, launch, choose Stop, wait beyond delay | Remaining entries stay unopened; settings become editable again |
| Schedules | Keep app awake across a future minute; test sleep/startup and simultaneous entries | Only eligible current-minute launches; missed/consumed entries do not queue |
| Files | Export, review/import, cancel each panel; import malformed/oversized fixtures | Responsive file panels; independent imported copy; schedule off; no execution on import |
| Pro UI | Try free access, Debug preview, restart, actual Release | Accurate lock/paused states; preview resets and is unavailable in Release |
| StoreKit | Use Xcode local transaction manager and test restore/refund/pending | Access follows verified active ownership; no invented price or success |

These are acceptance instructions, not a claim that every row has been run in every environment. The completed checks and limits are recorded in [Hardening](Hardening.md). Public StoreKit availability, a real purchase, distribution/notarization, alternate macOS versions, every browser version, and every display topology require separate environment-specific validation.

## Verify app artifacts

Both Debug and Release app configurations should compile after code changes. For the exported build under review, replace the example path with its actual bundle:

```sh
codesign --verify --deep --strict --verbose=2 /path/to/Deylo.app
codesign -d --entitlements :- /path/to/Deylo.app
plutil -p /path/to/Deylo.app/Contents/Info.plist
```

A successful code-signature check confirms signature consistency. It does not prove notarization, App Store approval, Accessibility permission, Spotify callback setup, or paid-product availability. Inspect actual identifiers, entitlements, deployment target, and signing mode for the selected build.

## Documentation checks

Documentation changes should retain valid relative file/section links, balanced code fences, valid JSON examples, and matching UI labels. Validate the sample archive through `WorkspaceArchive.decode`/`previewImport` without opening apps or saving user data. When app behavior changes, update [UserGuide](UserGuide.md), [Integrations](Integrations.md), [Troubleshooting](Troubleshooting.md), and [DataReference](DataReference.md) together as applicable. Keep dated test evidence separate from general usage instructions.

