Flatpak notes
=============

`LV2_PATH` is set in a wrapper script because variable substitution
doesn't work.

There are many patches to allow Audacity to build or work:

- `patches/audacity/0001-flatpak-Hardcode-the-proper-path-to-suil.patch`:
  is necessary for suil (LV2 UI) to work. A fix was submitted upstream
  in the past but was ignored.  This version of the patch is highly
  specific to flatpak.
- `patches/audacity/0002-c-requiring-std-fs-is-an-ubuntu-ism.patch`:
  Because of ubuntu they add libstdc++fs to the link line without
  checking.  It's not in the SDK. It's not needed either.
- `patches/audacity/0003-appstream-Add-mandatory-screenshot.patch`:
  before 3.4, Audacity pulled the rug by removing the screenshot, and
  removing them from the appstream file. This is the path of least
  resistance.
- `patches/audacity/0004-vst-Set-proper-path-since-it-ignore-VST_PATH.patch`:
  `VST_PATH` doesn't need to be set (it's ignored). We patched the
  default location in the source. See
  https://github.com/flathub/org.audacityteam.Audacity/issues/139
- `patches/audacity/0005-cmake-CMake-hate-compatibility.patch`: In
  25.08 CMake is incompatible. And the build system call cmake to
  build the VST3 SDK so we need to patch too.

There are a few patch for the VST3 SDK. Due to the nature of git
submodules, there are more patches that necessary.

- `patches/vst3-cmake/0001-build-don-t-add-libstdc-fs.patch`
- `patches/vst3-public-sdk/0001-vst-Fix-gcc-version-test.patch`: Same as
  above, but we also need to fix the version detection.
- `patches/vst3-pluginterfaces/0001-c-gcc-compatibility-fix.patch`
- `patches/vst3-public-sdk/0002-c-gcc-compatibility-fix.patch`: These
  2 patches fix includes with newer gcc.

To update the patches the follow github repositories will contain the
patchsets:
https://github.com/hfiguiere/vst3_cmake.git
https://github.com/hfiguiere/vst3_pluginterfaces.git
https://github.com/hfiguiere/vst3_public_sdk.git

Checkout the branch `audacity-flatpak-patches`
