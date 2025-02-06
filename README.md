 
# Build ReactOS with MSVC on Wine
This is about building ReactOS using MSVC compiler on Wine. While it's possible, it's not very efficient and not by any means officially supported. Some of these workarounds are also need for ROSBE MinGW build if you want to use it through Wine for some reason.

## Prerequisites
You will really need to <a href="https://github.com/lutris/docs/blob/master/WineDependencies.md">install Wine dependencies</a> for this to work properly. Note that if you don't use a distribution with multilib/multiarch, Wine should use its new experimental WoW64 support.

## Installing Visual Studio 2019
You can apply all required dependencies and tweaks using winetricks:
`winetricks -q dotnet48 renderer=vulkan win7`

If you don't have a GPU that supports Vulkan then either install the lavapipe ICD (and also its x86 build if you're using multilib/multiarch) or otherwise invoke `d3dcompiler_47` instead of `renderer=vulkan` in winetricks.

Now download Visual Studio installer and run it, once it loads check "Desktop development with C++" and install.

Visual Studio IDE won't load, but MSVC build tools should be available once you run build tools command prompt.

## Building ReactOS on Wine

1. Run `C:\Program Files\Microsoft Visual Studio\2019\Community\VC\Auxiliary\Build\vcvarsamd64_x86.bat` (vcvars64.bat for amd64 build target arch)
2. CD to reactos repo folder
3. Set necessary environment variables (if you build with ROSBE, you also need replace bison/flex/m4 with those from build-tools)
   * `set PATH=%PATH%;path\to\build-tools\bin`
   * `set BISON_PKGDATADIR=path\to\build-tools\share\bison`
   * `set M4=path\to\build-tools\bin\m4.exe`
4. run configure.cmd and proceed as usually

## Known issues
* Build takes much more time than on Windows
* Build tools for x86 host architecture are unreliable, it is recommended to use x64 tools if possible
* Cross-compiling ReactOS for ARM targets seems to be broken.
* Visual Studio installer might break (returning "The install operation failed" message) if Wine Windows version is set to Windows 10. Set the Windows version to Windows 7 (via winecfg or winetricks win7).
* Visual Studio installer is not able to update itself in Wine. To update the installer, simply download the latest version <a href="https://visualstudio.microsoft.com">from here</a> and install it. Now the installer should download and install latest Visual Studio updates.

## Visual Studio 2022
Due to restricted availbility of older Visual Studio versions (thanks MS), you may want to use Visual Studio 2022 instead, however that currently requires the patched Wine build.
* Apply <a href="https://gitlab.winehq.org/wine/wine/-/merge_requests/6288">this patch</a> and build Wine.
* Proceed as with VS2019 setup, with the exception of dropping `win7` from the `winetricks` command.

## WinDBG on Wine
To run WinDBG on Wine:

0. If you are looking just for WinDBG on Wine (without installing VS2019 beforehand, otherwise skip this step)
   * `winetricks -q dotnet48 renderer=vulkan`
   * Or if you don't have a Vulkan ICD: `winetricks -q dotnet48 d3dcompiler_47`
1. Run `winetricks -q xmllite corefonts` (probably needed only for the new WinDBG)
2. add DLL overrides (Native then Builtin) for `dbghelp` and `dbgeng` in winecfg.

While both classic WinDBG from Windows SDK and the new WinDBG run on Wine, the latter seems to work much better and is much more usable.

You can download new WinDBG <a href="https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/">from here</a>. The download link is an Uri value under MainBundle section in `windbg.appinstaller` file.
To run new WinDBG, extract the `windbg.msixbundle` file, then extract `windbg_win7-<your_host_architecture>.msix` and run `DbgX.Shell.exe` binary.

You can specify serial devices in Wine registry; in `HKLM\Software\Wine\Ports` add string value named `COM[number]` which contains serial device path (eg. `/dev/ttyS0`). Shut down the Wine server (`wineserver -k` in terminal) to apply the serial device properties in registry.

In order to debug a VM guest, you can pipe the VM Unix socket to Wine via `socat`.
* Set the serial device path in Wine registry to a PTY that socat will create (eg. /tmp/wine-port) and run `wineserver -k`.
* Set the Unix socket path in VM (eg. /tmp/socket) and start the VM.
* Run `socat UNIX-CONNECT:/tmp/socket PTY,link=/tmp/wine-port`

Now you should be able to attach WinDBG to the VM.

If new WinDBG crashes when breaking into debugger, it could be due to host system setting LANG environment variable to some region specific value that is not handled by WinDBG (like en-DK), try setting `LANG=en-US`.

Note that kdusb and kd1394 are not going to work as WinDBG depends on additional Windows kernel drivers on the debugger machine for these.

## Used software
* Visual Studio 2019 - 16.11.40
* Visual Studio 2022 - 17.12.4
* WinDBG 1.2410.11001.0
* Wine Staging 10.0
