# camsnap Homebrew Release Playbook

This mirrors the lightweight Homebrew flow we use in other CLIs (e.g., `peekaboo`), but only targets our tap (no npm).

## 0) Prereqs
- macOS with Homebrew installed.
- Clean git working tree on `main`.
- Go toolchain installed (Go version is read from `go.mod`).
- Access to the tap repo (e.g., `steipete/homebrew-tap`).

## 1) Verify build is green
```sh
make fmt
golangci-lint run ./...
go test ./...
```

## 2) Bump the version in code
Edit `cmd/camsnap/main.go` and set `var version = "x.y.z"`.

## 3) Tag & push
```sh
git commit -am "release: vX.Y.Z"
git tag vX.Y.Z
git push origin main --tags
```

## 4) Verify the Homebrew tap formula
The release workflow dispatches `update-formula.yml` in `steipete/homebrew-tap` with:
```sh
artifact_template='{formula}_{version}_{target}.tar.gz'
```

Confirm the workflow succeeds and `Formula/camsnap.rb` contains matching URLs and checksums for:
- `darwin_amd64`
- `darwin_arm64`
- `linux_amd64`
- `linux_arm64`

If the automatic dispatch fails, rerun `update-formula.yml` with `formula=camsnap`, `tag=vX.Y.Z`, `repository=steipete/camsnap`, and the artifact template above.

## 5) Sanity-check install from tap
```sh
brew uninstall camsnap || true
brew untap steipete/tap || true
brew tap steipete/tap
brew install steipete/tap/camsnap
brew test steipete/tap/camsnap
camsnap --version
```

## 6) Announce
- Create GitHub Release for tag `vX.Y.Z` (link changelog).
- Optionally post in team channel with upgrade command: `brew update && brew upgrade steipete/tap/camsnap`.

## Notes
- Release automation updates the tap from the matching GoReleaser asset for each supported macOS and Linux architecture.
- Keep the tap formula small: version, url, sha256, license, dependencies.
