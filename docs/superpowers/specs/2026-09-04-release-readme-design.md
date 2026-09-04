# Release repo README — design

**Date:** 2026-09-04
**Repo:** `hidemyemulator/hidemyemulator` (GitHub, private for now)
**Status:** approved in chat, awaiting spec review

## Goal

Turn the blank org template into the public face of Hide My Emulator on GitHub: the place
release APKs will be uploaded and the README people read before they download. Model it on
`Xposed-Modules-Repo/com.wowsoftware.hidemyandroid` (the sibling product's repo), English only.

Releases themselves are out of scope for this pass ("release thì tính sau"). The README is the
deliverable.

## Deliverables

| Path | What |
|---|---|
| `README.md` | Replaces the template README. English only, structure below. |
| `SUMMARY` | One line, no trailing newline required: the module description modules.lsposed.org shows. |
| `images/thumbnail.png` | Banner at the top of the README. |
| `docs/superpowers/specs/…` | This document. |

`.gitignore` stays as is. No LICENSE file (the reference repo has none; the app is proprietary).

GitHub repo metadata (set with `gh repo edit`, approved by the owner):

- description: `Hide My Emulator`
- homepage: `https://hidemyemulator.com`
- topics: `android`, `lsposed`, `xposed`, `emulator`, `magisk`, `anti-detect`

Visibility stays **private**. Flipping it public is decided together with the first release.

## Sources of truth

Copy comes from the landing site so both places say the same thing:

- `hidemyemulator-landing/src/locales/en.json` — `hero`, `footer.description`, `features.items`,
  `howItWorks`, `howToInstall`, `faq.items`, `pricing`.
- `hidemyemulator-landing/src/data/site.ts` — every URL and name constant.
- `hidemyemulator/app/build.gradle.kts` + `AndroidManifest.xml` — package, minSdk, module facts.
- Reference README — section order, Important Notice / Disclaimer / Ongoing Updates /
  Feature Requests wording.

Constants used in the README (values from `site.ts`, two of them still marked `TODO(owner)` on
the landing and used as-is until the owner changes them):

| Constant | Value | Confirmed |
|---|---|---|
| Website | `https://hidemyemulator.com` | yes |
| Install guide | `https://hidemyemulator.com/en/how-to-install/` | yes (landing route) |
| Releases | `https://github.com/hidemyemulator/hidemyemulator/releases/latest` | yes |
| Telegram | `https://t.me/wowareofficial` | **owner to confirm** |
| Support email | `support@hidemyemulator.com` | **owner to confirm** |
| HME License on Google Play | `https://play.google.com/store/apps/details?id=com.wowsoftware.hmelicense` | package from manifest `<queries>` |
| Magisk | `https://github.com/topjohnwu/Magisk/releases` | yes |
| LSPosed | `https://github.com/JingMatrix/LSPosed/releases` | landing value |
| Sibling | `https://www.hidemyandroid.com` | yes |
| Package | `com.wowsoftware.hidemyemulator` | yes |

## README structure

Top to bottom. The reference wraps each language in a `## English` block with `###` headings.
There is one language here, so sections use plain `##` headings and no language switcher.

1. **Banner** — `![Hide My Emulator](images/thumbnail.png)`.
2. **Badges** — two static shields.io badges exactly like the reference (`Download`, `Latest
   Release`), both linking to `releases/latest`. Static, not the dynamic `github/v/release`
   badge, so nothing shows an error before the first release exists.
3. **Intro line** — `footer.description`: "An LSPosed module that hides your emulator, so apps
   think they run on a real phone." Then `hero.subtitle` (two sentences: apps can tell they run in an
   emulator; Hide My Emulator makes them see a real phone instead).
4. **Support / Discussion** — Telegram, website, install guide, support email, one per line as
   in the reference.
5. **Requirements** — bullets:
   - Android 9.0+ (API level 28 or newer)
   - A rooted Android emulator (Magisk or any other root method; Magisk is not required)
   - A working LSPosed environment
   - Developed and tested on the Android Studio emulator; other emulators that can be rooted and run
     LSPosed follow the same steps (from `faq.items.emulators`)
   - "If you are not familiar with Xposed modules, this project may not be suitable for your
     setup." (reference wording)
6. **How It Works** — one short paragraph from `faq.items.data` + `howItWorks` copy: only the apps
   you tick in the LSPosed scope see a real phone; other apps are not touched; app data and
   accounts stay as they are; masking applies the next time a ticked app opens.
7. **Feature List** — the twelve `features.items` as `**Title** — description` bullets, in
   `FEATURE_KEYS` order (build, traces, sensors, battery, input, telephony, camera, location,
   gpu, cpu, profile, scope).
8. **Getting Started** — the four `howItWorks.steps` (prepare, install, choose, relaunch) as a
   numbered list, then "Full guide with screenshots: <install guide URL>".
9. **Premium** — there is no free tier. One paragraph: Hide My Emulator is a paid module; protection
   works with an active Premium subscription managed by the HME License app on Google Play (install
   it, subscribe, Hide My Emulator picks up the status, cancel any time from Google Play
   subscriptions). Then one line: Premium includes everything in the feature list, updates while
   subscribed, and email and Telegram support. **No price, no table.**

   Revision 2026-09-04: the first draft had a Free vs Premium table copied from the reference
   repo's shape; the owner corrected that there is no free tier, so the table was dropped.
10. **Important Notice** — reference wording ("System-level modification always carries risk.
    Please back up your emulator image and important data before use."), with "ROM" replaced by
    "emulator image".
11. **Disclaimer** — reference wording with the product name swapped.
12. **Ongoing Updates** — reference wording with the product name swapped.
13. **Feature Requests** — reference wording; point to Telegram or the support email.
14. **Footer line** — "From the makers of [Hide My Android](https://www.hidemyandroid.com)."

## Copy rules

- Plain words, as on the landing: "a real phone", "emulator settings and files", "phone
  processor". Never name the shipped device profile, an emulator internal, or the hook layer.
- The README must pass `findBannedTerms` from
  `hidemyemulator-landing/scripts/copy-terms.mjs` with zero hits.
- No detection guarantees. The only hedge allowed is the FAQ's "no tool can promise 100%
  forever", and it is not needed in the README.
- No prices, no version numbers, no dates — nothing that goes stale between releases.

## SUMMARY

`An LSPosed module for Android emulators that makes the apps you pick see a real phone instead of an emulator.`

## Thumbnail

Source: `hidemyemulator/marketing/mockups/png/hero.png` (3200×1800, a mockup, not a screenshot).
Chosen over `og.png` because it has no drawn-in "Download APK" button and carries the checklist
of what apps see. Resize to 1600 px wide with `sips`, keep PNG, target under 1 MB. If PNG cannot
get under 1 MB at 1600 px, drop to 1400 px rather than switching format.

## Release conventions (recorded for later, not implemented now)

When releases start, follow the Xposed-Modules-Repo rules so the repo can be mirrored or
submitted to modules.lsposed.org unchanged:

- tag `<versionCode>-<versionName>` (the app is at `1-1.0`)
- release title `<versionName>`
- asset `release-<versionName>.apk`, signed with the release keystore
- release notes as a short bullet list
- repo description must be the module name (already `Hide My Emulator`)

## Out of scope

- Cutting a release, a release script, or CI.
- Making the repo public.
- Updating `DOWNLOAD_URL` on the landing to point at `releases/latest` (natural follow-up once a
  release exists).
- Submitting to modules.lsposed.org (needs a repo named after the package under
  Xposed-Modules-Repo).
- Translations.

## Verification

1. `findBannedTerms(README.md)` returns `[]`.
2. Every `http(s)` URL in the README answers with a non-error status, except `releases/latest`,
   which is expected to 404 until the first release.
3. `images/thumbnail.png` is under 1 MB and 1600 px wide.
4. `SUMMARY` is a single line.
5. `gh repo view hidemyemulator/hidemyemulator --json description,homepageUrl,repositoryTopics`
   shows the values above.
6. Work is committed on branch `readme`; `main` is untouched until the owner merges.
