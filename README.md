# Swar — download and install

**स्वर** — *voice, tone, musical note.*

Hold a key, talk, release, and cleaned-up text is typed into whatever had focus. The
speech model runs on your machine. No account, no sign-in, no telemetry, no cloud.

This repository is the download page. It carries the installers, their checksums, and
the release notes — nothing else. The source lives in
[vishaldesai/Swar](https://github.com/vishaldesai/Swar).

---

## Download

**[Latest release →](https://github.com/vishaldesai/Swar-install/releases/latest)**

Windows 11, x64. There is no macOS or Linux build yet.

| File | For | Installs to |
|---|---|---|
| [`Swar_0.1.0_x64-setup.exe`](https://github.com/vishaldesai/Swar-install/releases/download/v0.1.0/Swar_0.1.0_x64-setup.exe) | **One person. Start here.** | `%LOCALAPPDATA%\Programs\Swar`, no admin rights |
| [`Swar_0.1.0_x64_en-US.msi`](https://github.com/vishaldesai/Swar-install/releases/download/v0.1.0/Swar_0.1.0_x64_en-US.msi) | A fleet — `msiexec`, Group Policy, Intune | `Program Files`, asks for elevation |

They install the same application. They disagree about *where*, and *with whose
permission*. Take the `.exe` unless someone has told you to take the `.msi`.

## Verify what you downloaded

The installers are unsigned, so the checksum is the only thing standing between you and
a file that is not the one published here. It takes ten seconds.

```powershell
Get-FileHash -Algorithm SHA256 .\Swar_0.1.0_x64-setup.exe
```

Compare against the entry in [CHANGELOG.md](CHANGELOG.md) and the `SHA256SUMS.txt`
attached to the release. Those are two separate surfaces on purpose: the changelog is in
git history, the release asset is not, and agreeing with both is a stronger statement
than agreeing with either.

## Install

Because the build is unsigned, Windows will stop you once:

- **The `.exe`** — SmartScreen shows *"Windows protected your PC"*. Click **More info**,
  then **Run anyway**.
- **The `.msi`** — Windows Installer shows an unknown publisher on the elevation prompt.

That warning is accurate and you should not learn to ignore it in general. It means
Windows has no certificate to attribute this file to anyone. Verify the checksum first;
a signing certificate is the next milestone.

Swar starts in the system tray. The push-to-talk key is **Right Ctrl** by default, and
you can change it in Settings.

## First run downloads the speech model

Swar reaches the network exactly once, during setup, to fetch the speech model — and
never again.

The default is NVIDIA `nemotron-3.5-asr-streaming-0.6b`, **453.3 MiB**, pulled from the
[k2-fsa/sherpa-onnx `asr-models` release](https://github.com/k2-fsa/sherpa-onnx/releases/tag/asr-models),
checked against a pinned SHA-256, and installed fail-closed — if the hash does not
match, the file is discarded rather than used. Six other models are selectable in
Settings → Model.

It is not in the installer, which is why the installer is 10 MB rather than 460 MB. Once
the model is on disk, Swar works offline. Copy `%APPDATA%\Swar\models\` across from
another machine and it never reaches out at all.

## Uninstall

Uninstall from **Settings → Apps → Installed apps**, or run the uninstaller in the
install directory.

That leaves your data behind on purpose. To remove it too, delete:

```
%APPDATA%\Swar\
```

That one folder holds `config.toml`, `swar.sqlite` (your dictation history, dictionary
and snippets), `logs\`, and `models\` — so deleting it also throws away the 453 MiB
model download. Move `models\` out first if you plan to reinstall.

## What is not here yet

Stated plainly, because you are about to run this on your own machine:

- **The build is unsigned.** SmartScreen will warn. See above.
- **There is no updater.** Nothing checks for new versions and nothing updates itself.
  Watch this repository's releases, or check back.
- **Windows 11 x64 only.** macOS is planned; there is no date.
- **The speech model is not bundled.** First run needs the network. See above.

## About the keyboard hook

Swar installs a system-wide low-level keyboard hook and types into other applications.
If you are technical, you should want an explanation of that before installing it, so:

The hook inspects only whether the one configured push-to-talk key is down, and it
passes every event through untouched. It never swallows a keystroke and never records
what you type. Transcripts are stored locally in SQLite with a retention setting you
control, and a single button deletes all of them. Nothing is sent anywhere — the app
has no HTTP client outside the one that fetches speech models, and the WebView's own
content-security policy forbids outbound requests even if someone wrote one.

The code is public. Check it rather than taking this paragraph's word for it.

## Something broke

- **The installer failed, or the app will not start** —
  [open an install problem](https://github.com/vishaldesai/Swar-install/issues/new?template=install-problem.yml).
- **Swar is installed but misbehaving** —
  [open a bug report](https://github.com/vishaldesai/Swar-install/issues/new?template=bug-report.yml).
- **Questions, ideas, "does it do X"** —
  [Discussions](https://github.com/vishaldesai/Swar-install/discussions).
- **A security vulnerability** — do not open an issue. Read [SECURITY.md](SECURITY.md).

Logs are at `%APPDATA%\Swar\logs\`. They record paths, counts and durations — never
transcript text, dictionary entries or audio — so they are safe to attach to a bug
report.

## Licence

Swar is dual-licensed under [MIT](LICENSE-MIT) or [Apache-2.0](LICENSE-APACHE), at your
option. That covers this repository and the application in the release assets.

It does not cover the speech model, which you download separately at first run and which
carries its own licence — the NVIDIA Open Model License for the default. See
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) for everything redistributed inside the
installer and everything fetched beside it.
