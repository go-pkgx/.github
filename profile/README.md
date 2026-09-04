<p align="center">
  <img src="https://go-pkgx.github.io/img/logo.svg" alt="go-pkgx" width="96" height="96">
</p>

<h1 align="center">go-pkgx</h1>

<p align="center">
  <em>The pkgx world in pure Go — run and install packages on <code>FROM scratch</code>,
  with one static binary each, a shared backend, and a signed, attested package registry.</em>
</p>

<p align="center">
  <a href="https://go-pkgx.github.io/">Home</a> ·
  <a href="https://go-pkgx.github.io/docs/">Docs</a> ·
  <a href="https://github.com/go-pkgx/pkgx">pkgx</a> ·
  <a href="https://github.com/go-pkgx/pkgm">pkgm</a> ·
  <a href="https://github.com/go-pkgx/bottle">bottle</a>
</p>

---

Everything is `CGO_ENABLED=0` Go with no runtime dependencies of its own — single
static binaries that materialise a package's full dependency closure on demand and
work on a literally-empty `FROM scratch` image.

| repo | role |
| --- | --- |
| [**pkgx**](https://github.com/go-pkgx/pkgx) | **runtime** — run packages on the fly (`pkgx node@22`), bring several into an environment (`pkgx +git +bash -- …`), and name and load environments with `pkge`, a module system that needs no Lmod |
| [**pkgm**](https://github.com/go-pkgx/pkgm) | **installer** — drop-in reference-pkgm CLI (`install`/`uninstall`/`shim`/`list`/…), plus a `run` for scratch images |
| [**bottle**](https://github.com/go-pkgx/bottle) | **shared backend** — the pkgx bottle protocol: resolution, download, soname-exact `FROM scratch` closure completion, loader-aware exec |
| [**mirror**](https://github.com/go-pkgx/mirror) | **mirror** — sync a local mirror of pkgx bottles; serve it and point tools at it with `PKGX_DIST` |
| [**bk**](https://github.com/go-pkgx/bk) | **build tool** — a pure-Go brewkit: builds a pantry recipe and packages a bottle, with the build *target* a first-class value, so Windows PE bottles cross-build from a linux host |
| [**packages**](https://github.com/go-pkgx/packages) | **registry** — a pure-Go factory that builds pantry recipes with `bk` and publishes **signed, attested** bottles (SBOM + SLSA provenance + signature) to `ghcr.io/go-pkgx/packages` |

## Install

Install `pkgm` (the pure-Go installer) in one line — it grabs the static binary
for your os/arch from the named release and verifies it against `SHA256SUMS`:

```sh
# Linux / macOS
curl -fsSL https://go-pkgx.github.io/install.sh | sh -s -- pkgm v0.1.2
```

```powershell
# Windows (PowerShell)
$env:PKGM_VERSION='v0.1.2'; irm https://go-pkgx.github.io/install.ps1 | iex
```

The version is named on purpose: this line copied today and the same line
copied in six months install the same bytes. `sh -s -- pkgm latest` (or
`PKGM_VERSION=latest`) asks for the newest instead, and `sh -s -- pkgx v0.1.3`
/ `sh -s -- mirror v0.1.3` install the other two tools.

Go users can `go install github.com/go-pkgx/pkgm@latest`. Then `pkgm install
lz4.org` verifies each bottle against the signed registry by default.

Both tools import `go-pkgx/bottle`, so there is one source of truth and no
duplication. `net/http` with an embedded CA bundle replaces curl+openssl;
`compress/gzip` + a pure-Go xz decoder replace info-zip+xz. The `packages` factory
signs every bottle against a pinned key, and `PKGX_VERIFY=1` refuses anything that
does not verify. Everything is BSD-3-Clause, and CI does not stop at cross-compiling:
the suites **run** under `qemu-user` on arm64, riscv64, ppc64le, s390x and loong64 —
s390x because it is big-endian and nothing else is, and this code reads formats it
did not write.
