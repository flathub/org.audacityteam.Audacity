Audacity Flatpak
================

Flatpak packaging for [Audacity](https://www.audacityteam.org/) 4
(Qt 6 / MuseScore `muse` framework).

How it is built
---------------

- **Runtime:** `org.kde.Platform` / `org.kde.Sdk` 6.11. It provides all Qt
  modules Audacity needs (Graphs, Quick3D, Core5Compat, NetworkAuth,
  ShaderTools, WebSockets, StateMachine).
- **Source:** the official release tarball `audacity-sources-<version>.tar.xz`
  from the Audacity GitHub releases. It contains the `muse` / `muse_deps`
  submodules and `offline-deps/downloads` (sources of all bundled
  dependencies), so no network access is needed during the build. See
  `OFFLINE_BUILD.md` inside the tarball.
- **Build:** a single `cmake-ninja` module. Audacity's `muse_deps` builds its
  own dependencies (libsndfile, FLAC, opus, wxWidgets for the legacy AU3 code,
  ...) from `-DEXTDEPS_CACHE=<tarball>/offline-deps`.
- **Updates:** `x-checker-data` follows the latest GitHub release, so
  flatpak-external-data-checker opens update PRs with the new tarball URL and
  checksum. The `ffmpeg` module is tracked through Anitya.

Details worth knowing
---------------------

- `patchelf` module: `muse_deps` uses it to relocate the ELF files of the
  libraries it builds. It is not part of the SDK.
- `patches/0001-extdeps-install-libs-to-lib.patch`: the `muse_deps` sub-builds
  installed into `lib64` here, but its resolver only looks in `lib`. The patch
  passes `-DCMAKE_INSTALL_LIBDIR=lib`. Check on each update whether it still
  applies or is still needed.
- `cflags: -Wno-error=implicit-function-declaration`: the bundled wxWidgets
  zlib uses implicit function declarations, which is an error since GCC 14.
- `ffmpeg` module: the runtime's FFmpeg cannot be used. Qt Multimedia already
  loads the runtime's `libavcodec.so.61`, so the codecs-extra
  `libavformat.so.61` fails to bind (`undefined symbol
  avpriv_h264_has_num_reorder_frames`). FFmpeg 8 (`.so.62`) avoids the clash;
  Audacity supports it. `--disable-ffnvcodec` is needed because nvenc does not
  build against the SDK's newer `nv-codec-headers`, and GPU video encoding is
  irrelevant for audio.
- Audacity 4 forces `QT_QPA_PLATFORM=xcb` (`src/app/main.cpp`), hence
  `--socket=x11`. There is no native Wayland support; do not use
  `fallback-x11` (it grants nothing on a Wayland session).
- Post-install fix-ups for upstream's desktop file and metainfo:
  - the desktop entry is named "Audacity 4 Portable" upstream, set to
    "Audacity", and `Exec` uses `%F` so files are passed as paths
  - the metainfo `launchable` points to `audacity.desktop`, fixed to
    `org.audacityteam.Audacity.desktop`
  - upstream ships no screenshot (mandatory for Flathub), so one is injected,
    pointing to `screenshots/main-window.jpg` in this repository
  - the `sed` calls run before the builder renames `.appdata.xml` to
    `.metainfo.xml`
- `audacity.sh` wrapper: sets `LADSPA_PATH`, `VST3_PATH` and `LV2_PATH` for the
  plugin extension point `org.freedesktop.LinuxAudio.Plugins`, because variable
  substitution does not work in `finish-args`.

Permissions
-----------

- `--filesystem=home`: projects (`.aup4`) are SQLite databases, and SQLite
  needs real access to the directory next to the file (journal and temp
  files). Through the document portal, saving fails with
  `SQLite error (3850): disk I/O error` on `/run/user/<uid>/doc/...`, and
  "Open with" forwards files the same way. VST3 plugins in `~/.vst3` are found
  through the same access.
- `--socket=x11`: see above.
- `--socket=pulseaudio` and `--filesystem=xdg-run/pipewire-0:ro`: audio
  (PortAudio is built with JACK, which goes through PipeWire).
- `--share=network`: audio.com cloud features and usage information.
- `--device=dri`, `--share=ipc`: rendering.

Known limitations
-----------------

- VST3 plugins are only found in `~/.vst3`, `/usr/lib/vst3`,
  `/usr/local/lib/vst3` and next to the application. Audacity ignores
  `VST3_PATH`, so VST3 plugins installed through the
  `org.freedesktop.LinuxAudio.Plugins.*` extensions are not discovered. A patch
  adding `/app/extensions/Plugins/vst3` to the search paths would fix this.
- LV2 and LADSPA plugins from the extension point are untested.

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

Ideas for upstream
------------------

Changes in Audacity that would let this Flatpak drop workarounds:

- `muse_deps`: install dependency libraries to `lib` regardless of the
  platform's default libdir.
- Desktop entry name ("Audacity 4 Portable") and metainfo `launchable`,
  screenshots.
- Native Wayland support instead of the forced xcb platform.
- Honor `VST3_PATH` in the Linux VST3 module search.
