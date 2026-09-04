# Release Repo README Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Replace the blank template with the public README, one-line SUMMARY and banner for the Hide My Emulator release repo, and set the GitHub repo metadata. Revised 2026-09-04 after owner review: no free tier (the Free vs Premium table became a Premium section) and Magisk is one root method, not a requirement.

**Architecture:** Three static files at the repo root (`README.md`, `SUMMARY`, `images/thumbnail.png`), no build step. Copy is lifted from the landing site so both say the same thing, and a throwaway check script (kept in the scratchpad, not committed) scans the README for banned emulator internals, required sections and dead links. Repo metadata is set once with `gh repo edit`.

**Tech Stack:** Markdown, `sips` (macOS image resize), Node ≥ 22 (to import the landing's `copy-terms.mjs`), GitHub CLI (`gh`, already logged in as the owner).

**Spec:** `docs/superpowers/specs/2026-09-04-release-readme-design.md`

## Global Constraints

- Work on branch `readme`; `main` stays untouched until the owner merges.
- English only. No language switcher, no translations.
- Copy rules: plain words ("a real phone", "emulator settings and files", "phone processor"). Never name the shipped device profile, an emulator internal or the hook layer. README must pass `findBannedTerms` from `/Volumes/Woware/Projects/2026/hidemyemulator-landing/scripts/copy-terms.mjs` with zero hits.
- No detection guarantees, no prices, no version numbers, no dates in the README.
- `images/thumbnail.png`: 1600 px wide PNG under 1 MB (fall back to 1400 px, still PNG, if 1600 px is over 1 MB).
- `SUMMARY`: exactly one line.
- Repo visibility stays **private**. Do not run `gh repo edit --visibility`.
- Commit messages end with `Claude-Session: https://claude.ai/code/session_015GfFJSsguj44uGN7qcTmfe`.

## File map

| Path | Responsibility | Task |
|---|---|---|
| `images/thumbnail.png` | Banner shown at the top of the README | 1 |
| `SUMMARY` | One-line module description for modules.lsposed.org | 2 |
| `README.md` | The public page: support links, requirements, how it works, features, getting started, Premium, notices | 3 |
| `<scratchpad>/check-readme.mjs` | Throwaway verifier for Task 3 (banned terms, required headings, live links). Not committed. | 3 |
| GitHub repo settings | description, homepage, topics | 4 |

Scratchpad directory for this session: `/private/tmp/claude-501/-Volumes-Woware-Projects-organizations-hidemyemulator/ecf99ec8-c635-4ff2-8c0a-1bef62cb195c/scratchpad`

---

### Task 1: Banner image

**Files:**
- Create: `images/thumbnail.png`
- Source (read only): `/Volumes/Woware/Projects/2026/hidemyemulator/marketing/mockups/png/hero.png` (3200×1800 PNG)

**Interfaces:**
- Produces: `images/thumbnail.png`, referenced by Task 3 as `![Hide My Emulator](images/thumbnail.png)`.

- [ ] **Step 1: Resize the source to 1600 px wide**

```bash
cd /Volumes/Woware/Projects/organizations/hidemyemulator
mkdir -p images
sips --resampleWidth 1600 /Volumes/Woware/Projects/2026/hidemyemulator/marketing/mockups/png/hero.png --out images/thumbnail.png
```

- [ ] **Step 2: Verify dimensions and size**

Run:
```bash
sips -g pixelWidth -g pixelHeight images/thumbnail.png
stat -f '%z bytes' images/thumbnail.png
```
Expected: `pixelWidth: 1600`, `pixelHeight: 900`, and a byte count under `1048576`.

If the byte count is 1048576 or more, re-run Step 1 with `--resampleWidth 1400` and verify again (expected `pixelWidth: 1400`, `pixelHeight: 788`, under 1048576 bytes).

- [ ] **Step 3: Look at it**

Open the file (the Read tool renders PNGs) and confirm it shows the emulator window on the Status tab beside the "Make your emulator look like a real phone" headline and the checklist, with no clipping.

- [ ] **Step 4: Commit**

```bash
git add images/thumbnail.png
git commit -m "feat: add README banner from the marketing hero mockup

Claude-Session: https://claude.ai/code/session_015GfFJSsguj44uGN7qcTmfe"
```

---

### Task 2: SUMMARY

**Files:**
- Create: `SUMMARY`

**Interfaces:**
- Produces: `SUMMARY`, read by modules.lsposed.org if the repo is ever mirrored there. Nothing else depends on it.

- [ ] **Step 1: Write the file**

```bash
cd /Volumes/Woware/Projects/organizations/hidemyemulator
printf '%s\n' 'An LSPosed module for Android emulators that makes the apps you pick see a real phone instead of an emulator.' > SUMMARY
```

- [ ] **Step 2: Verify it is one line**

Run: `wc -l SUMMARY && cat SUMMARY`
Expected: `1 SUMMARY` followed by the sentence above, verbatim.

- [ ] **Step 3: Commit**

```bash
git add SUMMARY
git commit -m "feat: add one-line SUMMARY for the module listing

Claude-Session: https://claude.ai/code/session_015GfFJSsguj44uGN7qcTmfe"
```

---

### Task 3: README

**Files:**
- Modify: `README.md` (replace the whole template file)
- Create (not committed): `<scratchpad>/check-readme.mjs`

**Interfaces:**
- Consumes: `images/thumbnail.png` from Task 1.
- Consumes: `findBannedTerms(text: string): string[]` exported by `/Volumes/Woware/Projects/2026/hidemyemulator-landing/scripts/copy-terms.mjs`.
- Produces: `README.md`, the final deliverable.

- [ ] **Step 1: Write the check script**

Create `<scratchpad>/check-readme.mjs` with exactly this content:

```js
// Throwaway verifier for the release-repo README. Not committed.
// Usage: node check-readme.mjs /path/to/README.md
import { readFileSync, existsSync } from 'node:fs';
import { dirname, join } from 'node:path';
import { findBannedTerms } from '/Volumes/Woware/Projects/2026/hidemyemulator-landing/scripts/copy-terms.mjs';

const file = process.argv[2];
const md = readFileSync(file, 'utf8');
const failures = [];

// 1. No emulator internals / hook layer / device profile names.
const banned = findBannedTerms(md);
if (banned.length) failures.push(`banned terms: ${banned.join(', ')}`);

// 2. Every section from the spec, in this order.
const required = [
  '## Support / Discussion',
  '## Requirements',
  '## How It Works',
  '## Feature List',
  '## Getting Started',
  '## Premium',
  '## Important Notice',
  '## Disclaimer',
  '## Ongoing Updates',
  '## Feature Requests',
];
let last = -1;
for (const h of required) {
  const i = md.indexOf(`\n${h}\n`);
  if (i < 0) failures.push(`missing heading: ${h}`);
  else if (i < last) failures.push(`out of order: ${h}`);
  else last = i;
}

// 3. Banner file exists next to the README.
if (!md.includes('](images/thumbnail.png)')) failures.push('banner not referenced');
if (!existsSync(join(dirname(file), 'images/thumbnail.png'))) failures.push('images/thumbnail.png missing');

// 4. Nothing that goes stale.
if (/\$\s?\d/.test(md)) failures.push('contains a price');
if (/\b(20\d\d|v?\d+\.\d+(\.\d+)?)\b/.test(md.replace(/Android 9\.0/g, ''))) failures.push('contains a version number or year');

// 5. Every http(s) link answers. releases/latest is allowed to 404 until the first release.
const urls = [...new Set([...md.matchAll(/https?:\/\/[^\s)\]>"']+/g)].map((m) => m[0]))];
for (const url of urls) {
  try {
    const res = await fetch(url, { method: 'GET', redirect: 'follow', headers: { 'user-agent': 'Mozilla/5.0' } });
    const ok = res.status < 400 || (url.endsWith('/releases/latest') && res.status === 404);
    console.log(`${ok ? 'ok ' : 'BAD'} ${res.status} ${url}`);
    if (!ok) failures.push(`link ${res.status}: ${url}`);
  } catch (e) {
    console.log(`BAD err ${url} (${e.message})`);
    failures.push(`link error: ${url}`);
  }
}

if (failures.length) {
  console.error('\nFAIL\n- ' + failures.join('\n- '));
  process.exit(1);
}
console.log('\nOK');
```

- [ ] **Step 2: Run it against the template README to see it fail**

Run:
```bash
cd /Volumes/Woware/Projects/organizations/hidemyemulator
node /private/tmp/claude-501/-Volumes-Woware-Projects-organizations-hidemyemulator/ecf99ec8-c635-4ff2-8c0a-1bef62cb195c/scratchpad/check-readme.mjs README.md
```
Expected: exit code 1 with `FAIL` and ten `missing heading:` lines plus `banner not referenced`.

- [ ] **Step 3: Replace README.md with the final content**

Overwrite `README.md` with exactly this (two trailing spaces at the end of the "Support" lines and the notice/disclaimer lines are intentional line breaks, as in the reference README):

```markdown
# Hide My Emulator

![Hide My Emulator](images/thumbnail.png)

[![Download](https://img.shields.io/badge/Download-2D333B?style=for-the-badge&logo=github&logoColor=white)](https://github.com/hidemyemulator/hidemyemulator/releases/latest) [![Latest Release](https://img.shields.io/badge/Latest%20Release-2F81F7?style=for-the-badge)](https://github.com/hidemyemulator/hidemyemulator/releases/latest)

An LSPosed module that hides your emulator, so apps think they run on a real phone.

Apps can tell when they run in an emulator. Hide My Emulator makes them see a real phone instead.

## Support / Discussion

Support: https://t.me/wowareofficial  
Official Website: https://hidemyemulator.com  
How to install: https://hidemyemulator.com/en/how-to-install/  
Email: wowareofficial@gmail.com

## Requirements

- Android 9.0+ (API level 28 or newer)
- A rooted Android emulator (Magisk or any other root method)
- A properly working LSPosed environment
- Developed and tested on the Android Studio emulator (AVD). Other emulators that can be rooted and run LSPosed follow the same steps.
- If you are not familiar with Xposed modules, this project may not be suitable for your setup.

## How It Works

Tick the apps you want to protect in the LSPosed scope. When one of them opens, Hide My Emulator changes what it sees about the device, so every detail describes one real phone. Apps you did not tick are not touched at all, and the data, accounts and files of the apps you did tick stay as they are. Masking applies the next time a ticked app starts.

## Feature List

- **Phone name and model** — Apps see a real phone brand and model, not an emulator.
- **Emulator settings and files** — Settings and files that only an emulator has are gone, just like on a real phone.
- **Sensors** — Sensors look like real phone parts, not the emulator's fake ones.
- **Battery** — Battery size, health and charging look like a real phone.
- **Touchscreen and keys** — The touchscreen and buttons carry real hardware names.
- **Phone radio** — The modem shows a normal phone version instead of nothing.
- **Camera** — Camera count and direction match a real phone.
- **Location** — A believable last location instead of an empty one.
- **Graphics** — The graphics chip shows a real phone GPU, not the emulator's.
- **Processor** — Apps see a phone processor, not the PC chip the emulator runs on.
- **Everything matches** — Every detail describes the same phone, so nothing gives it away.
- **Pick your apps** — Choose which apps get the real-phone look. Turn each part on or off with one tap.

## Getting Started

1. **Prepare your emulator** — Root the emulator ([Magisk](https://github.com/topjohnwu/Magisk/releases) or any other root method) and install the [LSPosed](https://github.com/JingMatrix/LSPosed/releases) framework.
2. **Install Hide My Emulator** — Install the APK, turn the module on in LSPosed Manager and tick the apps you want.
3. **Choose what to mask** — Turn on what you want hidden in the Status tab and tap Apply. It applies the next time the app opens.
4. **Relaunch the target app** — Force-stop and reopen the app. It now sees a real phone.

Full guide with screenshots: https://hidemyemulator.com/en/how-to-install/

## Premium

Hide My Emulator is a paid module. Protection works with an active Premium subscription, managed by the [HME License](https://play.google.com/store/apps/details?id=com.wowsoftware.hmelicense) app: install it from Google Play, subscribe, and Hide My Emulator picks up your Premium status automatically. Cancel any time from Google Play subscriptions.

Premium includes everything in the feature list, updates while subscribed, and email and Telegram support.

## Important Notice

System-level modification always carries risk.  
Please back up your emulator image and important data before use.

## Disclaimer

Use at your own risk.  
By installing or using Hide My Emulator, you are solely responsible for how you use it.  
The developers are not responsible for misuse, violations of laws/platform policies, account penalties, data loss, instability, or bootloops.

## Ongoing Updates

Hide My Emulator is actively maintained with continuous feature and stability updates.

## Feature Requests

Feature requests are welcome.  
If you need a specific capability, share your use case on Telegram or by email and we will prioritize based on community demand.

---

From the makers of [Hide My Android](https://www.hidemyandroid.com).
```

- [ ] **Step 4: Run the check and make it pass**

Run:
```bash
cd /Volumes/Woware/Projects/organizations/hidemyemulator
node /private/tmp/claude-501/-Volumes-Woware-Projects-organizations-hidemyemulator/ecf99ec8-c635-4ff2-8c0a-1bef62cb195c/scratchpad/check-readme.mjs README.md
```
Expected: one `ok` line per URL and a final `OK`, exit code 0.

Known acceptable outcome: if the Google Play URL for HME License returns 404, the listing is not published yet. Keep the link (the landing site uses the same one), note the 404 in the task report, and treat the run as passing only if every other line is `ok`.

- [ ] **Step 5: Confirm the two owner-unconfirmed links are the ones in the spec**

Run: `grep -n 't.me/\|support@' README.md`
Expected: `https://t.me/wowareofficial` and `wowareofficial@gmail.com`, nothing else. These are the values the owner approved with the note that they may change later.

- [ ] **Step 6: Commit**

```bash
git add README.md
git commit -m "feat: public README for the release repo

Claude-Session: https://claude.ai/code/session_015GfFJSsguj44uGN7qcTmfe"
```

---

### Task 4: GitHub repo metadata

**Files:**
- None in the repo. Changes the settings of `hidemyemulator/hidemyemulator` on GitHub.

**Interfaces:**
- Consumes: nothing from earlier tasks.
- Produces: repo description, homepage and topics visible on the GitHub page.

- [ ] **Step 1: Check the current values**

Run:
```bash
gh repo view hidemyemulator/hidemyemulator --json description,homepageUrl,repositoryTopics,visibility
```
Expected: description `hidemyemulator project`, empty homepage, `repositoryTopics: null`, visibility `PRIVATE`.

- [ ] **Step 2: Set description, homepage and topics**

```bash
gh repo edit hidemyemulator/hidemyemulator \
  --description "Hide My Emulator" \
  --homepage "https://hidemyemulator.com" \
  --add-topic android --add-topic lsposed --add-topic xposed \
  --add-topic emulator --add-topic magisk --add-topic anti-detect
```

- [ ] **Step 3: Verify**

Run:
```bash
gh repo view hidemyemulator/hidemyemulator --json description,homepageUrl,repositoryTopics,visibility
```
Expected: description `Hide My Emulator`, homepage `https://hidemyemulator.com`, the six topics `android`, `lsposed`, `xposed`, `emulator`, `magisk`, `anti-detect`, and visibility still `PRIVATE`.

No commit for this task; nothing in the working tree changed.

---

## Done when

- `git log --oneline main..readme` shows the spec, plan, banner, SUMMARY and README commits.
- `node <scratchpad>/check-readme.mjs README.md` prints `OK`.
- `gh repo view` shows the metadata above and `PRIVATE`.
- The owner merges `readme` into `main` and pushes; nothing is pushed by this plan.
