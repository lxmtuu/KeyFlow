# Changelog · Keyflow

> **Bản tiếng Việt: [CHANGELOG.md](CHANGELOG.md)** · This is the English edition of the same document. The
> two files carry the same content and **the same list of versions**; `tools/check_sources.py`
> (`scan_release_version`) compares that list in both against `<Version>` in `PianoPath.csproj`, so neither
> edition can drift from the other and a new entry cannot sit in the repository without being in a release.

Everything a user can see is recorded here, following [Keep a Changelog](https://keepachangelog.com/en/1.1.0/)
and [semantic versioning](https://semver.org/).

The version number is written down in **one** place in the code — `<Version>` in `PianoPath.csproj` — and the
application reads it back through `AppInfo.Version` (the start-up menu's version label, the About box, the
`--bench` report). The copies that cannot read an assembly at run time (the fallback in
`installer\Keyflow.iss`, the `git tag` and `/DAppVersion=` examples in both READMEs, the newest entry of both
of these files) are pinned to the same number by `scan_release_version`: forgetting one turns the static
check red instead of shipping a release that states two versions.

The product's technical limits live in the README's *[Current limitations](README.en.md#current-limitations)*
section and are not repeated here; the open work lives in [`docs/ROADMAP.md`](docs/ROADMAP.md).

## 1.0.0 — 2026-10-02

The first official release. Keyflow is a Windows desktop application (C# · WPF · .NET 10) to **play, practise
and make piano videos from MIDI** at concert-performance quality, with an interface in **English and Tiếng
Việt** that switches while the application runs. The whole v0.5–v0.8 scope of `docs/ROADMAP.md` has landed;
this entry describes the product as released rather than listing every commit (no release was cut before
1.0.0, so the detailed history is the commit log and the pull requests).

### Stage & graphics

- A **Direct3D 11 GPU engine** (Vortice) draws the stage on its own thread: 16-bit HDR, multi-layer bloom, a
  frame target of 60/120/144/240 FPS or unlimited, and it never blocks MIDI input. The HLSL shaders are
  embedded in the assembly and compiled at start-up with the `d3dcompiler_47.dll` Windows already ships.
- 88-key keyboard shading after the Unreal model: Cook-Torrance GGX + softbox + ACES filmic tone mapping;
  sparks that cool by thermal radiation, acoustic resonance waves, fire at the strike point, acoustic dust,
  petals and stage lights — each layer switchable on its own.
- The **WPF renderer** (multi-threaded CPU shaders) is the fallback when Direct3D will not start, and the
  path the transparent PNG recording draws on; the General page says which engine is running and why.
- **14 built-in presets** and **3 community presets** embedded in the assembly (`presets/*.json`); user
  presets keep a thumbnail the stage rendered itself; a **look-sharing code** `KEYFLOW-LOOK-1:…` (gzip JSON in
  base64url) carries a look to another machine, drops the background-image path and re-`Clamp()`s on decode.
- Three interface themes (Concert Grand, Concert Noir, Velvet Gold) plus a user **theme studio**: five base
  colours derive twenty tokens, stored in `<settings>/themes/*.json`; Windows high contrast is detected and
  repainted the moment the system changes.

### Audio

- A hand-written **SoundFont 2** synthesizer with no extra dependency: presets/zones/samples, generator
  semantics (instrument overrides, preset accumulation), loops, the release stage, a polyphony limit and
  sustain/sostenuto/soft.
- The bundled SoundFont: FreePats' **YDP Grand Piano** (Yamaha Disklavier Pro multisamples), licensed
  **CC BY 3.0**, ~113 MiB through Git LFS; `Assets/ATTRIBUTION.txt` carries the credit and ships in every
  package.
- `waveOut` output; a machine that cannot open an audio device still runs everything else and says exactly
  why.

### Import & sheet music

- **MIDI** format 0/1/2, both time-division kinds (PPQ and SMPTE), tempo maps, the percussion channel, track
  names, a beat grid grouped per measure (`Meter.Of`: 4/4, 3/4, 2/2, 3/8 count the written unit; 6/8, 9/8,
  12/8 count groups of three; 7/8 still counts seven), and corrupt files are refused.
- **MusicXML** `.musicxml`/`.xml`/`.mxl` (compressed read through `META-INF/container.xml`), a cursor kept per
  part/measure against `divisions`, `backup`/`forward`, tempo changes mid-piece, chords sharing one onset, and
  a hand-split point read from `<staff>`.
- A **sheet-music layer** drawn on the stage: the piece's key signature inferred (Krumhansl–Kessler
  correlation), accidentals spelled per measure, short notes beamed to the beat, rests written for the silent
  hand, ties across a barline, chords merged into one stemmed column, a slur where the line changes hands,
  and ledger lines only when a note leaves the staff.
- **Hand-split inference** from the piece itself: two clusters by sounding duration, accepted only with ≥5
  semitones of gap between the hands and ≥10% of the duration each, preferring the note nearest C4; a
  one-handed or overlapping piece keeps the user's choice, and the result is remembered per song.

### Practice

- Note-by-note scoring (hits/misses, accuracy, longest streak) that **keeps every graded note** of a run (up
  to 256) so the dock can draw a **ghost**: the best run above, the latest below.
- A **14-day chart** grouped by local day (a day without practice is still a zero row), in the dock and in the
  HTML report.
- **Automatic practice tempo**: more misses in a row than the threshold (1–6, default 3) steps down 5% to a
  floor of 50%, four correct notes in a row step up 2% and stop at 100% — every step goes through the tempo
  slider itself.
- **Practice history** appends one JSON line per run to `history/practice.jsonl` (the latest 200 runs, corrupt
  lines skipped); the **History** page prints the recent runs and the best run of the open song, with
  **EXPORT HTML** and **CLEAR HISTORY**.

### Recording

- **MP4 (H.264 + AAC)** written directly by the Media Foundation sink writer Windows already ships, with the
  audio inside the file: the stage's BGRA frames converted to NV12, a bitrate from size × fps, and the audio
  engine's PCM encoded to AAC through the shared `IAudioTrack` contract.
- **AVI** (MJPEG, or uncompressed RGB when the machine has no codec; 2 GB ceiling) with the audio in a
  16-bit stereo **WAV** beside it, plus the ffmpeg command line to mux the two.
- A **32-bit PNG sequence with alpha** for transparent-background post work, with `sequence.json` and the
  `libvpx-vp9` + `yuva420p` ffmpeg line that rebuilds it into an alpha WebM.
- Encoding runs in a **child process** (`--encode-take`) with a codec probe before the take and a plumbing
  probe when the take produced no file, so a codec that takes the process down still leaves the verdict; the
  recorder's open window is wrapped in a 10-second `HangGuard` and **falls back to AVI** with a message on a
  machine whose media stack hangs.
- A **camera overlay** (a live camera or a looping video file, placed by corner, mirrored, opacity, green
  key) and **hand tracking** with classical vision: a YCbCr skin-colour rule with its own sensitivity, a
  32×24 grid, a finger count from the column profile, and a lit band on the key the hand is over.

### Library, profile & design desk

- The Play dialog holds **RECENT** (the 12 latest songs with note count, track count, tempo, hand split, fall
  speed, playback tempo and preset — reopening restores all of them through the very sliders) and a
  folder-based **LIBRARY** (a bounded BFS of 3 levels/500 files read by the app's own readers, cached by size
  + modification time in `library-index.json`, per-song tags, search over title–file name–tags, and a folder
  watcher so the list follows the disk).
- A **settings profile** `Keyflow.profile.json` gathering the stage settings + language + theme, imported and
  exported on the General page; drag-and-drop of `.json` (a profile), `.mid`/`.midi` (open a song) and images
  (set the background) onto the window.
- **Undo/redo of 32 steps** on the design desk (`Ctrl+Z` / `Ctrl+Shift+Z` / `Ctrl+Y`); one slider drag is one
  step because the snapshot is committed on the auto-save timer's idle tick.
- **Dock search** with per-setting synonyms, token intersection, the setting name as the last safety net, and
  the matched part painted in the accent colour.

### Interface, languages & accessibility

- A dock of **13 pages in 4 groups** (STAGE DESIGN, SOUND & INPUT, SESSION, APP); the page catalogue lives in
  `Ui/SettingsPages.cs` and is compared against the XAML tab strip both statically and at run time.
- **Two languages**: 1,266 string keys, a complete Vietnamese table, switching on an open window with every
  label repainted at once; language/theme/preset ids stay English in the saved files, so switching language
  can never change what is on disk.
- **Accessibility**: `AutomationProperties.Name` on every glyph-only button and every generated row, Tab
  cycling inside the dock in the order the rows are printed, the `SystemColors` palette when Windows turns on
  high contrast, and at 1080×700 every page keeps its controls inside their card even when a word is longer
  in the other language.
- The shortcut card (F1), Escape leaving the topmost layer first, and a start-up menu with the language picker
  and a read-out of the current stage look.

### Packaging & distribution

- **`publish.ps1`**: one command to a single-file **self-contained** (ReadyToRun) or **framework-dependent**
  build for `win-x64` or `win-arm64`, with `-Zip`; the script refuses to publish while
  `Assets\ConcertGrand.sf2` is still a Git LFS pointer, because that build would start without its piano.
- A bilingual **Inno Setup installer** (`installer\Keyflow.iss` plus the 108-message partial translation in
  `installer\Languages\Vietnamese.isl`): it picks the language from Windows, creates Start Menu/Desktop
  shortcuts and an uninstall entry, and `tools/build_installer.ps1` turns any unexpected ISCC warning into an
  error.
- **`release.yml`**: pushing a `v*` tag produces a GitHub Release with **three ZIPs** (self-contained
  `win-x64`, framework-dependent `win-x64`, self-contained `win-arm64`), the **`win-x64` `.exe` installer**
  compiled from that same publish folder, and a **`SHA256SUMS.txt`** of every file so a download can be
  checked by the person who downloaded it (the packages are not code-signed). The published build is smoke
  tested with `--verify` before anything is attached.
- **`LICENSE.txt` (MIT) and `Assets/ATTRIBUTION.txt` (the SoundFont's CC BY 3.0) are inside every package** —
  the ZIPs and the installer's target folder — not only in the repository, which is what both licences ask of
  a copy.
- `build.yml` runs **one real publish** (framework-dependent, `-AllowLfsPointer` because CI does not fetch
  LFS) on every push and checks that the output holds `PianoPath.exe`, `LICENSE.txt` and `Assets\`: a defect
  in the packaging path now turns red at push time instead of waiting for a release to be cut.

### Verification

- **Three verification layers**:
  - `tools/check_sources.py` — static, runs on any machine, needs no .NET SDK. This round added **two
    rules**: `scan_release_version`, which pins the version number to one source (`<Version>` in
    `PianoPath.csproj`), and `scan_project_files`, which parses every MSBuild file (`.csproj`, `.pubxml`)
    the way MSBuild itself would, because an XML comment containing a double hyphen stops the whole
    project from loading.
  - `--verify` — **58 verification functions** on Windows: shader maths, MIDI, SoundFont, AVI/MP4/PNG,
    settings, dock, language, history, profile, accessibility, high contrast, the GPU engine on WARP and
    the perf gate.
  - **README pictures the application renders itself in CI** (one picture per subject per language, plus
    the preset gallery), never staged screenshots.
- **`tests/PianoPath.Tests`**: 133 xUnit test cases on plain `net10.0` (10 linked source files, about a minute
  on Linux) for MIDI, MusicXML, the hand split, the beat grid, WAV, the frame budget and the plain half of
  `Loc`.
- **The `--bench` perf gate**: every frame time measured at 1920×1080 for two scenes (the default settings at
  p95 < 8 ms, the heaviest built-in preset at p95 < 16 ms) on a machine with a real graphics card; on a
  software adapter (every CI runner) it only compares against the previous run and prints `NOTE`, because two
  runs of the same code on WARP have differed by nearly a factor of two.

### Known, and left for after 1.0.0

- **No code signing**: Windows SmartScreen still shows "Windows protected your PC" on first launch (the README
  says how past it). `SHA256SUMS.txt` lets a downloader check a file; it is not a signature.
- **No MSIX, no winget manifest and no in-application update check** — all three need an
  account/certificate or a decision about networking, while the product promises offline and no telemetry
  (`docs/ROADMAP.md` §4 #5).
- **A real MP4 file still waits for an ordinary Windows machine to confirm it**: CI runners hang inside
  `IMFSample::SetSampleTime`, so that check reports `SKIP` there; the application already falls back to AVI
  on a machine like that.
- **The 8/16 ms budget has not been confirmed by a measurement on a real graphics card** — the sandbox and the
  runners only have WARP. The same goes for the `win-arm64` package: CI publishes it, no ARM machine has run
  it yet.
- No *lite* build without the SoundFont yet (~113 MiB is about a third of the total download) —
  `docs/ROADMAP.md` §4 #4.
