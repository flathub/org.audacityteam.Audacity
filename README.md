Flatpak notes
=============

This Flatpak packages Audacity 4 (Qt 6 / MuseScore `muse` framework). It is
a complete rewrite of the Audacity 3.x (wxWidgets) packaging; none of the old
modules or patches apply anymore.

How it is built
---------------

- Runtime: `org.kde.Platform` / `org.kde.Sdk` 6.11. It already provides every
  Qt module Audacity needs (Graphs, Quick3D, Core5Compat, NetworkAuth,
  ShaderTools, WebSockets, StateMachine).
- Source: the official release tarball `audacity-sources-<version>.tar.xz`
  from the Audacity GitHub releases. It contains the `muse` / `muse_deps`
  submodules and `offline-deps/downloads` (sources of all bundled
  dependencies), so the build needs no network access. See `OFFLINE_BUILD.md`
  inside the tarball.
- The `audacity` module is a single `cmake-ninja` module. Audacity's
  `muse_deps` builds its own dependencies (libsndfile, FLAC, opus, wxWidgets
  for the legacy AU3 code, ...) from `-DEXTDEPS_CACHE=<tarball>/offline-deps`.
- `x-checker-data` follows the latest GitHub release, so
  flatpak-external-data-checker can open update PRs with the new tarball URL
  and checksum.

Things worth knowing
--------------------

- `patchelf` module: `muse_deps` uses it to relocate the ELF files of the
  libraries it builds. It is not in the SDK.
- `patches/0001-extdeps-install-libs-to-lib.patch`: `muse_deps` sub-builds
  installed into `lib64` here, but its resolver only looks in `lib`. The patch
  passes `-DCMAKE_INSTALL_LIBDIR=lib`. Check on each update whether it still
  applies / is still needed.
- `cflags: -Wno-error=implicit-function-declaration`: the bundled wxWidgets
  zlib uses implicit function declarations, an error since GCC 14.
- Audacity 4 forces `QT_QPA_PLATFORM=xcb` (`src/app/main.cpp`), so the
  Flatpak needs `--socket=x11`; there is no native Wayland support. Do not
  use `fallback-x11` (it grants nothing on a Wayland session).
- Post-install fix-ups for the upstream desktop file and metainfo:
  - desktop entry name is "Audacity 4 Portable" upstream, set to "Audacity"
  - metainfo `launchable` points to `audacity.desktop`, fixed to
    `org.audacityteam.Audacity.desktop`
  - upstream ships no screenshot (mandatory for Flathub); one is injected
    (TODO: the current one is from Audacity 3, replace it)
  - these `sed` calls run before the builder renames `.appdata.xml` to
    `.metainfo.xml`
- `audacity.sh` wrapper: `LADSPA_PATH`, `VST3_PATH` and `LV2_PATH` are set in a
  wrapper script because variable substitution doesn't work in `finish-args`.
  The plugin extension point is `org.freedesktop.LinuxAudio.Plugins`.

Building locally
----------------

```
flatpak install --user flathub org.kde.Sdk//6.11 org.flatpak.Builder
flatpak run org.flatpak.Builder --user --force-clean --install-deps-from=flathub \
  --repo=repo build org.audacityteam.Audacity.yaml
flatpak run --command=flatpak-builder-lint org.flatpak.Builder manifest org.audacityteam.Audacity.yaml
flatpak build-update-repo repo
flatpak remote-add --user --no-gpg-verify local repo
flatpak install --user local org.audacityteam.Audacity
```

Known TODOs
-----------

- Replace the stopgap screenshot with an Audacity 4 one and fix the upstream
  release entry in the metainfo (upstream still lists "4.0.0 beta").
- Review `finish-args` (`--filesystem=host`, `--filesystem=/tmp`,
  `--share=network`).
- Test audio I/O, plugin loading (LV2/VST3/LADSPA) and `mod-script-pipe`.
- Ideas to upstream: `lib` vs `lib64` in `muse_deps`, the desktop entry name,
  the metainfo launchable/screenshot/release data, native Wayland.
