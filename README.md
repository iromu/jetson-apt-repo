# Jetson Nano custom apt repository

A small, **GPG-signed** static apt repository of pre-built `arm64` (aarch64)
Debian packages for the Jetson Nano (Ubuntu 18.04 / Bionic, L4T). Served
statically from GitHub Pages — no server, no PPA, no build farm.

## Packages

| Package    | Version   | What it is |
|------------|-----------|------------|
| `nodejs24` | 24.20.0-1 | Node.js 24 runtime (from-source aarch64 build) + npm, npx, corepack and native-addon headers. Installed under `/usr`. |
| `llama-cuda` | 5092    | llama.cpp built for the Jetson Nano's sm_53 GPU with CUDA 10.2. |

> `nodejs24` is named `nodejs24` (not `nodejs`) on purpose, so it does **not**
> clash with Ubuntu 18.04's own `nodejs` (v10) package.

## Prerequisites

All packages are `arm64` (aarch64) — they will not install on x86.

| Package      | Runtime prerequisites |
|--------------|-----------------------|
| `nodejs24`   | glibc ≥ 2.17 (`libc6`), `libstdc++6` |
| `llama-cuda` | `libc6`, `libstdc++6`, `libgcc1`, **plus** the CUDA 10.2 runtime (`libcudart.so.10.2`) and the Tegra GPU driver (`libcuda.so.1`) shipped with the board's L4T image |

> The CUDA runtime and driver are provided by the Jetson's system image, not by
> the package, so they are not in `llama-cuda`'s apt `Depends`.

To consume the repo with apt you also need `curl` (or `wget`) and a normal `apt`.

> Verified on a Jetson Nano (Ubuntu 18.04 / L4T R32.7.6, CUDA 10.2): aarch64,
> glibc 2.27 — every prerequisite above is satisfied and both installed binaries
> (`/usr/bin/node`, `/usr/bin/llama-cli`) resolve all of their libraries.

## Install

The base URL is `https://iromu.github.io/jetson-apt-repo/` (served by GitHub
Pages from the `master` branch).

```bash
# 1) Add the signing public key to a dedicated keyring
sudo install -d -m 0755 /etc/apt/keyrings
sudo curl -fsSL https://iromu.github.io/jetson-apt-repo/jetson.gpg \
     -o /etc/apt/keyrings/jetson.gpg

# 2) Add the repository (flat layout, signed with the key above)
echo "deb [signed-by=/etc/apt/keyrings/jetson.gpg] https://iromu.github.io/jetson-apt-repo/ ./" \
     | sudo tee /etc/apt/sources.list.d/jetson.list

# 3) Fetch the package index (verifies the signature + checksums)
sudo apt-get update

# 4) Install what you need
sudo apt-get install nodejs24
sudo apt-get install llama-cuda
```

`apt-get update` will report `OK` for this source only if the repository
signature and file checksums validate. If it errors on the signature, the
key at step 1 does not match the one that signed the repo — re-download
`jetson.gpg`.

## Updating

When new package versions are published to the repo, just re-run:

```bash
sudo apt-get update
sudo apt-get upgrade
```

There are **no automatic security updates** — review and apply updates
deliberately.

## Uninstall

```bash
sudo apt-get remove nodejs24
sudo apt-get remove llama-cuda
# optional: remove the repo + key
sudo rm /etc/apt/sources.list.d/jetson.list /etc/apt/keyrings/jetson.gpg
sudo apt-get update
```

## Notes / caveats

- **`/usr/bin/node` conflict:** `nodejs24` installs `/usr/bin/node`. If the
  distro `nodejs` (v10) package is already installed, remove it first to avoid
  a file-ownership conflict: `sudo apt-get remove nodejs`.
- **Scope:** these are hand-built packages for the Jetson Nano (aarch64). They are not
  part of Ubuntu and receive no upstream security patching.
- **Reproducibility:** the Node build is a local from-source aarch64
  compilation; `llama-cuda` is built against the Jetson Nano's CUDA 10.2 / sm_53.

## Repository layout

```
Packages, Packages.gz     package index (generated with dpkg-scanpackages)
Release                   unsigned index metadata
InRelease                 clearsigned Release (what apt checks)
Release.gpg               detached signature of Release
jetson.gpg                binary public key (for /etc/apt/keyrings)
jetson-public.asc         armored public key (human-readable)
nodejs24_*.deb            the Node 24 package
llama-cuda_*.deb          the llama.cpp package
```
