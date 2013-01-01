Integrating ROX Filer and XfCE
==============================

Here are some notes I've made (and supporting script) for integrating
ROX Filer and XFCE.

They were written on Xubuntu 12.04, with XfCE 4.10 installed from the PPA.
They should also work for Xubuntu 12.10 but I haven't tested that yet.

Other than the version of XfCE, I use the standard Xubuntu session
provided with Xubuntu 12.04, with some tweaks to settings.

The instructions assume you have a working copy of the ROX Filer
installed, and you can invoke it with `rox`, and `gigolo` installed for
mount management.

This is the case in Xubuntu if you install the `rox-filer` and `gigolo`
package from the official Ubuntu repository.

# Mounting remote filesystems

For mounting remote filesystems, I use `gigolo` the and `gvfs-fuse` packages, along with a small `perl` script I called `rox-wrapper`.

## Install needed programs
You need the `rox-wrapper` script somewhere in your `$PATH` (`/usr/local/bin` is
a good place).  See "The rox-wrapper script" below to find out why.

You'll need `perl` installed for that to work.

Install `gigolo` and `gvfs-fuse` from your package manager, and of course
`rox`.

## Set preferred file manager to rox-wrapper
From the XfCE Settings Manager, open the `Preferred Applications`
menu item, and open the `Utilities` tab.

Under `File Manager`, drop-down the combo box and select `Other`.

Enter the command line pattern below and press `OK`:

```
rox-wrapper "%s"
```

Note if `rox-wrapper` isn't somewhere in `$PATH` you'll need to specify the full path to it.

## Configure gigolo to use gvfs-open to open its filesystems
1. Open `gigolo`.
2. Click `Edit` -> `Preferences`
3. Replace whatever's in the `File Manager` box with `gvfs-open`.
4. Click `Close`.

## Usage
You should then be able to mount an SSH or SMB share in Gigolo using
`Actions` -> `Connect`, then double-click the resulting icon within to
have it open in ROX Filer.

The location opened in the Filer should be `~/.gvfs/SHARENAME` for remote
shares, or `/media/LABEL` for local filesystems mounted via gigolo.

Gigolo can handle mounting of encrypted partitions too - just make sure
you have `cryptsetup` installed.

You should use the `Disconnect` option in Gigolo to unmount any filesystems.

Put a launcher for `gigolo` on your XFCE Panel somewhere and you'll have
fairly quick access to all your remote and removable disks.


# The rox-wrapper script
`rox-wrapper` is a very thin wrapper that will take either a URI or a
normal path on the command line and launch `rox` with or without the `-U`
switch depending on what it is.

This makes it compatible with both `gvfs-open`, which passes straight paths
to it, and XFCE tools like `exo-open` that prefer to pass `file:///` URIs
to everything.  It will also handle running without arguments, for the
benefit of launchers that specify target directories by switching working
directory.

It's a `perl` script so you'll need perl installed.  It doesn't rely on any
CPAN modules at the moment.

You can find it in the `scripts` directory in this repository.

# Purpose

I like the "application is a directory" and direct drag-and-drop saving
approach to the "ROX" desktop, but find XFCE is the best window manager
to use with it.

I use the XFCE Panel and xfce4-desktop for the panels and desktops,
rather than the ROX Pinboard and ROX Iconbar, just because the plugins for
those are more up to date and stable in my opinion (and there are more of
them).  I think much of the ROX plugins have been left behind unfortunately.

# Limitations of this setup
If you `send to wastebasket` any items on the XFCE desktop, they still get
sent to the usual trash can that's managed by Thunar, so Thunar is still
needed to permanently delete or to restore anything in there.
An XDG-compliant Trash app for ROX, that picks up trash from Thunar, would
be very useful.

Also, Thunar is still pretty good for the availability of a really huge
icon view, which is good if you're browsing through detailed photos or
PDF documents.  A patch to ROX to allow for larger icons and thumbnails
in-view would be useful.


# My ROX config preferences
Here I document changes from the defaults that come with `rox-filer` on Ubuntu.  `[X]` means ticked.
## Filer Windows
* `[X]` Short titlebar flags
* `[X]` Unique windows
* `[X]` New window on button 1
* `[ ]` Single-click navigation
* `[X]` Double-click on background resizes
* `[X]` Directories come first (for sort by name)

The above matches very closely RISC OS Filer's behaviour (though I actually
settled on those choices before I tried RISC OS for real, so I'm not just
trying to ape it for the sake of it, but it does help me move between the
two systems quite well).

## Filer Windows : Display

* View Type: Icons View
* Sort by: Name
* Default size: Automatic
* Change at: 10 items
* Default details: No details

I wish the real RISC OS Filer had the ability to auto-change view at
a pre-defined number of items.

* `[X]` Order small icons vertically

## Filer Windows : Thumbnails

I don't tick `Show image thumbnails` - I right-click the 'eye' icon on
the toolbar if I want them.  This saves time generating the thumbnails when
I'm just passing through the directory.

## Action windows

Note to self: Might want to look at alternative 'mount', 'unmount' and 'eject' commands now gigolo is working.

## Menus

* Terminal emulator program: `xfce4-terminal`

## Compatiblity

* Drag and drop:
  * `[X]` Don't use hostnames

# My XfCE4 Window Manager preferences

## Window Manager : Focus

* `[X]` Automatically give focus to newly created windows
* `[ ]` Automatically raise windows when they receive focus
* `[ ]` Raise window when clicking inside application window

The 2nd and 3rd of those makes it easier to drag and drop between
overlapping windows.  If you need to bring a window forward and can't see
its border, hold down `Alt` and LEFT-click anywhere in the window.

## Window Manager Tweaks : Focus

* `[ ]` Activate focus stealing prevention

That makes sure that newly-opened terminal windows opened with the Backtick key
from rox get focus, so you can start typing in them immediately.

If you end up with things stealing focus that you don't want to, re-enable
this.

# Drag and Drop Loading

If you do `File` -> `Open...` within a GTK2 or GTK3 application, and drop
the file on the browser, it will navigate straight to the file,
ready for you to press Open.

Some GTK applications support loading of files by dragging and dropping a
file icon into the main area.  Others will insert the content of the
file wherever you drop it, while some will insert the path to the file
you dropped.

OpenOffice will insert an image in the document if you drop one in an open
document.

# Drag and Drop Save

Most applications from the ROX project support drag and drop saving.

See https://github.com/nmbooker/gtk-xds for dirty patches for GTK2 and GTK3
enabling basic drag and drop save support within any applications that use
the standard Gtk+ gtkfilechooser for saving.

You should be comfortable patching and compiling Debian and Ubuntu source
packages for personal use if you want that.

# Generating Application Directories for your favourite apps

## .desktop files
An ugly way is just to copy the `.desktop` file into your Apps folder.
You can do that by dragging and dropping entries from the menu.
ROX will display the icon, but will display the `blah.desktop` filename
directly, and it doesn't give you much opportunity to customise behaviour
when a file or stream is dropped on the application icon.

## Proper Application Directory wrappers
A good way to get a proper skeleton wrapper is to use the `AppFactory`
app from the ROX file manager.  Specify a binary name to run and drop an
icon for it, click Create, specify a name you want to see it as and drag-and-drop it into your Apps directory.  You can then modify the AppRun and AppInfo.xml files to customise the way it handles arguments.

Installing AppFactory and its dependency ROX-CLib2:

1. `mkdir ~/lib`
2. `rox ~/lib`
3. `sudo apt-get install build-essential libgtk2.0-dev libxml2-dev autoconf automake`  # or equivalent
4. Download ROX-CLib2 from http://sourceforge.net/projects/rox/files/ROX-CLib/
5. Extract the ROX-CLib2 directory in the archive into `~/lib`
6. Double-click ROX-CLib2 in the ROX window and, assuming you have your C compiler and all the gtk2 development headers installed, it should build.
7. Download and extract AppFactory from rox.sourceforge.net into your Apps directory.
8. Double-click AppFactory.  It should automatically build against your ROX-CLib2 and run.


# Evince "Open Containing Folder" feature
In Xubuntu Precise, Evince will fail to "Open Containing Folder", at
`exo-open` (so this isn't necessarily ROX specific though part of my fix
is).

It complains `permission denied` when it tries to run exo-open.

To fix this:

1. ```sudo cp apparmor.d/local/usr.bin.evince /etc/apparmor.d/local/usr.bin.evince```

2. ```sudo apparmor_parser -r /etc/apparmor.d/usr.bin.evince```  # [sic]
