# GlyphCore

[Русский](README.md) · **English**

### Guild Wars 2 localization: non-Latin script rendering (Cyrillic, Korean) + text translation

> GlyphCore makes **Guild Wars 2** render the letters the stock client shows as empty
> boxes (Cyrillic, Korean Hangul, and more) and **translates the game text** into any
> language — **without modifying the on-disk `Gw2.dat`**.
> *Guild Wars 2 localization (Russian / Korean / …).*

<img width="237" height="247" alt="image" src="https://github.com/user-attachments/assets/2d33dddc-26aa-4c0f-bf60-b0dcd12c216c" />

It's an early-load proxy DLL. It teaches the live font engine to draw the needed glyphs
and swaps strings **in memory** — after the game has read and verified the genuine
data, but before it's shown on screen. `Gw2.dat` is left untouched, so ArenaNet's
archive integrity checks pass and no "repair" is triggered. Substitution is
**language-neutral**: you can put a translation into any language into the dictionary.

**What already works:**

- ✅ Non-Latin text (Cyrillic, Korean Hangul) renders without crashing.
- ✅ Text translation, including the previously unreadable encrypted (RC4) strings — ~70% of all game text.
- ✅ **In-game overlay** (the **Insert** key): dictionary editor, string collection and
  settings right in the game, with an interface-language switch (Русский / English / 한국어).
- ✅ String collection and custom glyph fonts survive game updates and don't crash the client.

> ⬇️ **Prebuilt files are on the [Releases](https://github.com/nourlie/GlyphCore/releases/latest) page.**
> No need to build from source.

> ⚠️ **A ready-made translation is NOT bundled.** You fill in the dictionary
> (`main_strings.csv` and/or `dictionary.bin`) yourself: extract the English text with
> the dump tool and translate it, or collect strings live in the game. Without a
> dictionary the glyphs render, but the text stays English.

---

## 📦 Installation

### Option A — the installer (recommended)

Run **`GlyphCoreInstaller.exe`** from the [latest release](https://github.com/nourlie/GlyphCore/releases/latest):

1. Point it at your Guild Wars 2 folder (it validates that `Gw2.dat` is there).
2. Check which translation categories to enable (or select all).
3. Click **"Install"** — it drops `version.dll` and `glyphCore\dictionary.bin`, and
   creates or updates `config.ini` (unchecked categories are disabled via
   `disabled_files`; your other settings are left untouched).

### Option B — manual

Download **`version.dll`** from the [release](https://github.com/nourlie/GlyphCore/releases/latest)
and drop it next to `Gw2-64.exe`. Launch the game — the glyphs render.

```
Guild Wars 2\
├─ Gw2-64.exe
└─ version.dll      ← the downloaded file
```

> `version.dll` exports are forwarded to the real system `version.dll` from `System32`
> automatically — no separate `version_orig.dll` copy is needed.

> 🔄 **Auto-update.** On launch `version.dll` checks the
> [GitHub releases](https://github.com/nourlie/GlyphCore/releases) and, if a newer one
> exists, quietly updates itself for the next launch (in the background, when online).
> **To roll back:** delete `version.dll` from the game folder.

---

## 🪟 In-game overlay (the Insert key)

The main way to work with the localization without leaving the game. Opened with the
**Insert** key: vertical tabs, dark theme, interface language switchable
(Русский / English / 한국어) on the *Interface* tab.

| Tab | What's inside |
|-----|---------------|
| **Settings** | status and general controls: translation on/off, auto-update, auto-collect, full seed-based collection, uploading found strings to the server |
| **Found words** | strings collected while playing, grouped by file; drag a word onto a category file to add it as a new row |
| **Dictionary editor** | edit any `dict_*.csv` / `pn_*.csv`, `main_strings.csv`, and `dictionary.bin` categories right in the game: click-to-edit, filters (all / translated / untranslated), search, progress, "Go to untranslated" jump. **Changes apply live immediately on save** — no restart |
| **Translation files** | toggle individual dictionary files/categories on or off, export **"bin → CSV"** / import **"CSV → bin"**, a **"Fold in found"** button (merges translated found strings into the dictionary), and coverage stats |
| **Interface** | overlay language (Русский / English / 한국어), HUD icon position and size, overlay font scale, **game glyph font selection and its size** (on the fly) |
| **Log** | diagnostic log |

> The editor supports **Ctrl+V / Ctrl+C**; Korean input renders with glyphs instead of "boxes".

> String collection (`autocollect` / `seed_decode`) can be toggled right in the overlay,
> no restart. Collection hooks are resolved by signature and survive game updates; the
> custom glyph font path is panic-hardened and can't crash the game.

---

## 📝 Game text translation

The translation comes from two sources:

- **`glyphCore\main_strings.csv`** — the main dictionary (columns `english,translate`,
  any target language; editable in a spreadsheet or the overlay);
- **`glyphCore\dictionary.bin`** — a compiled binary dictionary bundling many categories
  at once (built with the overlay's "CSV → bin" button, or installed by the installer).

The English string is replaced with the translation **in memory**, after the integrity
checks — the `.dat` is untouched. **Translation length is unlimited.** Encrypted (RC4)
strings — ~70% of all game text — are translated too.

> Substitution is **language-neutral**: the `translate` column can hold Russian, Korean,
> or any other language. By default `translate` auto-enables only on Russian-locale
> systems; elsewhere the game stays in its original English (enable it in the overlay or
> `config.ini`).

```
Guild Wars 2\
├─ version.dll
└─ glyphCore\
   ├─ dictionary.bin           ← compiled dictionary (many categories at once)
   ├─ main_strings.csv         ← main dictionary (english,translate)
   ├─ csv\                     ← dict_*.csv for offline editing / rebuilding the bin
   ├─ discovered_strings.csv   ← auto-collected new strings (temporary)
   ├─ seed_ids.csv             ← id↔string for seed collection (see below)
   ├─ cyrillic.ttf             ← your own font (optional)
   ├─ config.ini               ← settings
   └─ version_proxy_log.txt    ← log
```

> The data folder and dictionary were renamed (`cyrillic\` → `glyphCore\`,
> `cyrillic_strings.csv` → `main_strings.csv`); on first launch of a new version,
> `version.dll` migrates old files to the new locations itself.

### Where the dictionary comes from

**Full dictionary = base offline dump (raw) + RC4 collection in-game.**

- **Unencrypted text** (item names, part of the UI, the official wiki, the GW2 API) is
  extracted offline with the dump tool into `dict_<source>.csv` (format
  `english,translate`, merge — your translations are kept). `version.dll` loads all
  `dict_*.csv` from `glyphCore\csv\` together with `main_strings.csv`.
- **Encrypted (RC4) strings — ~70% of the text — are NOT extracted offline** (they need
  the game's runtime keys). They land in the dictionary **as you play** via string
  collection: `version.dll` decodes them on the fly and writes them down.

### Collecting strings as you play

`version.dll` decodes the strings the game shows (including encrypted ones) and appends
them to `glyphCore\discovered_strings.csv`; on the next launch they're folded into the
dictionary. Two modes (in the overlay or `config.ini`):

- **`autocollect`** — collects strings as they appear on screen.
- **`seed_decode`** — decrypts **all** strings sent by the server (~18× coverage).
  Decoding runs on a background thread so it doesn't stall the game; the hooks are
  resolved by signature and survive game updates.

> **Uploading found strings (optional).** On first launch the overlay offers to enable
> uploading found untranslated strings to a shared server — this helps build a community
> translation pool. Toggle it any time (`upload` in `config.ini`); only game text is sent.

### Count declensions — `[one|few|many]`

Some languages inflect nouns by count (Russian: 1 snowflak**e**, 3 snowflak**es**,
5 snowflak**es** all differ), which the English `[s]` cannot express. Put **three forms
separated by `|`** in square brackets — `version.dll` picks the right one by the count
the game supplies:

```
[form_for_1 | form_for_2-4 | form_for_5+]
```

| When | Which number | Form used |
|------|--------------|-----------|
| **one** | 1, 21, 31, 101 … (but not 11) | `[`**form1**`|…|…]` |
| **few** | 2–4, 22–24 … (but not 12–14)  | `[…|`**form2**`|…]` |
| **many** | 5–20, 0, 25–30 …            | `[…|…|`**form3**`]` |

**Example.** Keep the common stem **outside** the brackets and leave only the differing
endings inside:

```csv
english,translate
%num1% Snowflake[s],%num1% снежин[ка|ки|ок]
```

In the game it renders: `1 снежинка`, `3 снежинки`, `7 снежинок`. (Whole words work too
— `[снежинка|снежинки|снежинок]`; both are equivalent.)

> A couple of extra notes:
> - two forms `[ый|ые]` — adjective/suffix: the form for **1**, otherwise the second
>   (e.g. `золот[ой|ые] [ключ|ключа|ключей]`);
> - a single `[ы]`/`[и]` without `|` yields **the singular only** (the engine can't
>   change the word stem) — for counters always use three forms.

Declensions apply both on dictionary load and **live** when you save an edit in the overlay.

---

## 🅰️ Glyph font

By default the game glyphs are drawn with the bundled **Tahoma** font (covers Latin and
Cyrillic). For Korean (Hangul), `version.dll` additionally uses the **Hangul glyphs
already shipped inside the game itself**, so Korean renders across the whole UI without a
separate font. To render Cyrillic with **your own** font — two ways:

- **In the overlay:** the **Interface** tab → pick a `.ttf` and size. Applied on the fly
  (on the next font-asset load — a zone/map change), no restart.
- **Drop-in:** put a **`cyrillic.ttf`** file into the **`glyphCore\`** subfolder.

```
Guild Wars 2\
├─ Gw2-64.exe
├─ version.dll
└─ glyphCore\
   └─ cyrillic.ttf   ← your font
```
> If you put `cyrillic.ttf` (or an old `cyrillic_strings.csv`) directly into the game
> folder, on the next launch `version.dll` moves it into `glyphCore\` itself.

> [!IMPORTANT]
> **A custom font must contain the script's glyphs.** If a `.ttf` has no Cyrillic,
> `version.dll` rejects it and falls back to the built-in font — no crash. Rasterization
> is also hardened against internal failures of the font library: even a broken `.ttf`
> can't crash the game. Check `glyphCore\version_proxy_log.txt`.

> [!WARNING]
> **Don't trust the Windows font preview** — it shows Cyrillic even for fonts without it
> (system font fallback) while the file itself has no such letters. Use fonts with real
> Cyrillic coverage:
> [Roboto](https://fonts.google.com/specimen/Roboto),
> [Open Sans](https://fonts.google.com/specimen/Open+Sans),
> [PT Sans](https://fonts.google.com/specimen/PT+Sans),
> [Noto Sans](https://fonts.google.com/noto/specimen/Noto+Sans).

---

## ⚙️ Settings — `glyphCore\config.ini`

Changed **in the overlay** (the Interface tab) or by hand (created on first launch or by
the installer).

| Option | Default | What it does |
|--------|:---:|------------|
| `autoupdate` | `true` | checks and installs `version.dll` updates from GitHub |
| `translate` | *by locale* | shows the translation in-game (on for Russian systems, off otherwise) |
| `ui_lang` | *by locale* | overlay interface language: `ru` / `en` / `ko` |
| `autocollect` | `false` | collects shown strings into `discovered_strings.csv` |
| `seed_decode` | `false` | decrypts all server-sent strings (full collection), on a background thread |
| `translate_proper_nouns` | `false` | translates names/locations from the `pn_*.csv` layer |
| `upload` | *unset* | upload found strings to a shared server (asked on first launch) |
| `disabled_files` | *empty* | which dictionary files/categories to skip loading (`;`-separated) |

```ini
autoupdate=true
translate=true              # on only on a Russian-locale system, otherwise original English
ui_lang=ru                  # ru | en | ko (defaults by OS locale)
autocollect=false
seed_decode=false           # ~18× coverage over autocollect
translate_proper_nouns=false
upload=false
disabled_files=             # semicolon-separated list of disabled dict_*.csv / bin categories
```

> String collection, font selection, and most settings apply **on the fly** when changed
> in the overlay — no restart needed.

---

## 🖥️ GUI — dictionary editor outside the game

**`gw2-cyrillic-gui.exe`** — a standalone graphical app to work with the dictionary
outside the game (built from [`gw2-cyrillic-gui/`](gw2-cyrillic-gui/)). Put it next to
`version.dll` or run it from anywhere and point it at the GW2 folder — it finds
`glyphCore\` automatically (including by searching Steam libraries).

Tabs:

- **Dictionary** — an `English / Translation` table over `main_strings.csv`: search,
  filters (all / translated / untranslated / with broken markup), highlighting of
  untranslated rows and broken placeholders, checkboxes and bulk operations, undo
  (**Ctrl+Z**), progress and jump to the next untranslated row.
- **Settings** — edit `config.ini` in a couple of clicks, plus a theme.
- **Status** — parses `version_proxy_log.txt`: what loaded, whether the signatures were
  found, whether the dictionary is visible — for diagnostics.

> The GUI breaks nothing in the game: it only edits the dictionary and `config.ini` and
> reads the log. The translation is still applied by `version.dll`.

---

## 🔧 Advanced — `gw2-dat-tool` (optional)

The CLI [`gw2-dat-tool/`](gw2-dat-tool/) works with **your own** `Gw2.dat`: exporting the
string dictionary, inspecting `AFNT` font chunks, generating glyph atlases. Most people
**don't need it** — `version.dll` and the overlay/GUI are enough. Details in
[`gw2-dat-tool/README.md`](gw2-dat-tool/README.md).

```powershell
# export the whole string dictionary to CSV (game closed):
cargo run --release -- --dat "C:\path\to\Guild Wars 2\Gw2.dat" strs-export-all --out dict.csv
```

---

## How it works (in brief)

Guild Wars 2 stores fonts and text inside the `Gw2.dat` archive. The direct approaches
don't work: **editing `Gw2.dat`** is impossible (the client checks the CRC and deletes
modified files → re-download), and a **regular add-on** loads *after* the game has
already read the data.

`version.dll` sidesteps both. GW2 imports it statically, so it loads **before** the
engine; the game reads and verifies the *genuine* data, and then the DLL swaps the
**decompressed font and string bytes in RAM** — after all integrity checks, but before
the parser. The on-disk `Gw2.dat` is not modified. The overlay is drawn on top of the
game via a Direct3D hook. Key hooks are resolved by signature (not a fixed address), so
the mod survives most game updates without changes.

The source of the injected `version.dll` is closed; the releases ship a prebuilt binary.

---

## ⚠️ Legal / game data

The repository contains tools for working with the dictionary and Cyrillic data. It
contains **no** data extracted from `Gw2.dat` — that is ArenaNet's property; you obtain
the prebuilt files from *your own* legally installed copy of the game (via the dump tool)
or from the releases.

Guild Wars 2 is a trademark of ArenaNet, LLC. This is an unofficial, non-commercial fan
project, not affiliated with or endorsed by ArenaNet.

## License

[MIT](LICENSE) for the code in this repository.
