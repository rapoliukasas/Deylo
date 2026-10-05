# Deylo Pro commerce setup

[Documentation home](README.md) · [User guide](UserGuide.md) · [Development](Development.md) · [Troubleshooting](Troubleshooting.md)

Deylo Pro is a single, non-consumable App Store purchase. Its identifier is `com.deylo.pro.lifetime`. The app reads the localized price from StoreKit; it does not use a hard-coded selling price. The €24.99 value in `Deylo.storekit` is a proposed price for local development, with a Lithuanian test storefront. There is no subscription or recurring billing product.

## What is implemented

`PremiumManager` loads verified current entitlements at launch and listens for transaction updates. A purchase unlocks Pro only when Apple verifies an active non-consumable transaction for the exact product identifier. Unverified, revoked, unrelated, and incorrect-type transactions are rejected. Delivered verified purchases are finished after access is granted. Refund and revocation updates remove access. Restore Purchase invokes Apple's explicit account sync, then checks ownership again.

Ownership does not depend on a preference or on product availability: a previously verified purchase can be restored from StoreKit's current entitlements even when the product catalog is unavailable. Fresh installations have no invented entitlement, price, or success state. Missing products report that the purchase is unavailable.

The development-only Pro Preview is separate from ownership. It is memory-only, resets when the app restarts, and cannot grant access in a Release build. It never initiates a purchase or writes a local license.

## Local StoreKit testing

Open the project in Xcode. In the run scheme's **Options**, explicitly select the project-root `Deylo.storekit` file as the StoreKit configuration. This activates Apple's local test environment, not the live App Store. Keep the configuration disabled for production runs. The file is outside the app's synchronized resource folder and is not a production license.

Use Xcode's transaction manager to test purchase, cancellation, pending approval, restore, and refund/revocation. Restart the app to verify ownership restoration. Disable the local configuration afterward. The injected-store regression suite covers these state transitions, rejected verification, unavailable products, duplicate purchase actions, and stale entitlement query races without contacting a payment service or charging an account.

## Before real purchases can ship

The account owner must configure the existing app's exact bundle identifier in App Store Connect, create the non-consumable `com.deylo.pro.lifetime` product, supply its display name, description and review materials, choose final pricing and availability, and complete Apple's required commercial agreements. The app also needs the owner's distribution signing and an approved App Store release. This work does not change the existing bundle identifier, Spotify callback scheme, or credential storage, so current workspaces and Spotify connections remain associated with the same app.

An ad-hoc development build does not establish a live paid product. Until the product is configured and available to this app, the upgrade panel correctly shows an unavailable purchase. The repository implementation does not create an App Store listing or publish, sign, approve, or activate commerce on the owner's behalf.

The current Debug and Release desktop targets have App Sandbox disabled. Apple requires App Sandbox for Mac App Store submission, so the owner must also resolve the distribution approach and verify compatibility of Deylo's window-control and Apple-event integrations with that channel. This is an open release prerequisite, not a completed store configuration. [Apple's App Sandbox configuration documentation](https://developer.apple.com/documentation/xcode/configuring-the-macos-app-sandbox). Direct distribution would need its own signing/notarization and a compatible purchase design; a Developer ID build does not activate this App Store product by itself. See [Development](Development.md) for the release workflow.

Apple references: [In-App Purchase setup and agreements](https://developer.apple.com/help/app-store-connect/configure-in-app-purchase-settings/overview-for-configuring-in-app-purchases/), [current entitlements](https://developer.apple.com/documentation/storekit/transaction/currententitlements), [transaction updates](https://developer.apple.com/documentation/storekit/transaction/updates), [explicit restore sync](https://developer.apple.com/documentation/storekit/appstore/sync()), and [finishing delivered transactions](https://developer.apple.com/documentation/storekit/transaction/finish()).
