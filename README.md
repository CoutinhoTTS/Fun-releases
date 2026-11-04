# Fun releases

Public release artifacts for **Fun**, a macOS desktop workspace built on GPUI.
The source repository is private; this repository only holds downloadable builds
and the update manifest.

## Update feed

The app checks a static manifest, not the GitHub API, so there is no anonymous
rate limit:

```text
https://github.com/CoutinhoTTS/Fun-releases/releases/latest/download/latest.json
```

`releases/latest` ignores drafts and pre-releases. Download URLs are served from
GitHub's object storage after a redirect and need no authentication. GitHub
caches release assets at the edge: after a publish, the feed can serve the
previous manifest for a few minutes. Checks are daily, so this converges.

## Manifest

Each stable release publishes `latest.json` and its Ed25519 signature next to
the DMG:

```json
{
  "version": "0.1.3",
  "tag": "v0.1.3",
  "channel": "stable",
  "published_at": "2026-11-04T03:09:24Z",
  "notes_url": "https://github.com/CoutinhoTTS/Fun-releases/releases/tag/v0.1.3",
  "notes": "## What's new\n\n…",
  "assets": {
    "macos-arm64": {
      "url": "https://github.com/CoutinhoTTS/Fun-releases/releases/download/v0.1.3/Fun-0.1.3-macos-arm64.dmg",
      "sha256": "…",
      "size": 27426093
    }
  }
}
```

Rules:

- `version` is the semantic version without a leading `v`; compare it with the
  bundle's `CFBundleShortVersionString`.
- `tag` is the exact release tag the asset URLs point at. Pin downloads to it.
- `channel` is `stable` or `prerelease`.
- `notes` is Markdown shown in the app's update window.
- `assets` is keyed by `<os>-<arch>`; only publish keys whose artifact exists.
- `sha256` is lowercase hex and is verified before installation.
- `latest.json.sig` is the base64 Ed25519 signature over the exact bytes of
  `latest.json`. From 0.1.3 the app refuses a manifest without a valid
  signature, using the public key compiled into it.
- `notes_url` must be publicly reachable (this repository, not the private source).

## Release layout

```text
v0.1.3
├── Fun-0.1.3-macos-arm64.dmg
├── SHA256SUMS
├── latest.json
└── latest.json.sig
```

## Versioning

A published tag is immutable. To ship a change, bump the version and publish a
new tag: the app compares versions from the manifest, and replacing an existing
asset can briefly leave the manifest and the cached DMG out of sync. The
publishing script refuses to overwrite a published release unless `--force` is
explicitly passed.

## Signing

Builds are ad-hoc signed and not notarized by Apple. On first launch, right-click
**Open**, or allow the app under System Settings → Privacy & Security. Every
artifact is published with its SHA-256 so downloads can be checked independently.

## Publishing

Releases are published from the private source repository with
`scripts/publish-release.sh`, which packages the app, writes and signs
`latest.json`, and uploads the files here with `gh release`. The signing key
comes from `scripts/update-keygen.sh`; the private key never enters either
repository.
