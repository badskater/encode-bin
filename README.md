# encode-bin

Source of truth + distribution for the **node tools folder** (`C:\bin`) of
the [encode-system](https://github.com/badskater/encode-system) farm —
all public builds (x265 fork, eac3to, opusenc, mkvmerge, MediaInfo,
DGIndexNV, ffmpeg, dovi_tool, SCXvid, …).

## How distribution works

The package ships as a **GitHub Release asset**, never as a committed git
file (144 MB > GitHub's 100 MB per-commit-file limit):

- Tag/release naming: `v<bin version>` — the number is the fleet's
  `bin_version` counter (the controller's publish API requires strictly
  increasing versions).
- Asset: `bin-package.zip` (the zipped contents of a node's `C:\bin`).
- The encode-system controller can pull it straight from the release URL
  (`POST /api/updates/bin/url` on the controller, or Settings → Push to
  nodes → *Fetch from URL* in the WebUI) and then pushes it to the fleet;
  nodes adopt it on their next idle heartbeat (SHA-256 verified on the node).

## Publishing a new version

1. Update the tools on a node (`C:\bin`).
2. Zip the folder on that node:
   `Compress-Archive -Path C:\bin\* -DestinationPath C:\bin-package.zip -CompressionLevel Optimal`
3. Copy the zip to a machine with `gh` auth and publish:
   ```bash
   gh release create v<N> ./bin-package.zip \
     --repo badskater/encode-bin \
     --title "bin v<N>" \
     --notes "<what changed>"
   ```
4. On the controller: Settings → Push to nodes → *Fetch from URL* with
   `https://github.com/badskater/encode-bin/releases/download/v<N>/bin-package.zip`
   and version `<N>` — the fleet converges automatically.

## Integrity

Each release notes the package SHA-256; the controller validates the zip at
publish time (zip-slip/symlink/drive-path guards) and every node re-verifies
the SHA-256 after download before extracting.
