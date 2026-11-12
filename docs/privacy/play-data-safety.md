# Play Console Data safety: draft answers after refine v2

Draft for the founder. **Nothing has been changed in Play Console.** The founder reviews this table and submits the form.

Source of truth: the app code on `feat/refine-v2` (tarekN98/voltresso, draft PR #8) as of 12 November 2026. Each row names the file and function that justifies the answer. Anything I could not settle from the code is marked **QUESTION**.

## How Google's definitions apply (short version)

- **Collected** means data is sent off the device, to us or to anyone else. Data that is only processed on the phone is **not** collected.
- **Shared** means data is transferred to a third party. Transfers to a **service provider** that processes data on our behalf are **not** sharing (Supabase, Sentry, PostHog, Anthropic). Transfers the user starts on purpose (for example tapping "Open in Maps") are also not sharing.
- **Processed ephemerally** means held only in memory to serve the request and not kept.
- **Approximate location** is an area of at least 3 km². **Precise location** is anything finer.

## Data types

| Data type (Play category) | Collected | Shared | Ephemeral | Required / optional | Linked to user | Purposes | Code path |
|---|---|---|---|---|---|---|---|
| **Location: precise** | No | No | n/a | n/a | n/a | n/a | `lib/use-device-location.ts` `useDeviceLocation.request` reads one position with while-in-use permission and keeps it in React state on the device. Distances: `lib/cafes/geo.ts` `distanceM`. Never sent anywhere. |
| **Location: approximate** | **QUESTION**: probably Yes | **QUESTION** | **QUESTION** | Optional (only if the user opens the café map) | No | App functionality | `lib/cafes/osm.ts` `cafesNear` sends a grid cell of about 2.2 km × 2.2 km (≈4.8 km², so "approximate") around the user to the Overpass API. `components/cafes/CafeMap.tsx` `NativeCafeMap` loads map tiles for the area on screen from OpenFreeMap; at the default zoom that area is close to the user. Neither Overpass nor OpenFreeMap is our contracted service provider, which is why "shared" is unclear. |
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
| **Device or other IDs** | Yes (only with analytics on) | No | No | Optional | Yes when signed in | Analytics | PostHog anonymous device ID (`lib/analytics.ts` `initAnalytics`). |
| **Financial info: purchase history** | Not yet | | | | | | Pro purchases (RevenueCat / Google Play Billing) are not live. Declare when they launch. |

## Third-party traffic

| Destination | What it receives | Google treatment | Notes |
|---|---|---|---|
| Map tiles: OpenFreeMap (`lib/map-config.ts` `ACTIVE_TILE_PROVIDER`) | IP address and the map area on screen | **QUESTION**: collection of approximate location? sharing? | Not our contracted processor. The area is usually close to the user. |
| Overpass API, overpass-api.de (`lib/cafes/osm.ts` `cafesNear`) | IP address and a ~2.2 km grid cell | **QUESTION**: approximate location collected and shared? Ephemeral? | Independent operator; check its retention. Results cached on device 24 h. |
| PostHog (`lib/analytics.ts`) | Events, device info, anonymous ID; account ID when signed in | Collection (service provider, so not sharing) | Opt-out in Settings. |
| Sentry (`lib/sentry.ts`) | Error details, device info, traces; may log IP | Collection (service provider) | No user ID attached. |
| Supabase (whole app) | Account, brewing data, suggestions | Collection (service provider) | EU (Ireland). |
| Anthropic via `volt-chat` | Volt Pro messages and context | Collection (service provider) when Pro is live | Not live yet. |
| Google Custom Search via `volt-chat` | Search query and server IP | **QUESTION** | Unchanged by refine v2. |
| Open Food Facts (`lib/open-food-facts.ts`) | Barcode number and IP | **QUESTION**: likely nothing to declare | Unchanged by refine v2. |
| Google Maps / Apple Maps deep links (`lib/cafes/links.ts`) | The café's location, when the user taps | User-initiated transfer: not sharing | No API calls from the app. |

## Answers that probably change in the current form

The current Data safety answers are not in this repo. **Founder: please paste the current form (or a screenshot) into the PR** so each change can be checked. Based on the code, these are the likely changes:

1. **Location: approximate.** Likely now **collected** (Overpass grid cell and map tiles), optional, purpose App functionality. Sharing and ephemeral answers depend on the questions above.
2. **Location: precise.** Stays **not collected** (used only on the device).
3. **User IDs and app interactions.** If the current form says analytics data is **not linked** to the user, change to **linked** (PostHog `identify` with the account ID when signed in).
4. **Other user-generated content.** Add machine and café suggestions.
5. **Security practices.** "Data is encrypted in transit": yes (HTTPS). "Users can request deletion": yes, but note that suggestions are unlinked, not deleted, under the current database design.

## Questions for the founder

1. Is location sent to Overpass and OpenFreeMap "collected" and "shared" under Google's rules? (We send only a ~2.2 km area, never the exact position, and neither service is our contracted processor.)
2. Do you want to sign a data processing agreement with a tile provider (or self-host tiles) so it counts as a service provider?
3. Should suggestions be deleted, not just unlinked, when an account is deleted? (Needs a database and account-deletion change.)
4. Please paste the current Data safety answers so this draft can be diffed against them.
