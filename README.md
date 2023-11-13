Flatpak notes
=============

`LV2_PATH` is set in a wrapper script because variable substitution
doesn't work.

`VST_PATH` doesn't need to be set we patched the default location in
the source. See https://github.com/flathub/org.audacityteam.Audacity/issues/139

`patches/audacity-suil-fix.patch` is necessary for suil (LV2 UI) to
work. A fix was submitted upstream in the past but was ignored.
This version of the patch is highly specific to flatpak.

`screenshots`: before 3.4, Audacity pulled the rug by removing the
screenshot, and removing them from the appstream file. This is the
path of least resistance.
