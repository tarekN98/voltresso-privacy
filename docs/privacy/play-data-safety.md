# Play Console Data safety: draft answers after refine v2

Draft for the founder. **Nothing has been changed in Play Console.** The founder reviews this table and submits the form.

Source of truth: the app code on `feat/refine-v2` (tarekN98/voltresso, draft PR #8) as of 12 November 2026. Each row names the file and function that justifies the answer. Anything I could not settle from the code is marked **QUESTION**.

## Submitted to Play on 2026-11-14 (current form)

What the founder submitted in Play Console > App content > Data safety on 2026-11-14. This is the form as it stands for the live closed-test builds (Play AAB `19d16f6`; internal APKs `36b643e` / `09712aa` / `f23b24c`), none of which uses location.

### Changed in that submission

| Data type | Before | Submitted |
|---|---|---|
| Location > Approximate location | Ticked in a draft, never answered | **Removed** (no live build uses location) |
| Location > Precise location | Ticked in a draft, never answered | **Removed** |
| App activity > App interactions | Collected; shared (Analytics); required; Analytics | Collected; shared (Analytics, unchanged); **optional**; Analytics |
| App activity > Other user-generated content | Not declared | **Added**: collected; not shared; not ephemeral; optional; App functionality |

### Left unchanged, pending the lawyer (`lawyer-review.md` item 11)

| Question | Current form | Worksheet view (not applied) |
|---|---|---|
| "Shared" ticks on User IDs, App interactions, Device or other IDs, Crash logs, Diagnostics, Other in-app messages | Shared | Not shared: PostHog, Sentry and Supabase act as service providers |
| Personal info > Name | Required | Optional (display name) |
| Device or other IDs | Required; App functionality, Analytics | Optional (analytics can be turned off) |
| Messages > Other in-app messages | Collected; shared; ephemeral; optional; App functionality, Personalisation | Unclear: Volt Pro (Anthropic) is not live |
| App activity > In-app search history | Not declared | QUESTION: free Volt web search goes through our server to Google Custom Search |

### Full current form (for reference)

| Data type | Collected | Shared | Ephemeral | Required / optional | Purposes |
|---|---|---|---|---|---|
| Personal info > Name | Yes | No | No | Required | App functionality, Account management |
| Personal info > Email address | Yes | No | No | Required | App functionality, Account management |
| Personal info > User IDs | Yes | Yes (Analytics) | No | Required | App functionality, Analytics, Account management |
| Messages > Other in-app messages | Yes | Yes (App functionality) | Yes | Optional | App functionality, Personalisation |
| Photos and videos > Photos | Yes | No | No | Optional | App functionality, Personalisation |
| App activity > App interactions | Yes | Yes (Analytics) | No | Optional | Analytics |
| App activity > Other user-generated content | Yes | No | No | Optional | App functionality |
| App info and performance > Crash logs | Yes | Yes (App functionality) | No | Required | App functionality |
| App info and performance > Diagnostics | Yes | Yes (App functionality) | No | Required | App functionality |
| Device or other IDs | Yes | Yes (App functionality, Analytics) | No | Required | App functionality, Analytics |

Security: data encrypted in transit: Yes. Account creation: username and password; username, password and other authentication. Delete-account URL: the GitHub Pages `delete-account` page. Partial data deletion without deleting the account: No.

Play's form has no separate "linked to the user" question, so that worksheet field is not entered anywhere.

## Release checklist: café map build to Play

**Everything goes live together.** The Data safety form, the privacy policy (PR #1, `privacy/refine-v2`), the terms (PR #3, `terms/refine-v2`) and the café map build are released **in the same submission**. None of them goes live without the others: merge both policy PRs to GitHub Pages and update the form in the same Play submission that carries the build, then send that one submission for review.

All four say the same about location: the app requests coarse and fine location (`ACCESS_COARSE_LOCATION`, `ACCESS_FINE_LOCATION`), **foreground only**; the phone may allow approximate only; the **position never leaves the phone**; the map **tile provider sees the IP address and the area on screen**, not the position. Sources: privacy PR #1 (box, 3.9), terms PR #3 (Section 2), the in-app notice (`components/WhatsNewNotice.tsx`, voltresso `d0eb85e`), this checklist.

The refine-v2 build adds the café map. Its Android manifest requests `ACCESS_COARSE_LOCATION` and `ACCESS_FINE_LOCATION` (`app.json` `android.permissions`), foreground only: `ACCESS_BACKGROUND_LOCATION` is in `blockedPermissions`, and the `expo-location` plugin sets `isAndroidBackgroundLocationEnabled: false` and `isAndroidForegroundServiceEnabled: false`. The app reads one position with `requestForegroundPermissionsAsync` and `getCurrentPositionAsync({ accuracy: Balanced })` (`lib/use-device-location.ts`). It keeps that position on the phone and never sends it anywhere. None of the live builds (`19d16f6`, `f23b24c`) declares any location permission or includes `expo-location`.

1. **Before uploading the refine-v2 build to any Play track** (internal testing, closed, open or production), in the **same submission** as the build, update the Data safety form:
   - Add **Location > Approximate location**: collected; **shared** (the map tile provider receives the map area on screen); not ephemeral; **optional**; App functionality.
   - Add machine and café **suggestions** under App activity > Other user-generated content: already declared (collected, not shared, optional, App functionality). Check that the answers still fit; no new row is needed.
2. **Precise location:** our reading is **not collected**, so it is not declared. The precise position never leaves the phone, and tiles reveal only the area shown (zoom 14, roughly a 2.4 km tile). But the build does request `ACCESS_FINE_LOCATION`. Confirm with the lawyer (`lawyer-review.md` item 12), or drop `ACCESS_FINE_LOCATION` from `app.json` so the build asks for approximate location only. The map would still work, just less exactly centred.
3. **Location permissions declaration:** Play may also ask about location permissions (App content). Foreground only, no background location, so no background-location declaration should be needed. Check what Play Console asks when the build is uploaded.
4. **Privacy policy and terms:** privacy PR #1 (location and map sections) and terms PR #3 must be **published** (merged to GitHub Pages) **in the same release** as the build and the form, and before that build reaches any tester.
5. **Founder testing before that:** an **internal preview APK** (EAS `preview` profile, sideloaded, not uploaded to Play) needs **no** form change.
6. **Target audience.** At the café-map release, move the Play target audience from 18+ to 16+ only if the lawyer approved 16 (lawyer item 13). Until then Play says 18 and over (set 2026-11-14, not yet sent for review) while the draft terms and policy say 16 (TO CONFIRM).

## How Google's definitions apply (short version)

- **Collected** means data is sent off the device, to us or to anyone else. Data that is only processed on the phone is **not** collected.
- **Shared** means data is transferred to a third party. Transfers to a **service provider** that processes data on our behalf are **not** sharing (Supabase, Sentry, PostHog, Anthropic). Transfers the user starts on purpose (for example tapping "Open in Maps") are also not sharing.
- **Processed ephemerally** means held only in memory to serve the request and not kept.
- **Approximate location** is an area of at least 3 km². **Precise location** is anything finer.

## Data types

| Data type (Play category) | Collected | Shared | Ephemeral | Required / optional | Linked to user | Purposes | Code path |
|---|---|---|---|---|---|---|---|
| **Location: precise** | No | No | n/a | n/a | n/a | n/a | `lib/use-device-location.ts` `useDeviceLocation.request` reads one position with while-in-use permission and keeps it in React state on the device. Distances: `lib/cafes/geo.ts` `distanceM`. Never sent anywhere. |
| **Location: approximate** | **Yes** (founder decision) | **Yes** (founder decision) | No | Optional (only if the user opens the café map) | No | App functionality | Founder decision, Phase 4 checkpoint. Why: `components/cafes/CafeMap.tsx` `NativeCafeMap` loads map tiles from OpenFreeMap for the area on screen, which starts centred on the user (zoom 14). City café packs are **bundled in the app** (`lib/cafes/packs.ts`, data in the JS bundle), and choosing the city (`lib/cafes/pack-logic.ts` `cityForPoint`) runs on the device, so getting a café list sends nothing to our servers. The tiles alone are the reason for the declaration. The app never calls Overpass at runtime (`FLAGS.cafeOverpassDev` off; checked by `check:brew`). |
| **Personal info: email address** | Yes | No | No | Required (account) | Yes | Account management, app functionality | `context/AppContext.tsx` `signUp`, and the `profiles` upsert in `completeOnboarding` (Supabase Auth + `profiles.email`). |
| **Personal info: name** | Yes | No | No | Optional (display name) | Yes | Account management, app functionality | `context/AppContext.tsx` `signUp` (name in user metadata), `updateProfile`. |
| **Personal info: user IDs** | Yes | No | No | Required | Yes | Account management, analytics | Supabase user ID (`context/AppContext.tsx` session). Also sent to PostHog: `lib/analytics.ts` `identifyUser` (called from `AppContext` when signed in and analytics is on). |
| **Photos and videos: photos** | Yes (only if the user uploads an avatar) | No | No | Optional | Yes | App functionality (profile photo) | `app/(tabs)/profile.tsx` avatar upload to the `upload-avatar` edge function (public `avatars` bucket). |
| **Photos and videos: camera for barcodes** | No photo collected | No | n/a | Optional | n/a | n/a | `components/beans/ScanBagModal.tsx` uses the camera only to decode a barcode; `lib/open-food-facts.ts` `lookupBarcode` sends **only the barcode number** to Open Food Facts. **QUESTION**: Google has no data type for a product barcode; we believe nothing needs declaring for it. |
| **App activity: app interactions** | Yes (only with analytics on) | No | No | Optional (Settings toggle) | Yes when signed in | Analytics | `lib/analytics.ts` `track` (PostHog). Includes `brew_logged` with `count`, `method`, `via` and now `roastLevel`. The sanitiser `sanitize` drops free text, emails and unknown keys. |
| **App activity: in-app search history** | Yes (free Volt web search) | **QUESTION** | **QUESTION** | Optional | **QUESTION** | App functionality | `lib/volt-api.ts` `fetchFreeSearch` → `supabase/functions/volt-chat` → `_shared/search.ts` `searchGoogle` (Google Custom Search). Unchanged by refine v2, but check the current form covers it. |
| **App activity: other user-generated content** | Yes | No | No | Optional | Yes | App functionality | Brew logs, recipes, saved items: `lib/sync.ts` (`syncBrewLogUpsert`, `syncCustomRecipeUpsert`, `syncSavedRecipeAdd`, `syncSavedBeanAdd`). **New:** machine suggestions `context/AppContext.tsx` `submitMachine` (`machine_submissions`); café suggestions `lib/cafes/voltresso.ts` `submitCafe` (`cafes`). |
| **App activity: other actions** | No new | | | | | | Local-only additions in refine v2 (roast level on bags, My machines, level cap, skill-check time) are never sent off the device. |
| **Messages: other in-app messages** | Only when Volt Pro is live | No (Anthropic is a service provider) | **QUESTION** | Optional | **QUESTION** | App functionality | `lib/volt-api.ts` `streamVoltMessage` → `supabase/functions/volt-chat` → Anthropic. Volt Pro is not live (`PRO_PAYMENTS_LIVE` false); declare when it launches. |
| **App info and performance: crash logs** | Yes | No | No | Required (always on) | No | App functionality, analytics (diagnostics) | `lib/sentry.ts` `initSentry` (`sendDefaultPii: false`, `beforeSend` scrubs emails, tokens and free text; no `setUser` anywhere). |
| **App info and performance: diagnostics** | Yes | No | No | Required | No | App functionality | `lib/sentry.ts` performance traces (`tracesSampleRate: 0.2`). |
| **Device or other IDs** | Yes (only with analytics on) | No | No | Optional | Yes when signed in | Analytics | PostHog random device ID (`lib/analytics.ts` `initAnalytics`). |
| **Financial info: purchase history** | Not yet | | | | | | Pro purchases (RevenueCat / Google Play Billing) are not live. Declare when they launch. |

## Third-party traffic

| Destination | What it receives | Google treatment | Notes |
|---|---|---|---|
| Map tiles: OpenFreeMap (`lib/map-config.ts` `ACTIVE_TILE_PROVIDER`) | IP address and the map area on screen | **QUESTION**: collection of approximate location? sharing? | Not our contracted processor. The area is usually close to the user. |
| City café packs (`lib/cafes/packs.ts`) | Nothing: the packs are bundled in the app (JS bundle) | Nothing to declare | Founder decision: bundled, no pack server. New or refreshed packs arrive with an app update. Overpass is called only by the founder-run build script, never by the app. |
| App updates: Expo EAS Update (`app.json` `updates.url`, new in refine v2) | IP address, platform, app/runtime version, current update ID, when the app checks for an update | **QUESTION**: likely collection by a service provider (app functionality); not covered by the current policy | The live build (`36b643e`) has no `expo-updates`; refine v2 adds it. Founder wording needed in the policy (Expo row) and here. |
| PostHog (`lib/analytics.ts`) | Events, device info, anonymous ID; account ID when signed in | Collection (service provider, so not sharing) | Live builds: on by default, opt-out in Settings. From refine v2: opt-in, no PostHog call before a yes. |
| Sentry (`lib/sentry.ts`) | Error details, device info, traces; may log IP | Collection (service provider) | No user ID attached. |
| Supabase (whole app) | Account, brewing data, suggestions | Collection (service provider) | EU (Ireland). |
| Anthropic via `volt-chat` | Volt Pro messages and context | Collection (service provider) when Pro is live | Not live yet. |
| Google Custom Search via `volt-chat` | Search query and server IP | **QUESTION** | Unchanged by refine v2. |
| Open Food Facts (`lib/open-food-facts.ts`) | Barcode number and IP | **QUESTION**: likely nothing to declare | Unchanged by refine v2. |
| Google Maps / Apple Maps deep links (`lib/cafes/links.ts`) | The café's location, when the user taps | User-initiated transfer: not sharing | No API calls from the app. |

## Answers that probably change in the current form

Superseded by "Submitted to Play on 2026-11-14" and the release checklist above.

## Questions for the founder

1. Decided: approximate location is collected and shared (founder).
2. Do you want a data processing agreement with a tile provider (or self-hosted tiles) so it counts as a service provider?
3. Decided: suggestions are unlinked on account deletion and their free text deleted (founder).
4. Done: the form was read and updated on 2026-11-14 (section above).
5. Settled: the live closed-test build is EAS `b571e6c9` (Android, `preview` profile, internal APK, 2026-11-01) from commit `36b643e` on `feat/sprint1-p0`, not `main`/130a7cb. Please confirm testers have that APK.
6. EAS Update: refine v2 turns on over-the-air updates (Expo). How should the policy and this form describe the update check?
