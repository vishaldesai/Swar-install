# Release notes

Newest first. Every release records the SHA-256 of each asset here as well as in the
`SHA256SUMS.txt` attached to it, so the two can be compared against each other.

---

## 0.1.0 — 2026-08-27

The first public build. Dictation works end to end: hold **Right Ctrl**, talk, release,
and cleaned-up text is typed into whatever had focus, with live partial text on the
overlay while you are still speaking.

**In this build**

- Push-to-talk dictation with a configurable key, streaming on-device speech recognition,
  and text injection into whatever application has focus.
- Seven selectable speech models across three tiers. The default,
  `nemotron-3.5-asr-streaming-0.6b`, streams cache-aware, so text appears before you
  release the key.
- The Hub: dictation history, a personal dictionary, snippets, settings, and a
  notification centre.
- Voice Profile — the dictionary learns the words you actually use.
- Insights — words per minute, streaks, per-application usage, and a scratchpad.
- Local storage only. One SQLite file under `%APPDATA%\Swar\`, a retention setting you
  control, and one button that deletes every transcript.

**Assets**

| File | Bytes | SHA-256 |
|---|---:|---|
| `Swar_0.1.0_x64-setup.exe` | 10,606,339 | `b033a1440393a7fb69196fb287257b2a38a197cb6eaab3f0d62f3b9251d1a5b4` |
| `Swar_0.1.0_x64_en-US.msi` | 14,913,536 | `7ae113d721d811b4849d41fcc04f61a35aca8200b6a6e0d51305291dc63f4fac` |

**Known limitations**

- **Unsigned.** SmartScreen warns on the `.exe`; the `.msi` shows an unknown publisher.
  Verify the checksum above before running either. A signing certificate is next.
- **No updater.** Nothing checks for a new version. Watch this repository's releases.
- **Windows 11 x64 only.** No macOS or Linux build.
- **The speech model is not bundled.** First run downloads 453.3 MiB from the
  k2-fsa `asr-models` release, SHA-256 pinned and installed fail-closed. After that,
  Swar does not touch the network.
- **The MSI upgrade code is derived, not pinned.** A future release may not upgrade a
  0.1.0 MSI install cleanly. Uninstall before installing a newer `.msi` if it refuses.

**Not built yet.** The cleanup tier, transforms, and the meeting notetaker are designed
but unbuilt — each is blocked on whether a local language model earns its place, not on
effort.
