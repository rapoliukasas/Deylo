# Deylo hardening report

[Documentation home](README.md) · [Testing](Testing.md)

Verified on 5 October 2026.

## Error handling and interface

System errors now explain the failed action and a recovery step. Network loss, timeouts, secure-connection failures, missing files, file permissions, full disks, read-only locations, Spotify connection failures, and App Store restrictions have specific messages. Unknown failures use a contextual fallback instead of exposing raw OS descriptions, paths, response bodies, or failing URLs. Existing validation and integration messages retain their actionable explanations.

The interface now has a clearer workspace header, schedule and sequence summaries, app cards with direct settings controls, window-placement previews, scrollable settings, workspace search when the list grows, and recovery panels with readable details and file-operation retries. Sheets retain the workspace they were opened for, and duplicate save, test, and file-dialog actions are guarded.

## Reproduced failures and fixes

| System | Reproduced failure | Fix and regression coverage |
| --- | --- | --- |
| Launch callbacks | Repeated callbacks could replay an action, finish early, or queue an ordered app twice. | One-shot boundaries and launch-generation checks reject duplicate and stale callbacks. |
| Large launch sequences | Immediate failures could recursively advance a very large ordered sequence. | Sequence advancement drains iteratively; 10,000 missing-app entries complete with bounded error details. |
| Edits during launch | Configurations could change while a launch or placement test used them. | UI and manager guards keep edits, recording, saving, and reloading out of active launches. |
| Saved data | Duplicate identities, invalid frames, extreme delays, or damaged data could enter memory or be overwritten. | Transactional validation, size limits, rollback, and preservation of unreadable files. Existing loaded data survives a failed reload. |
| External file changes | Saving could replace data edited outside Deylo since it was loaded. | Saved bytes are compared before writing; a detected change requires reloading instead of overwriting it. |
| Schedules | Backward clock changes could replay a time; queued timer work could survive stopping. | Clock high-water checks, consumed-minute tracking, weak lifetime handling, and timer-generation checks. DST, wake, missed times, busy state, and free access are covered. |
| Spotify launching | Duplicate delivery callbacks could replay autoplay. | Spotify actions run once; opener failures receive contextual recovery copy before wrapping. |
| Spotify account data | Malformed or newline-containing tokens could be stored or used in requests. | Full-token validation, safe character and size limits, bounded token decoding and streamed responses, and useful rate-limit/network/Keychain messages. |
| Pro ownership | A late successful purchase result could undo a newer refund. | A changed entitlement revision triggers reconciliation against current verified ownership. Debug preview still cannot grant Release ownership. |
| Websites and commands | Unsupported schemes, embedded credentials, control characters, or excessive input could reach integrations. | Validation precedes app or script side effects; localhost handling and AppleScript escaping retain coverage. |
| Displays | Placement could succeed using a disconnected display; a recorded layout could move a window onto a missing monitor. | Display checks during placement, reachable recorded-window checks, and recovery messages. Connected secondary displays retain exact saved coordinates. |
| Geometry | Finite input coordinates could overflow during conversion, or a half-width could underflow to zero. | Computed targets are validated before returning them. |
| File import | Opening a pipe with no writer could block the import reader. | Nonblocking open, no symlink following, descriptor-based regular-file checks, and bounded reads. The old reader was reproduced in an isolated process and stopped; the new reader rejects the same fixture promptly. |

## Verification

- Debug: **2,243 regression assertions passed**.
- Release: **2,242 regression assertions passed**; the difference is development-preview behavior.
- Both builds pass without compiler warnings or errors, and their code signatures verify.
- A deterministic stress run covers **240 mixed editing steps**, disk round trips, independent copy/import identities, schedule deactivation on copying, external changes, corrupt-file recovery, and **64 binary import attempts**.
- Native UI checks verified unsupported-website feedback, Pro-preview labeling, paused paid settings after restart, the new app/settings layout, cancelling a 30-second launch delay, and successful export with the dialog closing cleanly. Import through the final reader showed the review screen, created an independent workspace, retained its launch settings, and kept its schedule disabled.
- The rebuilt main app reopened with all three existing workspaces and loaded the connected Spotify account's playlist list. After refreshing the existing macOS permission entry to the current build, Deylo recognized window-control access and successfully positioned Spotify's open window on the left.
- The three existing user workspaces validate under the new rules. Their file hash remained unchanged during this work.

Automated checks use isolated temporary files and mocked system integrations. They do not open real apps, execute terminal commands, access live Spotify accounts, or charge purchases. Editing, launch cancellation, export, and import checks used the separate validation app identity. The final restart, Spotify account, and window-placement checks used the rebuilt main app.

StoreKit commerce continues to require the owner setup described in [Commerce.md](Commerce.md).
