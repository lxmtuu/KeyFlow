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

The product's technical limits live in the release README's *[Current limitations](README.en.md#current-limitations)*
section and are not repeated here; the internal development roadmap is not part of the public release repository.

## Unreleased

## 2.1.1 — 2026-10-16

### Correct immutable package publication
- **First downloadable build of the 2.1 feature set:** includes all particle/AR/Settings features listed under 2.1.0 below. The source 2.1.0 release was published without assets and locked by immutability; it has no downloadable build and its tag is not moved, deleted or reused.
- A single tag-creation trigger avoids duplicate publication. Create a draft, upload the complete packages, then publish; immutability is not disabled and immutable releases are not overwritten.

## 2.1.0 — 2026-10-16

### Settings, particle effects and AR export
- **Settings now has 12 approved groups:** Notes & color → Press → Hold → Release → Lighting, with Advanced sections, accent-insensitive VI/EN search and aliases for all 13 legacy pages. Presets/share codes, camera/calibration, Audio/MIDI, practice/history, recording, themes and profiles retain their existing handlers/IDs. The Stage/AR header selects a settings context, not a camera/output mode.
- **Upgraded particle effects are opt-in**, off for existing looks. Wisps/Flames and Float Up/Dissolve/Smoke release materials are shared by GPU Live Stage and AR, with a role/note-group fair allocation of the AR cap of 192. Includes Prism Flow - Wisps / Prism Flow - Flames presets; software rendering keeps the classic effects.
- **Offline AR exports retain particle lifecycle state** throughout each take instead of recreating simulation on every frame, using the planned scene clock and substeps of at most 50 ms. The legacy half-rate camera-file policy remains; this is not a universal A/V-sync fix or webcam/ARM hardware certification.

### AR overlay
- **AR render-to-video — export UI (S7 complete)**: the Recording page gains an **AR RENDER‑TO‑VIDEO** card — RENDER AR TAKE… renders the loaded song over the calibrated camera video file offline: no live clock, a fixed time step, the machine's exact look (AR overlay, the kind‑16 watermark strip, every layer), the piano mixed tick-exact into a `.wav` beside the AVI (resolution cap and fps follow the OUTPUT choices; Record audio and Watermark are honoured), with the ffmpeg mux line printed on completion. Song → tick script is pure code (`ArVideoExportScript`, xUnit `SongExportScriptTests`); verify drives the real route with an MP4 written by the app's own recorder and decoded by its reader: the fixture's stripe brightens in step with every engine tick.
- **AR render-to-video — watermark**: the **Watermark** toggle's «KEYFLOW» strip is now drawn by the GPU itself (sprite kind 16 + the `MarkTex` texture), so the GPU screen recording and every offline take carry exactly what the live stage shows; the placement (centred, the stage's own 18 DIPs above the key line, shrinking only on narrow frames) is pure code with xUnit tests. Verify byte-compares one tick with and without the strip: the difference lights exactly the strip's rect and nothing else.
- **AR render-to-video — voice half**: every offline take can now also leave a `.wav` next to the AVI/PNG — the engine mixes the piano at exactly the video's ticks (note events fire with the frame they appear on, 44.1 kHz blocks pace each tick so the sample count lands sample-exact even at awkward rates like 13 fps), and a take without notes yields silence of the right length. Shape: an optional `audio` hook (synth + event script + `WavWriter`) through the engine's `Run`; the caller owns and closes the writer/synth. Verify drives a synthetic SFZ: 22050 samples for exactly six 12 fps frames, loud through the held ticks and near-silent right at the onset.
- **Release hardening:** the toolbar starts its idle clock at the first rendered frame, and live-bar coverage MORE is not hidden behind a second Advanced gate. Public publishing preserves the source/packages, stages through an isolated Git index and uses valid PowerShell append commands; all release test/native gates remain required.

- **AR render-to-video — the frame half** (roadmap §14.10, AR‑S7 part one): the groundwork for exporting the AR scene to video offline. `Video/ArVideoRenderProfile.cs` (pure) plans geometry/rate/frame count with every safe clamp — even sides the AVI container wants, the 60 fps ceiling of both existing writers, a long-side cap so an over-sharp camera cannot burn GPU for nothing — plus a 1/2/4 downsample choice over 2×2 box averages (`Video/BgraPack.cs`, pure). `Video/ArRenderToVideo.cs` is the offline sweep engine: given a scripted feed per tick, every output frame advances the scene by exactly `1/fps` at a fixed seed — no clock of its own — so two sweeps of one script are **byte-identical**, and any tick matches, pixel for pixel, a live-shaped render of the same tick at the same time step and seed (the roadmap's own "compare live ↔ render" contract). Writes go through the existing `AviVideoRecorder`/`PngSequenceRecorder` at exactly the planned size and rate. Checked by the two pure xUnit suites (`VideoRenderProfileTests`, `BgraPackTests`) and the `VerifyArRenderToVideo` block in `--verify` — the AVI take at 200×112@12 fps reads back with the camera's left stripe and the C4 key light on exactly the pressed ticks, the PNG take boxes 320×180→160×90 across six frames plus manifest. Song audio (mixdown), the GPU-side watermark and the export UI belong to the rest of AR‑S7 and are not in this half.
- **The look share code goes version 2 — the look travels with its graphs** (roadmap §14.9): COPY CODE now issues a `KEYFLOW-LOOK-2:…` marker, telling the payload carries with the look every AR graph — the note width-by-velocity graph, the note fade-by-velocity graph and the live bar's lifetime fade — so the receiving machine redraws those curves too, not only the palette. APPLY CODE **still reads every version-1 code** (`KEYFLOW-LOOK-1:…`) ever shared, bit for bit, since the JSON envelope was complete from day one; a code from any other version is refused with the matching "made by a different version of Keyflow." reason. The sender's background picture and camera stay home as before. The code transport splits out into `Stage/ShareCodeEnvelope.cs` (pure, settings-free) so xUnit checks it on any machine: the `ShareCodeEnvelopeTests` six facts (version-2 URL-safe issue, version-1 codes opening bit for bit, a reason for every damaged code, chat-wrapped codes still opening, a zip bomb halting at the payload limit). The look/graph level is locked by the extended `VerifyPresetShareCodes` block in `--verify`.
- **AR performance budget: the overlay drops effect families in a fixed order when the GPU is over budget** (roadmap §14.8): `Camera/ArQualityBudget.cs` plugs into the existing adaptive-quality controller that already watches GPU frame time. As the tier worsens (Full → BloomReduced → BloomMinimal → AmbientReduced → AmbientMinimal), the frame loses exactly one more family down the roadmap's sentence — particles → pulses → note lights → light bars → note style (3D notes flatten to the plain path; the notes themselves stay) → impact glow — and restores them as pressure passes, with zero settings touched. Keys, notes and keypresses are outside the table, so no over-budget frame can ever lose the overlay itself. Locked by the `ArQualityBudgetTests` xUnit suite, the `VerifyArQualityBudget` block and an over-budget WARP rehearsal in `VerifyArNoteProjection` (deep cut: beams/pulses/impact/particles/note lights all gone, key tint and the damper strip intact; back to Full and the lights return untouched).
- **The AR look now retunes itself per camera view** (roadmap §14.7, Thêm 5): `Camera/ArViewPresets.cs` — a pure table keyed by the view id the machine already knows, not by any new setting. On the leaned cameras (45°, free mark, legacy) the paper-thin heights stay the catalogue's (impact flare 0.2 cm over the key, pulses at least 0.3 cm over the plane) and the stand-up effect quads — hold arcs and light beams — keep drawing; on the **90° overhead** view the lifts shrink to 0.05 cm / 0.1 cm (nothing stands in front of anything on a straight-down camera) and the stand-up quads stay home, because they squash to a noise-thin hairline from directly above. Unknown or empty view ids keep the leaned row, matching how the calibration store normalizes the same ids, and presets/share codes still carry nothing — the view id is machine-local. Locked by the `ArViewPresetTests` xUnit suite and the extended `VerifyArNoteProjection` block in `--verify` (overhead: beams/arcs hidden, impact lights/pulses/particles keep projecting whole on WARP).
- **Camera tips inside Calibrate, and a live lookout for hands in the picture** (roadmap §14.7, Thêm 4): the calibration window now says the two set-up things nobody remembers first time — *hang the camera over your shoulder, looking down the keys* (a square-overhead view reads every key; a camera across the keys hides the far ones) and *avoid a very wide lens* (ultra-wide bends the key rows toward the edges, and corners marked on bent rows never stay on the real keys) — plus the reminder to mark with hands out of the picture. And the window watches: a classic Y′CbCr skin-lobe lookout samples the still it shows, and when a skin-coloured share fires — anywhere in the picture before the corners are down, only inside the marked keys after — the note line quietly asks for the hands to be moved; it is advice, not a gate, and sunlit beech can still borrow the colour of a hand without harm. Locked by the `CalibrationAdviceTests` xUnit suite and the `VerifyCalibrationAdvice` block in `--verify`.
- **Calibration backup — export and import the saved alignments of the camera in use** (roadmap §14.7, Thêm 3): two new buttons in the KEYBOARD ALIGNMENT card. **EXPORT CALIBRATION** gathers every saved calibration of the camera (every view, picture size and keyboard size) into one portable `keyflow-ar-calibration` JSON and drops a snapshot PNG of the camera's current frame beside the file, so an old backup tells its camera at a glance (restoring never reads the picture). **IMPORT CALIBRATION** reads such a file, restores every row under the backup's own camera link (or the camera in use when the file names none) and pushes each row through the exact checks a fresh calibration passes — rows that fail are counted and skipped, never drawn, while the wrong kind of file or one written by a newer Keyflow is refused with a plain reason. No new setting: presets and share codes stay free of calibrations, as before. Locked by the `ArCalibrationArchiveTests` xUnit suite and the `VerifyArCalibrationArchive` block in `--verify`.
- **Automatic drift watch — the overlay notices a moved camera and asks for a re-mark** (roadmap §14.7, Thêm 2): a **Watch for camera drift** toggle in the KEYBOARD ALIGNMENT card (on by default, machine-local). While the overlay draws with a calibration, every couple of seconds the app grids the key area's picture and compares it against the reference sampled when the watch began: the best shift is found on 3-pixel cells with exposure levelling and outlier trimming (playing hands and shadows do not count, and an ±18 % brightness swing never misreports). Once the keys sit past the threshold (~6 px on a 1280 px frame) for three readings in a row, the AR card's status line says *"The keys have drifted about N px from the marked frame — open Calibrate and mark the corners again."* — deliberately a nudge, never a self-edit of the calibration; the watch also parks itself while the camera delay is being measured, since both sides ride the same camera-frame tap. Locked by the `ArDriftMeterTests` xUnit suite (synthetic frames: a 9 px move measures within ±3 px; stills, hands and lighting stay quiet) and by the `VerifyArDriftWatch` block in `--verify`.
- **One press, one light — and the light itself came alive**: on the downward note direction, a key press used to flash at the fingertip and *then flash again* when the falling head of its live bar reached the key line — a second light with no second press. The effect now fires exactly once, in both note directions, with the finger as its only source (a note you miss still glints softly at touchdown as before, and Listen mode keeps its beat). The lighting itself was polished in the same pass: the impact flash blooms in smoothly (~55 ms) and settles gently instead of popping and clipping; brightness and size follow velocity much more expressively — a soft touch is an ember, a firm hit blooms wide (a ~3× wider dynamic band); and a long-hold key breathes — its tint sways a slow few percent after about a second, while a freshly struck key stays a steady glow. There are no new settings: the new look lands on the Live Stage and the AR overlay alike because both read one shared lighting pipeline; the change is covered by the `GpuStageSimulationLightCurvesTests` xUnit suite and three new assert blocks in `--verify` (one press = one impact family, the velocity dynamic band + flash envelope, and the held-key breath).
- **Damper strip: an on/off toggle of its own and a custom colour; the alignment frame gets a colour pick**: the **AR PLAYBACK** card gains a **Damper strip** toggle (switch the glowing back-edge line off entirely for a calmer picture) and a **Damper strip colour** choice — *Follow look (halo)* keeps the strip in the look's halo family as before, *Custom* tints it with a colour of its own (useful when a preset's halo washes out over real keys); the toggle and colour belong to the look, so presets and share codes carry them. The **Alignment check** frame (KEYBOARD ALIGNMENT card) separately picks one of five high-contrast colours — White (default), Cyan, Yellow, Magenta, Lime — so the frame stays readable over bright or dim camera pictures; the guide colour is machine-local like the check itself. All three switches apply to **every AR angle** (45°/90°/free), since all views share one drawing path through the calibration.
- **Alignment check — spot a drifted camera at a glance**: a new toggle in the **KEYBOARD ALIGNMENT** card redraws the saved key area (its four borders plus a faint line at every octave) right over the live camera picture, on every quality tier, packed after everything else so it reads over notes and effects alike. A bumped tripod or a shifted keyboard shows up the moment the lines stop sitting on the real keys — calibrate again and they snap back. The mode belongs to the machine, together with the view profile and its calibration, so presets and share codes never switch it on elsewhere, and it draws only while a calibration exists. In the same pass the catalogue's **height table** is locked in `ArElementHeights`: the impact flare's centre floats exactly 0.2 cm above the struck key and a pulse never comes closer than 0.3 cm to the key plane — enough to keep glints from hiding behind the key fronts on a leaned camera, yet thin enough that the 90° overhead view stays pixel-identical (xUnit `ArAlignmentGuidesTests`/`ArElementHeightsTests` pin the geometry; WARP gets its own alignment block).
- **A glowing damper strip hugs the back edge of the keys** (the catalogue's `Dampers` element, drawn from the AR Standard tier up): a soft bright line glued to the keys' back edge — the luminous "strike line" of top-down piano videos. The line always skims the edge; pressing the sustain pedal (level 0..1, not a boolean) reaches it out over the fallboard and brightens it. Its colour is the look's **halo colour** (the same family as the stage's Pedal Glow), so it blends with every preset, and bloom builds a halo around its hot core; the Lite tier never draws it, as the catalogue demands. At the same time the **90° overhead** view profile — the most common angle for this filming style — now has `VerifyArNoteProjection` projecting the whole pipeline on WARP once more through a fronto-parallel calibration — live bars, velocity tint, the damper strip — so CI locks that angle down on every build; `ArDampersTests` runs on any machine.

## 2.0.1 — 2026-10-12

### AR overlay

- **Live press bars can now run onto the keys** (AR‑6.2, coverage): under Settings → Live camera a **MORE · LIVE BAR COVERAGE** chevron (shown while bars are on) opens the **Bar coverage on the keys** slider (25–300 % of the keys' depth): 100 keeps the familiar look — the bar stops at the keys; 200 reaches a white key's front edge — Keysight's **full coverage**; beyond spills past the keys, below 100 holds the bar free of them. The flat bars and the 3D route (Standard/Ultra) honor it alike, and whatever part of the bar is not on the keys still follows the selected note direction; the default is 100, so nobody loses the look they already have. Covered by new facts in `ArNotesTests`/`Ar3DNoteProjectionTests` and by `--verify` (extended `VerifyArGraphVariable`); the slider belongs to the machine, so a look does not carry it.
- **Live press bars can fade over their lifetime** (AR‑6.2): under Settings → Live camera → "Live press bars", a new toggle **Bar fade follows lifetime** with two sliders — **Fade at birth** and **Fade at full length** — makes a live bar change how solid it is by how far it has stretched, instead of only dimming with length as before. While a key is held the fade walks from the first value toward the second (60 % → 100 % by default, so bars brighten as they grow); when the key is released the fade walks backwards across the release, and the familiar release dim still takes the bar away at its end. The effect applies to the flat bars and to the bars flying free in 3D (Standard/Ultra) alike — and it is **off by default**, so nothing about the old look changes. Covered by `ArLiveFadeTests` and new facts in `ArNotesTests`/`Ar3DNoteProjectionTests`, and by `--verify` (extended `VerifyArGraphVariable`).
- **The LIVE KEY RESPONSE card now says what it actually does, and shows the right rows**: the card's header text (Settings → Live camera) used to bundle key tint, falling notes and press bars into one blanket sentence, then mislabeled the velocity knobs as \"minding the falling notes\"; it now spells out the three independent channels — key tint, falling notes, press bars — and notes that bars follow the look's velocity tint by strike strength. The **Bar fade follows lifetime** row and its two sliders appear only while **Live press bars** is on, and the sliders only while the fade is on too, so the dock has no dead controls left.
- **The camera delay is now remembered per camera** (AR‑3.5): `Camera delay` used to be one number for the whole machine, while every camera is late by its own amount (a USB webcam ≈ 60–150 ms; a virtual camera or a phone-as-webcam is very different). The delay is now stored under the camera's identity — the same key the calibration file uses — in `ar-delay.json` beside the settings folder: leaving a camera files the slider's value under it, opening one again brings its own delay back to the slider and the engine alike, and dragging the slider or accepting a **MEASURE DELAY** result writes to the camera that is open. The new value is covered by `ArDelayStoreTests` and `ArDelaySessionTests` in the xUnit project that runs on Linux and Windows alike.

## 2.0.0 — 2026-10-10

### Interface & graphics

- **AR overlay (in development, not yet tried on a real webcam)**: use the camera as a full-screen background, mark the four key-strip corners and choose the matching camera view. Calibration saves FOV and a full 3D keyboard pose; Calibrate now has a Unity-style Move/Rotate/Scale gizmo plus numeric translation and uniform-scale sliders. Version 5 stores the complete transform, FOV and baseline; versions 1–4 still load with zero translation and unit scale (v3/v4 keep saved FOV, v1/v2 use 70°; v4 keeps rotation, v1–v3 use 0°). Enable **Notes in perspective** to render camera-facing note quads from that pose; note travel has its own preview arrow, Screen-up/key-depth aiming buttons and independent **Note direction tilt / azimuth** controls, without changing the keyboard transform. The settings controls stay directly after the toggle and before **Minimum length, share of width**. **Live press bars** are an independent, off-by-default toggle and do not disable key tint; flat stage-space falling trails are suppressed over the camera even before calibration. Ambient layers such as petals and shooting stars remain flat screen-space and are not perspective-projected. Learn / Perform, look-ahead, opacity and key tint remain available; Quality selects Lite / Standard / Ultra. AR camera-facing quads route known styles through the shared `ShadeNoteMaterial`; new WARP checks sample all six styles on calibrated quads and Gem/Crystal on the flat AR path, pending Windows CI. Full setting/value parity is not claimed, and real-camera validation remains **Unverified** until a webcam trial. Camera delay (0–300 ms) compensates for capture latency; `--bench-ar` measures the app's own share of the work. The sandbox has no .NET SDK for local build/tests; ambient stays flat on the camera, and the ambient choice should be recorded in the PR description for hardware testing.
- **Cameras on this machine**: one line per camera by its real name (never "Camera 1"), kind, port and VID/PID; Automatic prefers colour cameras, ranks virtual ones after them and infrared (Windows Hello) cameras last. Plugging a camera in or out, or starting/stopping OBS Virtual Camera, updates the list and reopens the camera when it has to; this stays still while a take is recorded, the delay is measured or the keys are marked. The camera is the machine's: applying a preset or pasting a share code does not change the camera you picked.
- Fixed two faults reading a real camera: a wrong Media Foundation key made every YUY2/MJPG camera report `0xC00D5212`, and a reader created on one thread and read on another gave `E_NOINTERFACE`; each reading of the camera list no longer leaks a COM reference.
- Live camera: the app picks the mode nearest 720p at the highest rate the camera has (up to 60 fps), reads at the camera's own pace instead of a fixed 30 fps cap, and shows the real mode (`720p · 60 fps · MJPG`) in the status line. Frames travel through a three-buffer ring, so a slow draw can no longer tear a picture.
- **88/76/61/49-key keyboards**: pick the size in Settings → Keyboard. The stage, falling notes, lit keys and the GPU engine all draw the chosen size, and song notes outside its range are not drawn. Presets leave the keyboard size alone. The page also suggests the size that fits the notes played on the real instrument.
- Restored the concert-grand home screen, its three built-in themes, localized labels and icon-button UI from `arena/01a107ec-pianopath` when resolving the merge with `main`.
- Kept the supplied **Keyflow Visualizer Piano** artwork and the Windows executable/installer icons from `main`. The Cosmic Nocturne shader source and embedded resource are retained but are not enabled on the restored home screen; the in-app layout and branding follow the concert-grand version.

### Audio & setup experience

- Replaced the bundled SF2 piano and SF2 engine with **Salamander Grand Piano V3 as the sole instrument**, rendered through SFZ; removed SF2 import and external instrument selection.
- Salamander is no longer stored in Git or bundled in ZIPs/installers. Users explicitly download the one-time library (about **748 MB**) from Audio settings. The downloader pins a commit, verifies every FLAC against its Git blob SHA-1, supports cancel/resume, and writes the SFZ mapping and attribution only after all samples verify.
- Split the startup menu into **Piano & Audio Setup** and **MIDI Devices** shortcuts; Audio settings now show clearer status, download progress, piano load/repair actions and output selection.
- Added a **Piano sound** switch to Quick Adjust and Audio settings. Muting stops only this app's audio output; visuals, outgoing MIDI and audio recorded into a take remain unchanged. The preference is saved locally and presets do not override it.
- Updated the READMEs, release instructions, source checks and generated public release page for the separate piano download; removed obsolete Git LFS, `Assets/` and old SF2-size guidance.

### Interface & dialogs

- Icon buttons replace the wall of all-caps labels in the settings dock: the preset toolbar (save as, delete, import, export, reset), the share-code row (copy), the History page (export HTML, clear), the Background/Camera/Profile file actions and theme create/edit/delete are now glyph tiles with localized tooltips. Primary actions keep their labels (“Apply preset”, “Apply code”, “Save”).
- The header’s **SETTINGS** label becomes a gear icon button and the LOOK chip’s “Change” label becomes a compact icon; the startup menu’s five links (Piano & Audio, MIDI, shortcuts, about, exit) gain matching icons.
- Unicode glyphs (⬇, ⟳, ●, ↺, ×) are gone from button labels — they never followed the theme and rendered inconsistently; the piano download, device refresh, recording, loop clear and per-slider resets now use the app’s own icon set.
- Secondary action labels are unified to sentence case; every icon-only button keeps its tooltip, which is also the accessible name screen readers announce.
- Quick Adjust stage switches now size to their captions instead of reserving 150-pixel items, keeping each switch close to its label.

### Packaging & release

- **Packages are not code-signed yet**: SignPath Foundation declined the open-source application, so 2.0.0 ships the three `PianoPath.exe` builds and the `Keyflow-Setup-*.exe` installer **unsigned**. Windows SmartScreen may show *"Windows protected your PC"* on install or first run: choose *More info → Run anyway*, then check the files against `SHA256SUMS.txt`. `release.yml` has no signing step any more; the plan for signing later is in `docs/CODE-SIGNING.md`.
- The three ZIP packages are now packed **after** signing (they used to be packed right after publish); file names and `SHA256SUMS.txt` keep their old shape.

## 1.0.0 — 2026-10-03

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
- **No more freezes and no more broken files when recording** — three fixes after the first cut: every frame
  and audio block releases its Media Foundation COM buffer in a `finally` (they used to wait for the GC, so
  memory ballooned and the window froze when they were all freed at once); frames and audio go through a
  bounded queue to a worker thread that does the NV12 conversion and the encoder write, and frames the
  encoder falls behind on are folded into the newest one as repeats so the file keeps real time; a whole MP4
  take lives on **one thread** (Media Foundation refuses a sink writer created on one thread and called from
  another, which is why earlier takes came out empty or without an index) and a failed close is reported
  instead of ignored; AVI is written off the UI thread too, repeats are written as zero-byte null frames
  rather than re-compressed duplicates, and a take is capped at **1080p** in both directions.
- **MP4 is now the default format**, and the file it writes opens in every player: `MF_MT_FRAME_SIZE` packs
  the width into the high word, but the recorder passed the height first, so the encoder read every frame
  with the wrong stride and a 1920×1080 take was declared 1080×1920 — horizontal static on screen. The size
  is now declared as width × height, `--soak-record` compares the size the file declares with the size that
  was asked for, and new profiles default to MP4 instead of AVI (an AVI without an MJPEG codec is raw
  frames, which fills the 2 GB limit in seconds at 1080p).

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
