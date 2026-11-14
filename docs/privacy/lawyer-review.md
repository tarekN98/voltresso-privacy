# Lawyer review: open points in the privacy policy (draft)

For the lawyer reviewing the policy before the production release. **Nothing here is decided legal advice.** Each item is marked `[TO CONFIRM]` in `index.html` (or noted where the page has no marker) and gives the question, why it matters, and where the behaviour lives.

- Policy: `index.html` on `privacy/refine-v2` (PR #1, the refine-v2 release) and `privacy/fix-current` (PR #2, the live closed-test builds).
- Play Console answers: `docs/privacy/play-data-safety.md`.
- App code: `tarekN98/voltresso`, branch `feat/refine-v2` (draft PR #8) unless a live-build commit is named.
- Controller: Tarek Sayed, sole developer, resident in Serbia (policy section 1). Users are mainly in the EU; Serbia's data protection law (ZZPL) follows the GDPR.

## 1. Legal basis for analytics

- **Question:** Is consent the right basis for analytics from the refine-v2 release, and what basis applies to the live builds, where analytics is on by default?
- **Why it matters:** Live builds (Play AAB `19d16f6`; November APKs `36b643e`, `09712aa`, `f23b24c`) send PostHog events by default (opt-out in Settings), and the November builds link them to the account ID. PR #2 no longer claims consent for these builds and marks the basis TO CONFIRM. Refine v2 asks first and sends nothing before a yes, and PR #1 names consent. The ePrivacy rules on device storage/access may also apply to the PostHog device identifier.
- **Where:** policy sections 3.7, 4, 5, 10; `lib/analytics-core.ts`, `lib/analytics.ts`, `components/AnalyticsConsentPrompt.tsx`, `context/AppContext.tsx` (`analyticsConsent`); proof in `scripts/check-analytics-consent.ts`. Live-build default: `context/AppContext.tsx` `safeParse(analytics) ?? true` at the commits above.
- **Also:** do existing testers' events collected under the old default-on behaviour need any action (notice, deletion)?

## 2. Expo (EAS Update) processor terms

- **Question:** Is Expo a processor for the update check, and are its data processing terms and transfer mechanism (United States) adequate? Which exact fields does Expo receive and log? What is the legal basis (performance of a contract or legitimate interests)?
- **Why it matters:** From refine v2 the app checks Expo's update service; Expo receives the IP address and app/platform details (platform, app and update version, channel). The live builds have no `expo-updates`, so this is new with the release. Play has no specific data type for it.
- **Where:** policy 3.8 ("App updates") and section 5 (Expo row); `app.json` `updates.url`, `runtimeVersion`; `package.json` `expo-updates`. Expo privacy policy: https://expo.dev/privacy

## 3. PostHog processor terms

- **Question:** Is a DPA with PostHog in place, and is the US transfer covered (SCCs / EU-US Data Privacy Framework)? Is the stated retention ("per their retention policy") enough? Should account deletion trigger a PostHog person deletion instead of an email request?
- **Why it matters:** Events are linked to the account ID (live November builds and refine v2 after a yes). Account deletion does not delete PostHog data today; the policy offers an on-request erasure.
- **Where:** policy 3.7, 5 (PostHog row, `[TO CONFIRM: PostHog data processing terms and transfer mechanism]`), 6, 10; `supabase/functions/delete-account/index.ts` header comment; host `EXPO_PUBLIC_POSTHOG_HOST` (default `us.i.posthog.com`).

## 4. Supabase processor terms

- **Question:** Is the Supabase DPA accepted, and does it cover sub-processors outside the EU (support, backups)? The policy says the database is in the EU (Ireland).
- **Why it matters:** Supabase holds the account, brewing data, settings, suggestions and avatars. No marker on the page today; the policy states "Data processing agreements are available from each provider on request".
- **Where:** policy sections 5 and 6; project ref in `supabase/config.toml`; migrations in `supabase/migrations/`.

## 5. Map tile provider (OpenFreeMap)

- **Question:** Is OpenFreeMap a processor or an independent third party (no DPA exists)? What is its hosting region and privacy page? What legal basis covers sending the IP address and the viewed map area?
- **Why it matters:** Tiles reveal the approximate area the user looks at, which is why Play's approximate location is declared collected and shared. Alternatives in voltresso PR #8: self-hosted tiles (Protomaps) or a contracted provider.
- **Where:** policy 3.9 (`[TO CONFIRM: link to OpenFreeMap's privacy page]`, `[TO CONFIRM: legal basis, see Section 4]`), 4 (`[TO CONFIRM: legal check of these bases.]`), 5 (OpenFreeMap row `[TO CONFIRM]`); `lib/map-config.ts` `TILE_PROVIDERS`, `ACTIVE_TILE_PROVIDER`; `components/cafes/CafeMap.tsx`.

## 6. OpenStreetMap data in the app (ODbL)

- **Question:** Does bundling city extracts of OpenStreetMap in the app meet the ODbL attribution and share-alike obligations (is a city pack a "Derivative Database" or a "Produced Work")?
- **Why it matters:** Café lists are prepared from OSM and shipped inside the app and its updates.
- **Where:** policy 3.9 and section 5 note; `scripts/data/build-cafe-pack.mjs`, `lib/cafes/bundled-packs.ts`, attribution in `lib/map-config.ts` `OSM_ATTRIBUTION`.

## 7. Children and age wording

> Update (KAN-116): both drafts now say 16, not 18. The open questions are in item 13.

- **Question:** The policy says the app is "not directed at children under 18". Is 18 the right threshold (EU digital-consent ages are 13 to 16; Serbia 15), and is a statement enough without an age check at sign-up? Does it match the Play Console target age group?
- **Why it matters:** The app has no age question or age gate (no age check in `app/login.tsx` or onboarding). If minors can sign up, consent for analytics may need a parent.
- **Where:** policy section 9 (no marker on the page); `terms.html` (has no age or eligibility clause); Play Console > App content > Target audience.

## 8. Retention

- **Question:** What retention period applies to machine and café suggestions after they are unlinked, and is "kept while useful" acceptable? Are the processor retention statements (PostHog, Sentry, Anthropic, Google Play / RevenueCat) specific enough?
- **Why it matters:** Suggestions survive account deletion in unlinked form (brand/model, café name), with typed free text deleted.
- **Where:** policy section 7 (`[TO CONFIRM: retention period]`) and 10; migrations `20261112000100_user_machines_and_submissions.sql` and `20261112000200_cafes.sql` (unlink triggers).

## 9. Governing law

- **Question:** The privacy policy has **no governing-law section**; `terms.html` section 16 names Serbian courts with an EU-consumer carve-out. Should the policy add one (or refer to the terms), and should it name a supervisory authority (Serbia's Commissioner and the user's EU authority)? Is an EU representative needed under GDPR Article 27, since the controller is outside the EU?
- **Why it matters:** Users in the EU are targeted (store listing, EU hosting), which brings GDPR obligations to a non-EU controller.
- **Where:** policy (no section); `terms.html` section 16 "Governing Law & Dispute Resolution".

## 10. Other open markers on the page

- **Release date** (policy header, `[TO CONFIRM: refine-v2 release date]`): set on publish.
- **`user_machines` sync** (policy 3.10 marker): the table exists, the app keeps the list on the phone. If sync is switched on, the policy must say the list is linked to the account and deleted with it.
- **Legal bases for the café map** (section 4, `[TO CONFIRM: legal check of these bases.]`): consent for location (phone permission prompt); performance of a contract for tiles and suggestions.
- **In-app search history** (worksheet QUESTION): free Volt web search sends the query via our server to Google Custom Search; does Play's "in-app search history" apply, and is Google a processor here?
- **Barcodes** (worksheet QUESTION): only the barcode number goes to Open Food Facts; we believe nothing needs declaring.
- **Sentry**: may log the IP address; "anonymized" in section 10 should be checked against Sentry's settings.

## 11. Play Data safety answers left unchanged on 2026-11-14

The founder updated the Play form on 2026-11-14 (`play-data-safety.md`, "Submitted to Play on 2026-11-14") but left these answers as they were until this review:

- **"Shared" ticks.** User IDs, App interactions, Device or other IDs, Crash logs, Diagnostics and Other in-app messages are declared as shared. Under Google's definitions, a transfer to a service provider processing data on our behalf is not "sharing". Do PostHog, Sentry, Supabase (and Anthropic, for Volt) qualify as service providers under our agreements, so that these ticks can be removed? Or should they stay, to be cautious? Why it matters: "shared" shows on the store listing as "may be shared with other companies".
- **Name: required.** The display name is optional in the app (`context/AppContext.tsx` `signUp`, `updateProfile`). Should the form say optional?
- **Device or other IDs: required.** The PostHog device ID is sent only while analytics is on: on by default with opt-out in the live builds, opt-in from refine v2 (`lib/analytics-core.ts`). Should it be optional, like App interactions?
- **Messages > Other in-app messages** (collected, shared, ephemeral, optional). Volt Pro (Anthropic) is not live (`PRO_PAYMENTS_LIVE` false), and the free Volt runs on the phone apart from web search. Should this row stay, change, or wait for Volt Pro?
- **App activity > In-app search history** (not declared). The free Volt web search sends the query through our edge function to Google Custom Search (`lib/volt-api.ts` `fetchFreeSearch`, `supabase/functions/volt-chat`, `_shared/search.ts`). Does this need declaring, and is it shared?

## 12. Precise location

- **Question:** The refine-v2 build requests `ACCESS_FINE_LOCATION` as well as `ACCESS_COARSE_LOCATION` (`app.json` `android.permissions`). The precise position never leaves the phone (`lib/use-device-location.ts`): it centres the map and sorts by distance. The tile provider sees only the map area shown (zoom 14, `components/cafes/CafeMap.tsx`). Is "Precise location: not collected" correct on the Play form and in the policy? Or does requesting the fine permission, or tiles centred on the user, require declaring it?
- **Why it matters:** it changes the store listing and the policy. The safe alternative is to request `ACCESS_COARSE_LOCATION` only; the map still works with approximate centring.
- **Where:** `app.json` (`android.permissions`, `blockedPermissions`, `expo-location` plugin: no background location); `play-data-safety.md` "Release checklist: café map build to Play"; policy section 3.9.

## 13. Eligibility age and age confirmation

- **Question:** Is 16 the right minimum age for Voltresso? It matches the Play Console target audience ("16 and older"). Should the App ask users to confirm their age at sign-up (a checkbox or a date of birth), or is the statement in the terms enough? Do the digital-consent ages in the target countries change this (Serbia, Egypt, EU member states between 13 and 16)?
- **Why it matters:** the terms said 18 and the policy said "not directed at children under 18"; both drafts now say 16. The App has no age question or age gate. Analytics consent from a minor may need a parent where the local age is above 16.
- **Where:** terms PR #3 Section 3; privacy PR #1 Section 9; `app/login.tsx` and `app/onboarding.tsx` (no age check); Play Console > App content > Target audience.

## 14. Map-data accuracy disclaimer and ODbL wording

- **Question:** Is the terms disclaimer enough (listings may be incomplete or wrong; not an endorsement or partnership; third-party map apps under their own terms)? Does the attribution "© OpenStreetMap contributors, Open Database License (ODbL)" with links meet the ODbL? Are bundled city packs a "Derivative Database" (share-alike) or a "Produced Work" (attribution only)? See also item 6.
- **Why it matters:** cafés are shown on a map with directions. Wrong or closed listings, and an implied endorsement, are the main risks.
- **Where:** terms PR #3 Section 10; privacy PR #1 3.9 and section 5 note; `lib/map-config.ts` `OSM_ATTRIBUTION`; `scripts/data/build-cafe-pack.mjs`.

## 15. Licence for machine and café suggestions

- **Question:** Is the licence in the terms right? It is worldwide, non-exclusive, royalty-free and **perpetual**, covering use, editing, combining with other data and publishing after review. Should it survive account deletion, given that the brand/model and café name stay after unlinking? Is the moderation wording adequate (we may edit, merge, decline or remove; 5 per day)?
- **Why it matters:** suggestions become part of the public catalogue and map, and outlive the user's account.
- **Where:** terms PR #3 Section 7 and 14; migrations `20261112000100_user_machines_and_submissions.sql` (rate limit, unlink trigger) and `20261112000200_cafes.sql`.
