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
GitHub's object storage after a redirect and need no authentication.

## Manifest

Each stable release publishes `latest.json` next to the DMG:

```json
{
  "version": "0.1.2",
  "tag": "v0.1.2",
  "channel": "stable",
  "published_at": "2026-11-04T01:55:51Z",
  "notes_url": "https://github.com/CoutinhoTTS/Fun-releases/releases/tag/v0.1.2",
  "assets": {
    "macos-arm64": {
      "url": "https://github.com/CoutinhoTTS/Fun-releases/releases/download/v0.1.2/Fun-0.1.2-macos-arm64.dmg",
      "sha256": "396f179e4ba41ee94092fe966d8f12d38393ac24787666ed08ace1cea8ac713b",
      "size": 27426093
    }
  }
}
```

Rules:

- `version` is the semantic version without a leading `v`; compare it with the
  bundle's `CFBundleShortVersionString`.
- `assets` is keyed by `<os>-<arch>`; only publish keys whose artifact exists.
- `sha256` is lowercase hex and is verified before installation.
- `notes_url` must be publicly reachable (this repository, not the private source).

## Release layout

```text
v0.1.2
├── Fun-0.1.2-macos-arm64.dmg
└── latest.json
```

## Signing

Builds are ad-hoc signed and not notarized by Apple. On first launch, right-click
**Open**, or allow the app under System Settings → Privacy & Security. Every
artifact is published with its SHA-256 so downloads can be checked independently.

## Publishing

Releases are published from the private source repository with
`scripts/publish-release.sh`, which packages the app, writes `latest.json`, and
uploads both files here with `gh release`.
