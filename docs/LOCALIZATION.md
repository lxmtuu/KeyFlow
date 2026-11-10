# Localization: architecture, conventions and how to add a new language

> **Bản tiếng Việt: [LOCALIZATION.vi.md](LOCALIZATION.vi.md)** · This English edition is the primary
> document; the Vietnamese edition follows it. The source files, tools and workflows named below
> (`Localization/`, `Ui/`, `tools/`, `installer/`, `build.yml`, `docs/ROADMAP.md`…) are paths in
> Keyflow's source repository.

This document describes how Keyflow translates its interface: **the key is the English wording in the
source code**, each language is **one table file**, and every label on screen **repaints itself when the
language changes**, without restarting the application. The catalogue of pages and navigation groups
that goes with it is in `docs/DOCK-NAVIGATION-AUDIT.md`; the directions to take next are in
`docs/ROADMAP.md`.

> Environment: the development sandbox has **no .NET SDK**, and WPF only compiles on Windows. The
> correctness of the string tables is therefore proven by `tools/check_sources.py` (static, any machine)
> and by `VerificationSuite` (`--verify`, Windows runner), not by the human eye.

## 1. Four principles

| # | Principle | Why |
|---|---|---|
| 1 | **The key is the English source text** (`"Falling notes"`, `"Preset “{0}” applied"`), not an id made up by a person | The source reads like the real interface; a missing translation falls back to English by itself instead of leaving an empty label; a translator who `grep`s the exact string in `Ui/`, `Stage/`, `Audio/` sees the context at once |
| 2 | **Ids are never translated.** Setting values, preset names, theme ids, Windows device names and paths stay in English; only the displayed *caption* is translated | A `visual-settings.json` and the `presets\*.json` files written on a Vietnamese machine must open on an English one; changing the language cannot break what has been saved |
| 3 | **A label registered with `Loc` repaints itself.** There is no "change the language, then restart" | The application is a performance tool: a user who has the design dock open, or is in the middle of a recording, must not lose the session over a change of language |
| 4 | **The string table is data, not logic.** Adding a language = one more table file + one registration line, with no change to a renderer | An `if (language == ...)` branch in drawing code is the road to interfaces that differ from one language to the next |

## 2. Files and their roles

| File | Role |
|---|---|
| `Localization/Loc.cs` | `AppLanguage`, `Languages` and the core of `Loc` (table lookup/format/diagnostics), plain .NET so that the portable test project can link it |
| `Localization/Localizer.cs` | The WPF part of `partial Loc`: registering live labels, the XAML marker and repainting the labels when the language changes |
| `Localization/Strings.English.cs` | **The inventory**: every key the application can print, mapped to itself. It is the translator's work list |
| `Localization/Strings.Vietnamese.cs` | The Vietnamese table: exactly the same set of keys, in the same order |
| `Ui/DeviceOption.cs` | The `(Id, Display)` record: the two rows "Computer keyboard only" / "No MIDI output" can be translated, while real devices, named by Windows, are left as they are |
| `Ui/MainWindow.Language.cs` | The entry point on the interface side: `ApplyLanguage`, `RetranslateSurfaces`, the headings of the navigation groups, the captions of the device combo boxes |
| `Ui/MainWindow.Settings.cs` | The **Application** page (SESSION & APP group) with the language chips; the dock's search box matches both English and Vietnamese |
| `Ui/MainWindow.xaml` | `local:Loc.Localize="True"` on every element that carries static text; the **INTERFACE LANGUAGE** card on the start menu |
| `tools/check_sources.py` | Static checks: the two tables must hold the same set of keys, the placeholders must be intact, **every string the source prints must be a key of the inventory**, and **every key of the inventory must still be held by a source file** (`scan_dead_keys`: a key that no source prints is a wasted translation — it lives on only in the two tables, and the next change of wording would leave a second copy of the same sentence beside it) |

## 3. The API in C# code

```csharp
Loc.T("Falling notes");                                   // look up a static string
Loc.F("Preset “{0}” applied", name);                       // template + arguments (CurrentCulture)
Loc.Set(caption, "Falling notes");                         // live label: changes with the language
Loc.Set(tooltip, "Reset fall speed", FrameworkElement.ToolTipProperty);
Loc.Format(status, "Fall speed {0}", value);               // live template, with arguments
Loc.Bind(label, () => Loc.T(name) + suffix);               // when the text has to be computed
Loc.Page(SettingsPages.Overview);                          // page/group names through the same table
Loc.OnChanged(RefreshChoiceCaptions);                      // surfaces rebuilt from scratch (not labels)
Loc.Apply("vi");  Loc.Refresh();                           // change the language, then repaint
```

Three things worth remembering when using it:

* **`Set`/`Format`/`Bind` register; they do not print at once.** The labels are held in a
  `ConditionalWeakTable` + a list of `WeakReference`s, so an element that has been removed is never
  repainted and is not kept alive either. That is why the code that creates a control *always* calls
  `Loc.Set`, even when the current language is English.
* **Wherever a text is *composed*, use a renderer** (`Bind`/`Format`), not `T` on the composed string.
  `Loc.T(preset.Description)` is valid because `Description` is an English key;
  `Loc.T(description + path)` is not, and the static check reports an unknown key.
* **Do not compare translated text.** To learn a state, compare the stored value
  (`_visualSettings.BackgroundMode == "Image"`), not `label.Text == "Image"`.
  `VerificationSuite` has been rewritten to this rule: it compares against
  `Loc.T("SALAMANDER GRAND · SETUP REQUIRED")` and `Loc.T(SettingsPages.Overview)`, not against
  hard-coded English strings.

### The marker in XAML

```xml
<TextBlock Text="LOOK" local:Loc.Localize="True" Style="{StaticResource EyebrowTextStyle}"/>
```

`Loc.Localize` is an attached property: when it is set to `True`, `Loc.Track` looks for a literal in the
order `TextBlock.Text` → `HeaderedContentControl.Header` → `ContentControl.Content` →
`FrameworkElement.ToolTip` → `Window.Title` and registers every property it finds. So
`<Button Content="A" ToolTip="Set the loop start at the playhead" local:Loc.Localize="True"/>`
translates the *tooltip* and leaves the letter `A` alone; a single character (a glyph, a sign, `↺`) is
skipped on purpose. The English literal stays in the XAML, so the markup reads naturally and a missing
translation simply prints English.

## 4. Adding a language (example: Japanese)

1. **Create the table** — copy `Strings.English.cs` to `Localization/Strings.Japanese.cs` and rename the
   class to `StringsJapanese`. The inventory already is the list of keys; keep its ordinal sort order.
2. **Register it** — in `Localization/Loc.cs`:

   ```csharp
   internal static readonly AppLanguage Japanese = new("ja", "Japanese", "日本語", StringsJapanese.Table);
   internal static readonly AppLanguage[] All = [English, Vietnamese, Japanese];
   ```

   That is all the code there is: the language picker, `--lang=ja`, `SettingsPages` and every page take
   their list from `Languages.All`.
3. **Translate by the checklist** — `Localization/Strings.English.cs` is the work list. Run
   `python tools/check_sources.py` after every round of translation: it lists exactly the keys the new
   table still lacks and the templates that have lost a `{0}`/`{1}`. An empty value counts as not
   translated; `--verify` reports that list as `UntranslatedKeys`.
4. **Check** — three layers, the cheapest first:

   ```powershell
   python tools/check_sources.py                                   # parity + key coverage (any machine)
   dotnet run --project .\PianoPath.csproj -c Release -- --lang=ja --show-settings --settings-tab=general
   dotnet run --project .\PianoPath.csproj -c Release -- --verify   # UntranslatedKeys must be empty
   ```

5. **Preview pictures (optional)** — `build.yml` renders the README pictures once per language: the
   `foreach ($lang in @('en', 'vi'))` loop runs every entry of `$shots` with `--lang=$lang` and writes
   it to `docs/previews/$lang/`. Add `'ja'` to that loop if you want pictures for the new language.

No step touches a renderer, `App.xaml`, `PianoVisualSettings` or `visual-settings.json`.

## 5. Conventions of the Vietnamese translation

| Kind | How it is handled |
|---|---|
| Technical terms | Kept as they are: MIDI, SMF, SFZ, FLAC, AVI, OBS, FPS, GPU, RGB, HEX, pedal, preset, theme, slider, timeline, note, hit/miss/streak |
| The product's proper names | Not translated: Keyflow, Concert Grand, Concert Noir, Velvet Gold, Neon Violet, Concert Gold, Sakura Nocturne…; `Moonlight Sonata` → `Ánh Trăng` and `Galaxy Voyage` → `Du Hành Thiên Hà` for the descriptive label, while the *id/stored name* stays in English |
| Upper-case headings | Upper-case Vietnamese with its diacritics: `KEYBOARD & SHORTCUTS` → `BÀN PHÍM & PHÍM TẮT`, `SESSION & CAPTURE` → `BUỔI TẬP & GHI HÌNH` |
| Form of address | Neutral and close: "bạn" (the everyday "you") for the user, not "quý khách" (the formal "valued customer"); short imperative sentences ("Chọn MIDI File", "Giữ phím để note ngân dài") |
| Templates | Keep the number and the indices of `{0}`, `{1}`; keep `\n`; keep the curly quotes “ ” and the `·` separator |
| Numbers and units | Formatted by `CultureInfo.CurrentCulture` (`0,5`, not `0.5`); the units `px`, `px/s`, `fps`, `%` stay as they are |
| File-dialog filter strings | Keep the extension pattern: `AVI video (*.avi)\|*.avi` |
| Labels on the stage | The `KEYFLOW` watermark and the note names (`C4`) are **part of the picture** and are not translated; `KEYS` / `PARTICLES` in the HUD are |

## 6. Deliberately not translated

| Item | Reason |
|---|---|
| Names of MIDI/audio devices | Windows supplies them; the two translated placeholder rows are enough |
| Preset file names, paths of background images | They are the user's data, not interface text |
| `A` / `B` / `↺` / `F1` / `Esc` / `Space` | Keys and glyphs are printed verbatim; so are the chips of the F1 card, because they describe real keys on the keyboard |
| Enumerated values in JSON (`"ChromaGreen"`, `"PerHand"`, `concert-noir`) | Durable ids, which already have their own `LegacyIds`/migration tables |
| The `KEYFLOW` watermark, the note name `C4` | They are content of the frame the user takes away to make a video |
| The `--verify` log and internal exception messages | Meant for developers, not for the screen |

## 7. Why not `.resx` / `ResourceManager`

`.resx` + satellite assemblies are the right choice for a large translation operation, but here they
would trade three things for one small benefit: every string would need an id (losing "the source is
the key"), a `ResourceManager` would have to be handed to every place that produces text, and the
translations would move out into `net10.0-windows` satellite assemblies while the published build uses
`-p:SatelliteResourceLanguages=en` to drop WPF's language resources. A static `Dictionary` file has no
life cycle of its own, needs no lazy loading and is compiled together with the source, so **no key
disappears silently** — `check_sources.py` sees every literal at build time.

Conversely, the two things `.resx` used to help with are things this project already does itself:
mandatory parity (section 8) and a work-list file for translators (`Strings.English.cs` itself).

## 8. Three layers of checks

| Layer | Where it runs | What it catches |
|---|---|---|
| `python tools/check_sources.py` | Any machine, **before** the build step in CI | The two tables hold exactly the same set of keys; English maps every key to itself; ordinal order; placeholders and `\n` intact; **every literal the source prints must be a key** (it follows the `Loc.*` calls, the row builders, the shortcut table, themes, presets, navigation pages and the XAML marker) |
| `--verify` → `VerifyLanguageSwitching` | Windows runner | No key is missing from a translation; `Languages.Find/Normalize` accept an id, a culture tag and a native name, and fall back to English; **changing the language on an open window** changes the real labels; the dock's search box finds a row in Vietnamese; no unknown key is left behind |
| README pictures rendered by CI | Windows runner | The language and the layout show in the pictures, so an empty label or an untranslated string is exposed right on the front page |

## 9. Still open

* **An unknown string is now an error, no longer a `NOTE`.** `VerifyLanguageSwitching` asserts that
  `Loc.UnknownKeys` is empty: a sentence the interface prints that is not a key of the inventory cannot
  be translated by any table, so it is a regression, not a piece of information about the environment.
  To add a new sentence, add its key to `Strings.English.cs` + `Strings.Vietnamese.cs` (in ordinal
  order) — the static check and `--verify` both report it if you forget. Names given by the user
  (devices, files, presets they saved) go to the screen verbatim without passing through `Loc`, so they
  do not count as unknown strings.
* Shortcut chips in the F1 card: the heading of each group and the description beside each chip go
  through `Loc.Set` and have keys in the inventory. The chip column itself is printed verbatim. For
  real keys (`A W S …`, `Space`, `A · B · ×`) that is the "real keys are not translated" convention;
  the five chips that describe an action rather than a key (`Click the keys`, `MIDI keyboard`,
  `Pointer idle`, `Drag the timeline`, `Practice modes`) are printed the same way, so they still read
  in English in every language — this is the part that remains open.
* `VisualPresets.SanitizeName` returns `"My preset"` as the **file name** when the user saves without
  giving a name; its displayed label is translated (`Preset của tôi`) because `DisplayName` goes through
  the table. A side effect: a preset the user names exactly like a key of the table (for example
  `Custom`) would be translated when printed — harmless, but `DisplayName` should be limited to the
  built-in presets (which is now the case).
* The description of a preset *saved by the user* is a composed string (`User preset · x.json`); it
  already follows the language at the moment the list is built, but a preset that was given a
  Vietnamese name should not be touched by `T()`.
* The documentation comes in two editions, and **English is the primary one**: the English file takes
  the plain name and the Vietnamese file carries `.vi` before the extension — `README.md` /
  `README.vi.md`, `CHANGELOG.md` / `CHANGELOG.vi.md`, `docs/LOCALIZATION.md` /
  `docs/LOCALIZATION.vi.md`, `docs/presentation/README.md` / `docs/presentation/README.vi.md`.
  (`README.en.md` and `CHANGELOG.en.md` are kept only as short pointers to the new names, for links
  published before the switch.) `scan_readme` in `tools/check_sources.py` scans **both** READMEs
  (pictures, anchors, internal links) and requires each edition to point to the other;
  `scan_cli_and_samples` checks the command-line table of each edition, so neither edition can be
  missing a switch that the other one documents.
* **The installer (`installer/`)** is the third surface with visible text, and it does not go through
  `Loc`: Inno Setup reads `installer/Languages/Vietnamese.isl` (UTF-8 **with a BOM**, so the compiler
  knows at once that it is Unicode), listed **after** `compiler:Default.isl` in `MessagesFile`, so the
  file only *overrides* the 108 messages the installer really shows — the rest follows the English of
  `Default.isl`. The three things that easily go wrong are all blocked: a message name that does not
  exist (Inno Setup only **warns** and drops the line, that is, it silently shows English), a message
  placed in the wrong one of `[Messages]`/`[CustomMessages]`, and a placeholder that got lost.
  `tools/inno_messages.py` generates `installer/Languages/messages.txt` (the valid names + their
  placeholders, taken from the `Default.isl` of three Inno milestones, 6.3.3/6.7.3/7.1.0),
  `scan_installer` in `tools/check_sources.py` compares the translation against that file, and
  `tools/build_installer.ps1` (run in both workflows) turns every other ISCC warning into a build
  error. The translation conventions are still those of §5 (`Huỷ`, `tệp`, `thư mục`, `shortcut`,
  `Start Menu`).
* RTL (Arabic, Hebrew, Persian) additionally needs `FlowDirection`, mirrored `Margin`s/`Grid` columns
  and a mirrored falling path for the notes; the table of an RTL language has to come together with
  that round of work and should not be added on its own.

## 10. Interface pictures: only pictures the application renders

`docs/previews/` **holds only pictures that `PianoPath.exe` itself renders in CI** (`--snapshot`), in two sets:
`docs/previews/en/` for `README.md` and `docs/previews/vi/` for `README.vi.md`. Each set captures the same list
of subjects (`$shots` in `build.yml`) in the language of the README edition that reads it (plus
`docs/previews/presets.jpg`, the gallery of every preset, built by a separate step of the same workflow and shared
by both editions). A user's screenshot — even a very beautiful one with a background picture of their own choice —
is not committed here, for two reasons: the picture would drift from the real UI at the very next change to the
interface, and the background in a screenshot is almost always third-party artwork that comes with no licence for
an MIT repository.

To have the documentation show a feature, make CI able to render that feature. The background picture is an
example that is already done: `tools/make_stage_background.py` **generates** `docs/samples/stage-backdrop.png`
(pure Python stdlib, runs in under a second, a deterministic picture because the starry sky is a seeded
catalogue), `--background-image=<file.png>` hangs that picture on the stage **for this run only**, and `build.yml`
captures `background-image.png`. The switch calls `MainWindow.PreviewBackgroundImage`, which sets the stage
directly instead of going through the handlers of the settings rows: it does not raise the "modified" flag, does
not arm the autosave clock, and a picture that cannot be read only leaves the stage to report the error in
`BackgroundLoadError` instead of opening a dialog — a capture run has nobody to press OK. `VerificationSuite`
checks exactly that contract, so the "look only, write nothing" behaviour cannot drift in silence.

`tools/check_sources.py` keeps three ends in line:

* every `![...]()` that points into `docs/previews/` must either exist already or be named in the workflow's
  `$shots` — it may be exactly one commit behind the README, because the very commit that renders the pictures
  catches up — **and** the `README.md` edition may only point into `docs/previews/en/`, the `README.vi.md`
  edition only into `docs/previews/vi/`, while the workflow must really render both languages (the
  `foreach ($lang in @('en', 'vi'))` loop);
* every command-line switch that `App.xaml.cs` or `VerificationSuite` reads must be in the README's *Command line
  switches* table and the other way round, and every switch + path that `build.yml` passes to `PianoPath.exe`
  must really exist — a mistyped switch does not break the build, it just silently produces a bad picture;
* `docs/samples/stage-backdrop.png` must match the `W, H` of the script that generates it: a hand-edited
  picture submitted to `docs/samples` is reported as an error instead of silently parting ways with the script.
