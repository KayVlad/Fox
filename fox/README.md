# Fox personal Firefox fork

Based on Mozilla Firefox release, initially 160.0.1 at
`b59ea98ffcd5707003e24c773f7a81eaca13a625`.

## Repository

- `origin`: https://github.com/KayVlad/Fox.git
- `upstream`: https://github.com/mozilla-firefox/firefox.git
- Local development branch: `fox`
- The local clone contains a shallow release history to conserve disk space.

## Scope

Allow persistent unsigned WebExtensions and restore a content-process selector
with a shared-process mode. These runtime changes are not implemented yet.
Keep the OS content sandbox enabled. A content-process cap does not include
GPU, networking, extension, and other service processes.

## Build configuration

The ignored root `mozconfig` sources the tracked `fox/mozconfig`. Recreate it
with `. "$topsrcdir/fox/mozconfig"` as its single line when setting up a new clone.

The configuration uses unofficial branding, four build jobs, no debug symbols
configuration override, and disables tests and the Mozilla application updater.
Updates must be built from upstream; do not point this fork at Mozilla's binary
update service. Full source builds are required to validate signing and native
process-model changes. Artifact builds are insufficient for those changes.

`MOZ_REQUIRE_SIGNING` is explicitly empty so the runtime signing preference can
be changed. After building, `xpinstall.signatures.required=false` is still
needed to install unsigned extensions; this configuration alone does not change
that preference's default.

On a supported Linux build host:

```sh
./mach --no-interactive bootstrap --application-choice browser --no-system-changes
./mach build
./mach run --no-remote --profile /absolute/path/to/a/dedicated/fox-profile
```

Bootstrap with `--no-system-changes` installs user-local tooling but does not
supply missing system libraries or GNU make. The initial host is SteamOS 3.9.5;
GNU make and native development packages are missing. Use a supported development
container or install the necessary build dependencies before attempting a full
build. The checkout itself is ready for editing.

For tests, use a separate mozconfig with `--disable-tests` removed and a distinct
object directory. Do not reuse a daily browsing profile for development.

## Upstream updates

```sh
git fetch --depth=100 upstream release
git switch fox
git merge upstream/release
```

If Git cannot find a common ancestor, deepen the upstream history until the
previous base is present. Review conflicts, build, and test before pushing.
Avoid replacing the fork branch with upstream or force-pushing as an update
shortcut.

## Implementation checkpoints

1. Implement unsigned-extension defaults and verify installation survives restart.
2. Confirm shared-process support in this exact release before changing defaults.
3. Restore the selector and verify it controls content allocation after restart.
4. Check multiple sites, cross-site frames, private windows, and container tabs.
5. Measure total memory, swapping, responsiveness, and content-process count.
