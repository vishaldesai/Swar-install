# Cutting a release

For the maintainer. The build happens in the [Swar](https://github.com/vishaldesai/Swar)
repository on a Windows machine; this repository only carries the result.

There is no CI. A local build plus `gh release create` is the honest version of this
today — automating it would mean putting a Windows runner and the toolchain behind a
workflow, and that is worth doing when releases stop being rare.

## 1. Build

In the Swar repository. The version lives in three files and all three must agree:

| File | Field |
|---|---|
| `Cargo.toml` | `[workspace.package] version` |
| `apps/desktop/package.json` | `version` |
| `apps/desktop/src-tauri/tauri.conf.json` | `version` |

Then:

```powershell
just check      # fmt, clippy, tests, the Win32 boundary gate, contrast, typecheck
just build      # both bundles
```

Artifacts land in `target/release/bundle/nsis/` and `target/release/bundle/msi/`.

Install the built artifact and launch it before going further. `just build` has shipped
an installer that produced a binary which died on launch with `0xC0000135` because
`bundle.resources` did not name the three native DLLs. Smoke-test the installed copy,
never the loose `.exe` in `target/release/`.

## 2. Hash

```powershell
$v = "0.1.0"
cd <Swar>/target/release/bundle
Get-FileHash -Algorithm SHA256 nsis/Swar_${v}_x64-setup.exe, msi/Swar_${v}_x64_en-US.msi |
  ForEach-Object { "{0}  {1}" -f $_.Hash.ToLower(), (Split-Path $_.Path -Leaf) } |
  Set-Content SHA256SUMS.txt -Encoding utf8NoBOM
Get-Content SHA256SUMS.txt
```

The encoding matters. A BOM or CRLF line endings make this file unreadable to `sha256sum -c`,
which is what the release-pointers workflow uses to re-check the build in step 5. That
workflow normalises a copy before checking, so a slip here will not fail the release — but
the file a stranger downloads should be clean regardless.

## 3. Write the notes

In this repository, add a section to the top of [CHANGELOG.md](CHANGELOG.md) following
the shape of the one below it: what changed, an assets table carrying the byte counts and
the hashes from step 2, and an explicit **Known limitations** block. The limitations block
is not boilerplate — reread it every release and delete the lines that stopped being true.

[README.md](README.md) carries no version number and needs no edit — its download links
resolve through the release-pointers workflow, described under [Stable URLs](#stable-urls)
below. If any dependency changed, regenerate the crate list in
[THIRD-PARTY-NOTICES.md](THIRD-PARTY-NOTICES.md) — the config and template are in
[`tooling/`](tooling/) and that file says how. Commit.

## 4. Tag and publish

```powershell
git tag v0.1.0
git push origin main --tags

gh release create v0.1.0 `
  --repo vishaldesai/Swar-install `
  --title "Swar 0.1.0" `
  --notes-file <the new CHANGELOG section, saved to a file> `
  "<Swar>/target/release/bundle/nsis/Swar_0.1.0_x64-setup.exe" `
  "<Swar>/target/release/bundle/msi/Swar_0.1.0_x64_en-US.msi" `
  "<Swar>/target/release/bundle/SHA256SUMS.txt"
```

Add `--prerelease` for a build you do not want surfaced as *Latest*.

## 5. Check it from outside

Download both assets from the public release page — not from your build directory — and
verify the hashes against the CHANGELOG entry. That is the only step that proves the file
a stranger gets is the file you hashed.

Publishing also starts the [Release pointers](.github/workflows/release-pointers.yml)
workflow. It re-verifies the assets against `SHA256SUMS.txt` and, only if they match,
attaches `Swar-x64-setup.exe`, `Swar-x64.msi` and `latest.json`. Wait for it, then check the
fixed-name URLs that the README and the website depend on:

```powershell
gh run watch
curl -sIL -o NUL -w "%{http_code}`n" https://github.com/vishaldesai/Swar-install/releases/latest/download/Swar-x64-setup.exe
curl -sIL -o NUL -w "%{http_code}`n" https://github.com/vishaldesai/Swar-install/releases/latest/download/Swar-x64.msi
curl -sL https://github.com/vishaldesai/Swar-install/releases/latest/download/latest.json
```

If that workflow fails, the release is published but every fixed-name link still serves the
previous version. That is the safe direction to fail in, and it is not a finished release —
fix the mismatch and re-run with `gh workflow run release-pointers.yml -f tag=vX.Y.Z`.

## Stable URLs

These are a published API. The website links to them directly and is not redeployed when a
release ships, so **renaming one silently breaks every download button on the site.**

| URL under `https://github.com/vishaldesai/Swar-install` | What it serves |
|---|---|
| `/releases/latest/download/Swar-x64-setup.exe` | The NSIS installer from the newest non-prerelease release |
| `/releases/latest/download/Swar-x64.msi` | The MSI from that same release |
| `/releases/latest/download/latest.json` | Version, publish date, and the byte count and SHA-256 of each asset |
| `/releases` | The archive. Every past version stays at its own versioned URL forever |

`latest.json` carries a `schema` field. Bump it if the shape changes.

A browser cannot read `latest.json`. GitHub serves release assets with no
`Access-Control-Allow-Origin` header, so a cross-origin `fetch` of it is blocked — it is
meant for build-time site generation, scripts, and anything running outside a browser. A
page that wants to show the current version client-side should call
`https://api.github.com/repos/vishaldesai/Swar-install/releases/latest`, which does send
`Access-Control-Allow-Origin: *` and returns `tag_name`, `published_at`, and a `size` and
`digest` for each asset. It is limited to 60 requests an hour per visitor IP, so treat it as
decoration over links that already work without it.

Prereleases are skipped. `/releases/latest/` ignores them and so does the workflow.
