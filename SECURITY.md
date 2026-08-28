# Security

Swar installs a system-wide keyboard hook and types into other applications. That is a
sensitive thing to run, and reports about it are welcome.

## Reporting a vulnerability

**Do not open a public issue.**

Use GitHub's private vulnerability reporting on this repository:
[**Report a vulnerability →**](https://github.com/vishaldesai/Swar-install/security/advisories/new)

That opens a private advisory visible only to you and the maintainer. Include the Swar
version, the Windows build, what you did, and what happened. A proof of concept helps;
a working exploit is not required.

This is a personal project maintained by one person, so calibrate expectations: an
acknowledgement within a week, and an honest answer about timelines rather than an
optimistic one. If a fix takes a while, the advisory stays private until it ships.

Supported: the latest release only. There is no updater, so a fix means a new release
that you install yourself.

## What is in scope

Anything that breaks one of the properties the application claims:

- **The hook reads one key.** It inspects whether the configured push-to-talk key is
  down and passes every other event through untouched. Anything that makes it observe,
  swallow, or record other keystrokes is in scope.
- **Nothing leaves the machine.** There is no backend, no telemetry, and no HTTP client
  outside the one that fetches speech models. Any outbound request beyond the model
  download is in scope, as is anything that defeats the WebView content-security policy
  that enforces this.
- **The model download is verified.** Models are fetched over HTTPS, checked against a
  pinned SHA-256, and installed fail-closed. Anything that gets an unverified file
  installed as a model is in scope.
- **Transcripts stay local.** They live in one SQLite file under `%APPDATA%\Swar\`.
  Anything that exposes them beyond the local user account is in scope. So is any log
  line containing transcript text, dictionary entries, or audio — the logger is supposed
  to record paths, counts and durations only.
- **Injection goes where you are looking.** Text is typed into the window that had
  focus. Anything that redirects it elsewhere is in scope.
- **The installers.** Anything in the install or uninstall path that escalates
  privilege, writes outside its install directory, or leaves data behind that the
  uninstaller claims to remove.

## Known, and not a finding

- **The builds are unsigned.** SmartScreen warns and Windows Installer shows an unknown
  publisher. This is tracked, and a certificate is the next milestone. Verify the
  SHA-256 published in [CHANGELOG.md](CHANGELOG.md) before you run either installer.
- **There is no auto-update**, so there is no update channel to attack — and no
  mechanism to push you a fix either.
- **A test-harness bypass exists.** The hook ignores synthetic input by default. It
  accepts it only when an environment variable is set in Swar's own process environment
  *and* the event carries a specific tag, which exists so an automated test can drive
  the app. Setting a variable inside another process's environment already requires
  control of that session, so this grants nothing new — but it is documented here rather
  than left for you to find.
- **The speech model is third-party.** It comes from the k2-fsa `asr-models` release
  with a pinned hash. Swar verifies the file it was told to fetch; it does not audit the
  weights.
