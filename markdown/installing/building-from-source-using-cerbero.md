# Building from source using Cerbero

> ![Warning] This section is intended for advanced users.

## Build requirements

The GStreamer build system provides bootstrapping facilities for all
platforms, but it still needs a minimum base to bootstrap:

-   python >= 3.5
-   git

### Windows users

Cerbero can be used on Windows using the Msys/MinGW shell (a Unix-like
shell for Windows). There is a bit of setup that you need to do before
Cerbero can take control.

You need to install the following programs:

-   [Python 3.5+]
    * First page of the installer
     - Check "Add Python 3.x to PATH"
     - Click "Customize installation"
    * Second page, check "pip"
    * Third page, select:
     - Install for all users
     - Associate files with Python
     - Add Python to environment variables
     - Customize install location: C:\Python3
-   [Git]
    * Select the install option "Checkout as-is, Commit as-is"
    * Ensure that git is installed in PATH, but no other tools are
-   [Msys/MinGW] (Install it with all the options enabled)
-   [CMake] (Select the option "Add CMake in system path for the
    current user")
-   [Yasm] (Download the win32 or win64 version for your platform, name
    it `yasm.exe`, and place it in your MinGW `bin` directory,
    typically, `C:\MinGW\bin`)
-   [WiX 3.11.1]

Several packages that have Meson build files are now built by default with
Visual Studio, so you need to install Visual Studio 2015 or newer in the
default location. The Visual Studio Community build which is free for
open-source use can be installed at:

  * https://visualstudio.microsoft.com/vs/older-downloads/

You should add the cerbero git directory to the list of excluded folders in your
anti-virus, or you will get random build failures when Autotools does file
operations such as renames and deletions. It will also slow your build by
about 3-4x.

Your user ID can't have spaces (eg: John Smith). Paths with spaces are
not correctly handled in the build system and msys uses the user ID for
the home folder.

Cerbero must be run in the MinGW shell, which is accessible from the
main menu once MinGW is installed.

(Note that inside the shell, / is mapped to c:\Mingw\msys\1.0??)

### macOS users

To use cerbero on macOS you need to install the "Command Line Tools" from
XCode. They are available from the "Preferences" dialog under
"Downloads".

### iOS developers

If you want to build GStreamer for iOS, you also need the iOS
SDK. The minimum required iOS SDK version is 6.0 and is included in
[XCode] since version 4.

## Download the sources

To build GStreamer, you first need to download **Cerbero**.
Cerbero is a multi-platform build system for Open Source projects that
builds and creates native packages for different platforms,
architectures and distributions.

Get a copy of Cerbero by cloning the git repository:

    git clone https://gitlab.freedesktop.org/gstreamer/cerbero

Cerbero can be run uninstalled and for convenience you can create an
alias in your `.bashrc` file*.??*If you prefer to skip this step,
remember that you need to replace the calls to `cerbero`??with
`./cerbero-uninstalled`??in the next steps.

    echo "alias cerbero='~/git/cerbero/cerbero-uninstalled'" >> ~/.bashrc

## Setup environment

After Cerbero and the base requirements are in place, you need to setup
the build environment.

Cerbero reads the configuration file `$HOME/.cerbero/cerbero.cbc` to
determine the build options. This file is a python code which allows
overriding/defining some options.

If the file does not exist, Cerbero will try to determine the distro you
are running and will use default build options such as the default build
directory. The default options should work fine on the supported
distributions.

To fire up the bootstrapping process, go to the directory where you
cloned/unpacked Cerbero and type:

    cerbero bootstrap

Enter the superuser/root password when prompted.

The bootstrap process will then install all packages required to build
GStreamer.

## Build GStreamer

To generate GStreamer binaries, use the following command:

    cerbero package gstreamer-1.0

This should build all required GStreamer components and create packages for
your distribution at the Cerbero source directory.

A list of supported packages to build can be retrieved using:

    cerbero list-packages

Packages are composed of 0 (in case of a meta package) or more
components that can be built separately if desired. The components are
defined as individual recipes and can be listed with:

    cerbero list

To build an individual recipe and its dependencies, do the following:

    cerbero build <recipe_name>

Or to build or force a rebuild of a recipe without building its
dependencies use:

    cerbero buildone <recipe_name>

To wipe everything and start from scratch:

    cerbero wipe

Once built, the output of the recipes will be installed at the prefix
defined in the Cerbero configuration file `$HOME/.cerbero/cerbero.cbc`
or at `$HOME/cerbero/dist` if no prefix is defined.

### Build a single project with GStreamer

Rebuilding the whole GStreamer is relatively fast on Linux and OS X, but it
can be very slow on Windows, so if you only need to rebuild a single
project (eg: gst-plugins-good to patch qtdemux) there is a much faster
way of doing it. You will need to follow the steps detailed in this
page, but skipping the step "**Build GStreamer**", and installing the
GStreamer's development files as explained in [Installing GStreamer].

By default, Cerbero uses as prefix a folder in the user directory with
the following schema \~/cerbero/dist/$platform\_$arch, but for GStreamer
we must change this prefix to use its installation directory. This can
be done with a custom configuration file named *custom.cbc*:

    # For Windows x86
    prefix='/c/gstreamer/1.0/x86/'

    # For Windows x86_64
    #prefix='/c/gstreamer/1.0/x86_64'

    # For Linux
    #prefix='/opt/gstreamer'

    # For OS X
    #prefix='/Library/Frameworks/GStreamer.framework/Versions/1.0'

The prefix path might not be writable by your current user. Make sure
you fix it before, for instance with:

    $ sudo chown -R <username> /Library/Frameworks/GStreamer.framework/

Cerbero has a shell command that starts a new shell with all the
environment set up to target GStreamer. You can start a new shell using
the installation prefix defined in *custom.cbc??*with the following
command:

    $ cerbero -c custom.cbc shell

Once you are in Cerbero's shell you can compile new projects targeting
GStreamer using the regular build process:

    $ git clone https://gitlab.freedesktop.org/gstreamer/gst-plugins-good; cd gst-plugins-good
    $ sh autogen.sh --disable-gtk-doc --prefix=<prefix>
    $ make -C gst/isomp4

### Cross-compilation of GStreamer

Cerbero can be used to cross-compile GStreamer to other platforms like
Android or Windows. You only need to use a configuration file that sets
the target platform, but we also provide a set of pre-defined
configuration files for the supported platforms (you will find them in
the `config` folder with the `.cbc` extension.

#### Android

You can cross-compile GStreamer for Android from a Linux host using the
configuration file `config/cross-android.cbc`. Replace all the previous
commands with:

    cerbero -c config/cross-android.cbc <command>

#### Windows

GStreamer can also be cross-compiled to Windows from Linux, but you should
only use it for testing purpose. The DirectShow plugins cannot be
cross-compiled yet and WiX can't be used with Wine yet, so packages can
only be created from Windows.

Replace all the above commands for Windows 32bits with:

    cerbero -c config/cross-win32.cbc <command>

Or with using the following for Windows 64bits:

    cerbero -c config/cross-win64.cbc <command>

#### iOS

To cross compile for iOS from OS X, use the configuration file
`config/cross-ios-universal.cbc`. Replace all previous commands with:

    cerbero -c config/cross-ios-universal.cbc <command>

  [Warning]: images/icons/emoticons/warning.svg
  [Python 3.5+]: https://www.python.org/downloads/
  [Git]: https://github.com/git-for-windows/git/releases/latest
  [Msys/MinGW]: https://sourceforge.net/projects/mingw/files/Installer/mingw-get-inst/
  [CMake]: http://www.cmake.org/cmake/resources/software.htm
  [Yasm]: http://yasm.tortall.net/Download.html
  [WiX 3.11.1]: https://github.com/wixtoolset/wix3/releases/tag/wix3111rtm
  [XCode]: https://developer.apple.com/devcenter/ios/index.action#downloads
  [Installing GStreamer]: installing/index.md
