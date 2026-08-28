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
  Set-Content SHA256SUMS.txt
Get-Content SHA256SUMS.txt
```

## 3. Write the notes

In this repository, add a section to the top of [CHANGELOG.md](CHANGELOG.md) following
the shape of the one below it: what changed, an assets table carrying the byte counts and
the hashes from step 2, and an explicit **Known limitations** block. The limitations block
is not boilerplate — reread it every release and delete the lines that stopped being true.

Then update the two version-pinned download links in [README.md](README.md). If any
dependency changed, regenerate the crate list in
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
