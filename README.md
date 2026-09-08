# DriveChk releases

`biozalp/drivechk-releases` — **public**. The update feed for [DriveChk](https://drivechk.com).

This repository exists so the app can check for updates without the source being public. It holds one
file that matters:

- **`appcast.xml`** — the Sparkle feed. Every shipped version, with its EdDSA signature.

Served to the app from:

```
https://raw.githubusercontent.com/biozalp/drivechk-releases/main/appcast.xml
```

## What is not here

**The DMGs.** Each build attaches to a GitHub Release under its version tag, and the appcast points
at that URL. Committing binaries would turn a five-megabyte app into a multi-gigabyte clone, and
Sparkle never reads them from git anyway.

**Anything secret.** The Sparkle private key lives in the login Keychain on the release machine and
has no copy here or anywhere else in the tree. Only the public half ships, compiled into the app.

## How it gets updated

Nothing here is edited by hand. `scripts/release.sh` in the source repo regenerates `appcast.xml`,
commits it, and pushes — which is why this repo has to be cloned as a sibling of `src/`:

```
drivechk/
├── src/        ← the release script runs from here
└── releases/   ← and writes the appcast to here
```

The script refuses to start if this directory is not a git clone, rather than discovering it after a
full notarized build.

## Verifying a release

Every enclosure carries a `sparkle:edSignature`. The app verifies it against the public key compiled
into the binary, **and** macOS verifies the Apple code signature on the downloaded app. An update
that fails either check is discarded. `release.sh` fails hard if the generated appcast is missing a
signature, because an unsigned feed is one every installed client silently refuses.
