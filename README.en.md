<div align="center">

# Retro Unity Android Game Modding · AI Agent Skills

### 改造老 Unity 安卓游戏 · AI Agent 技能包

**In-place dex string patching · Channel SDK stripping · DLL localization · Asset text · Chinese fonts · Automated verification**

Six skills, each hard-won on a real localization project — together they cover the
complete workflow of modding an old Unity Android game.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg?style=flat-square)](./LICENSE)
![Skills](https://img.shields.io/badge/skills-6-8957e5?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Android%20%7C%20Unity%204.x--5.x-3ddc84?style=flat-square&logo=android&logoColor=white)
![Lang](https://img.shields.io/badge/lang-%E4%B8%AD%E6%96%87%20%2F%20English-1f6feb?style=flat-square)
[![Last commit](https://img.shields.io/github/last-commit/lueuru/game-modding-skills?style=flat-square)](https://github.com/lueuru/game-modding-skills/commits)
[![Stars](https://img.shields.io/github/stars/lueuru/game-modding-skills?style=flat-square)](https://github.com/lueuru/game-modding-skills/stargazers)

**[English](#) · [中文](./README.md)**

</div>

---

## What is this

A set of **AI Agent Skills** for one specific, stubborn scenario:

> You have an old Unity Android game (Mono / .NET 3.5). You want to localize it, strip a
> channel SDK's popups, or change a single English string — but the classic
> decompile → smali → repack → re-sign route has you one typo away from a crash loop.

Six skills cover every stage of that workflow, and they share exactly one design principle:

**If an in-place, equal-length patch can do it, never repack. If a string must be judged
before it is touched, never blind-replace.**

One wrong byte in an old game can mean instant crash on launch, so every skill here is
"zero-length-change" school: file length unchanged → offsets unchanged → signature and
checksum recomputation shrinks to a couple of spots. The failure surface collapses.

## Skill overview

| Group | Skill | What it solves | Typical case |
|---|---|---|---|
| **Foundation** | `android-dex-string-patch` | In-place equal-length dex string patching | Put your own credit in the launch Toast |
| **Foundation** | `apk-channel-sdk-strip` | Channel SDK entry-point stripping | Real-name / ad popups on launch |
| **Foundation** | `android-emulator-autotest-setup` | Unattended "can this APK run?" verification | Patched APK, no test device |
| **Localization trio** | `unity-dll-chinese-patch` | DLL text layer (#US heap) | Crashes after patching / text that won't translate |
| **Localization trio** | `unity-resource-text-localization` | Asset text layer (level0 etc.) | Button text reverts to English / features break |
| **Localization trio** | `unity-bmfont-chinese-font` | Bitmap font sinicization | All-tofu boxes after translating |

## Install

Skills must live under your Agent's skill root. Example for WorkBuddy (adjust the path to your setup):

**Clone the whole repo (recommended — all six skills)**

```bash
git clone https://github.com/lueuru/game-modding-skills.git
cp -r game-modding-skills/skills/* ~/.workbuddy/skills/
```

**Grab just one (each skill is self-contained)**

```bash
cp -r game-modding-skills/skills/unity-dll-chinese-patch ~/.workbuddy/skills/
```

Once installed, the Agent auto-loads a skill when a trigger scenario matches — or you can
call it by name, e.g. "use `unity-dll-chinese-patch` on this DLL".

## Skill details

### android-dex-string-patch

Rewrite `classes.dex` strings inside a packaged APK (typical case: launch Toast credit /
Chinese copy) with **no recompilation, no bytecode changes, no length change**: pick a
sacrificial equal-length slot, rewrite in place, patch only the 2-byte operand of the
`const-string` instruction, then recompute signature and checksum.

Two hard constraints that cause instant-crash-on-tap (derivations are in the skill; here are the conclusions):

1. `utf16_size` counts **UTF-16 characters, not UTF-8 bytes** — on pure ASCII the two happen
   to be equal, so a bytes-based bug is invisible on English strings and **crashes the app
   the moment you insert Chinese**;
2. The string index table must stay in **lexicographic order**, and Chinese codepoints sort
   above every ASCII character — so Chinese strings cannot use arbitrary slots.

### apk-channel-sdk-strip

Channel builds (from Chinese Android marketplaces) get repacked with real-name verification,
ads, and analytics modules that pop up on launch. This skill **rewrites the AXML string pool
of `AndroidManifest.xml` in place, equal-length**, cutting the channel SDK's
`Application` / `Provider` activation entry points — no repack, no bytecode change.

Deleting code ≠ removing a feature: channel SDK activation is entry-point driven, so cutting
the entry is far safer than deleting code.

### android-emulator-autotest-setup

Set up an Android emulator plus a fully automated install / launch / crash-detection pipeline
on Windows, to verify unattended that an APK actually runs. Includes a **resource feasibility
precheck** (memory / disk / virtualization / hardware acceleration) — many environments simply
cannot run an emulator, and a two-minute precheck beats an hour of wasted setup.
Also provides a **static-verification fallback** when no emulator is available (parse APK
structure, verify offsets, inspect strings).

### unity-dll-chinese-patch

Parse the `#US` string heap of `Assembly-CSharp.dll` / `*-firstpass.dll`, use **IL context** to
classify each English string as "UI display text" (translatable) or "code reference /
comparison value" (do not translate), then apply **block-level in-place replacement** (zero
length and offset change).

- Offsets must come from parsing PE metadata, **never from heuristic scanning** — guessing is
  off by 30+ bytes and yields garbled text like `敩r杲b` *without any error*;
- The same string can have a **dual identity** (display text *and* a lookup key compared with
  `==`) — a blanket rule breaks both sides; judge every string;
- Ships two scripts: `scripts/il_logic.py` (IL-context classification), `scripts/fix_dll_logic.py` (write-back).

### unity-resource-text-localization

Scan string fields in `level0` / `sharedassets*.assets`, build a **forbidden-translation key
list**, and write translations back. Fixes "button text reverts to English after patching",
"some text stays English", and "crashes / broken features after translating".

The core is the forbidden-translation key list: old games drive logic through string
comparison (e.g. `if (shipData.shipclass == "Type XXI")`) — translating those deletes features.

### unity-bmfont-chinese-font

The original BMFont atlas only covers ASCII, so everything turns into tofu boxes after
translating. This skill builds a **2048² atlas + `.fnt` descriptor** and writes it back into
`Texture2D` / `TextAsset` inside `sharedassets*.assets`.

- The charset is "characters **actually used** after localization" — zero missing glyphs, and a
  larger, crisper font size than cramming in a full CJK set;
- **Must be done last**: do it earlier and you are forced into the full set, shrinking the font.

## Shared discipline

None of this is copied from documentation. Every line was learned the hard way on a real
legacy Unity game localization project:

- **Equal-length / in-place = zero risk**: length unchanged → offsets unchanged → checksum
  recomputation stays in two spots;
- **Judge before touching**: translatability, forbidden keys, and slot validity are all
  checked before anything is written;
- **Parse metadata for offsets**: heuristic scanning silently returns wrong answers (garbled,
  no error);
- **Criteria must resist false passes**: when a verification script reports "passed", first
  break it on purpose once to confirm it can actually go red.

## Scope

**Fits**: text localization, font replacement, credit injection, channel SDK stripping, and
unattended verification for **Unity 4.x ~ 5.x (Mono / .NET 3.5) Android games**.

**Does not fit**:

- Packed / encrypted assets (unpack first)
- IL2CPP builds (no `Assembly-CSharp.dll`; the path is `global-metadata.dat`, entirely different)
- **Logic-level** modifications (recompile route instead — in-place patching cannot do that)
- iOS (a completely different toolchain)

## Compliance notice

These skills are for apps **you own or are authorized to modify**. Stripping channel SDK
popups or replacing credits must happen under your right to modify that app;
**do not use them to tamper with and redistribute someone else's app**.

## Related repositories

- **[agent-engineering-skills](https://github.com/lueuru/agent-engineering-skills)** — AI Agent engineering practices — acceptance criteria that resist false passes, GitHub repo beautification, Windows cleanup audit, DSH troubleshooting and session review

## License

[MIT](./LICENSE) — free to use, modify, and redistribute; keep the copyright notice.

</div>
