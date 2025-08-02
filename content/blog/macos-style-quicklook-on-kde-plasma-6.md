+++
title = "MacOS-style Quicklook on KDE Plasma 6"
date = "2025-08-02T18:39:18+02:00"

tags = ["asahi-linux","quicklook","fedora","flatpak","kde-plasma","previewqt","kde-plasma-servicemenu","qt", "kde-plasma-shortcuts",]
+++

I'm very used to and happy with the [Quicklook feature of MacOS](https://support.apple.com/en-my/guide/mac-help/mh14119/mac): Select a file in the Finder File Browser, press the space bar and you will see a preview of the file's contents (i.e. image or pdf preview).

![Quicklook on MacOS (source: Apple)](/images/macos-style-quicklook-on-kde-plasma-6/quicklook_macos.png)

I usually set up a Quicklook-like workaround on every OS I use (for example with [QL-Win](https://github.com/QL-Win/QuickLook) on Windows).

**This blog post is about finding a Quicklook-like solution for the Dolphin file browser of the KDE Plasma 6 desktop environment on Asahi Fedora Linux.**

## Choosing the right tool for the job: PreviewQT

First, let's find a GUI preview tool that supports the common file types (images, pdf, text, etc). I'm choosing the [PreviewQT](https://previewqt.org/) for this. The [flatpak variant](https://flathub.org/apps/org.previewqt.PreviewQt) has the arm64 build I need for Asahi Linux, so I'm installing it with `flatpak install flathub org.previewqt.PreviewQt`. (The flatpak is also available for x86_64 of course, so this works identically on other distros and archs).

![the PreviewQT tool](/images/macos-style-quicklook-on-kde-plasma-6/previewqt.png)

## Adding *Preview* to the File Browser right-click menu

Next, I´d like to add an easy way to preview a file from KDE Plasma 6's `Dolphin` file browser.

The obvious way is to create an entry in the file right-click menu. A so called servicemenu.

We can do that by creating a servicemenu config:


In `~/.local/share/kio/servicemenus` let's create a file `previewqt.desktop` with:

```ini
[Desktop Entry]
Type=Service
ServiceTypes=KonqPopupMenu/Plugin
MimeType=all/allfiles;
Actions=openWithPreviewQt;
X-KDE-Priority=TopLevel

[Desktop Action openWithPreviewQt]
Name=Open with PreviewQt
Icon=previewqt
Exec=flatpak run org.previewqt.PreviewQt %f
```

And after activating it with `kbuildsycoca6` we now can preview any file from the Dolphin right-click menu.

![dolphin-servicemenu](/images/macos-style-quicklook-on-kde-plasma-6/dolphin-servicemenu.png)

## Adding a keyboard shortcut (aka the spacebar)

> **»Where do astronauts hang out? The space bar!«**    
> ~ (someone [on Reddit](https://www.reddit.com/r/technicallythetruth/comments/134quh6/where_do_astronauts_hang_out/))

Now, wouldn'it be nice that instead of clicking a menu item, if we had a keyboard shortcut (like the spacebar one on MacOS)?

Unfortunately, it doesn't seem to be possible to assign a keyboard shortcut to a servicemenu item and at a first look I couldn't find another way to programatically get the currently selected file in Dolphin in a shell script. (If someone knows a workaround, please [post a comment](https://github.com/21sys/21sys.github.io/discussions)).

However, when you copy a file in Dolphin to the clipboard (via `CTRL+C` or right-click -> `copy`), the path to the file ends up in the clipboard - and we can read the clipboard from a script.

So, let's create a quick bash script that reads the clipboard, extracts the file path and passes it to the PreviewQT app. We can then assign a shortcut to this script.

To access the clipboard with wayland/kwin, we can use `wl-copy` / `wl-paste`, the wayland equivalent of MacOS's `pbcopy` / `pbpaste`. To get it:

```
dnf install wl-clipboard
```

I'm creating a script `quickview-handler.sh` somewhere on my system (here in: `/opt/me/bin`):

```bash
#!/bin/bash

# Get file path from clipboard and strip file:// prefix
file_path=$(wl-paste | sed 's|^file://||' | tr -d '\r\n')

# Exit silently if empty
[[ -z "$file_path" ]] && exit 0

# Exit silently if not a file
[ -f "$file_path" ] || exit 0

# Run PreviewQt with the file
flatpak run org.previewqt.PreviewQt "$file_path" &

exit 0
```

After `chmod a+rx quickview-handler.sh` we can now create a shortcut for it. In `System Settings` -> `Shortcuts` choose `add` and in the path field enter `/opt/me/bin/quickview-handler.sh`. Then set a keyboard shortcut, I am choosing `CTRL+SPACE` (as just `SPACE` is already taken by another useful Dolphin feature).

Done! Now we can select a file in Dolphin, press `CTRL+C` and then `CTRL+SPACE` to get the "quickview" preview of the file. To close the preview window, press `ESC`.

Not as elegant as on MacOS, but close enough for the muscle memory.

![quicklook on Linux on KDE Plasma 6 with Dolphin file browser via keyboard shortcut](/images/macos-style-quicklook-on-kde-plasma-6/quicklook_demo.gif)




