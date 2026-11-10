# Product presentation · Keyflow v1.0.0 (archive)

> **Bản tiếng Việt: [README.vi.md](README.vi.md)** · This English edition is the primary document; the
> Vietnamese edition follows it. The slides themselves exist in Vietnamese only.

The archived slide deck of the **Keyflow 1.0.0** release, dated 2 October 2026; it shows the product
exactly as it was on that day. Version 1.0.0 still describes a piano bundled as an SF2 file; **this is not
current product documentation**. The sound model today is Salamander Grand Piano V3, downloaded
separately and SFZ only; see the [README](../../README.md) and the [CHANGELOG](../../CHANGELOG.md).
Every slide is one page, 16:9.

| File | Format | Contents |
| --- | --- | --- |
| `Keyflow-Presentation-v1.0.0.pdf` | PDF, 20 pages, 16:9 | The copy for projection — a dark, stage-like background, ready to show |
| `Keyflow-Presentation-v1.0.0.docx` | Word (landscape, 13.33″ × 7.5″) | The same 20 slides, for editing and reworking |

## List of slides

The titles below are English renderings of the Vietnamese slide titles.

1. Cover — Keyflow · Piano Performance & Concert VFX Studio
2. Overview — what is Keyflow?
3. Highlights — the numbers that speak for themselves (240 FPS, 16-bit HDR, 24,000 particles, 88 keys…)
4. The concert stage
5. The Direct3D 11 GPU engine
6. The library of performance effects
7. The ray-traced keyboard & the stage background
8. Sound — a hand-written SoundFont 2 synthesizer
9. Importing a song — MIDI & MusicXML, the staff on the stage
10. Practice with scoring
11. Recording — from the stage to video (AVI / PNG with alpha / MP4)
12. Interface, presets & themes
13. Camera overlay & hand tracking
14. Two languages and accessibility
15. Essential shortcuts
16. Distribution & installation
17. System requirements
18. Testing & verification (`--verify`)
19. Current limitations
20. Licence, author & download

## Notes

- The screenshots in the slides were taken from the Vietnamese picture set,
  [`docs/previews/vi/`](../previews/vi), as it stood at 1.0.0 — pictures the application itself
  renders in CI (`--snapshot`), not staged ones. That folder now shows the current build, so its
  pictures are no longer the ones in the deck.
- Every figure and statement in the slides follows the v1.0.0 release documentation; when there is a
  new release, update the content and then regenerate both files, so that the two formats do not
  drift apart.
