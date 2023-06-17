 
# Build ReactOS with MSVC on Wine
This is about building ReactOS using MSVC compiler on Wine. While it's possible, it's not very efficient and not by any means officially supported. Some of these workarounds are also need for ROSBE MinGW build if you want to use it through Wine for some reason.

## Installing Visual Studio 2022
You can apply all required dependencies and tweaks using winetricks:
`winetricks -q cmd dotnet48 renderer=vulkan win7`
Now download Visual Studio installer (<a href="https://visualstudio.microsoft.com">from here</a>), run it, once it loads check "Desktop development with C++" and install.
Visual Studio IDE won't load, but MSVC build tools should be available once you run build tools command prompt.

## Building ReactOS on Wine
You will need to apply patch which contains workarounds for wine (it only modifies the configure.cmd script).

* Apply 0001-Wine-workarounds.patch on ReactOS git repo
* Run C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsamd64_x86.bat (vcvars64.bat for amd64 build target arch)
* CD to reactos repo folder
* Set necessary environment variables (if you build with ROSBE, you also need replace bison/flex/m4 with those from build-tools)
- `set PATH=%PATH%;path\to\build-tools\bin`
- `set BISON_PKGDATADIR=path\to\build-tools\share\bison`
- `set M4=path\to\build-tools\bin\m4.exe`
* run configure.cmd and proceed as usually

## Known issues
* Build takes much more time than on Windows
* Build tools for x86 host architecture are unreliable, it is recommended to use x64 tools if possible
* Cross-compiling ReactOS for ARM targets seems to be broken.
* Visual Studio installer might break (returning "The install operation failed" message) if Wine Windows version is set to Windows 10. Set the Windows version to Windows 7 (via winecfg or winetricks win7).
* Visual Studio installer is not able to update itself in Wine. To update the installer, simply download the latest version <a href="https://visualstudio.microsoft.com">from here</a> and install it. Now the installer should download and install latest Visual Studio updates.

## WinDBG on Wine
To run WinDBG on Wine:
* Run `winetricks -q xmllite` (probably needed only for WinDBGX)
* add DLL overrides (Native then Builtin) for dbghelp and dbgeng in winecfg.

You can use both usual WinDBG from Windows SDK or the new WinDBGX. You can download WinDBGX <a href="https://learn.microsoft.com/en-us/windows-hardware/drivers/debugger/">from here</a>. The download link is an Uri value under MainBundle section in windbg.appinstaller file.
To run WinDBGX, extract the windbg.msixbundle file, then extract windbg_win7-<your_host_architecture>.msix and run DbgX.Shell.exe binary.
You can specify serial devices in Wine registry; in HKLM\Software\Wine\Ports add string value named `COM[number]` which contains serial device path (eg. /dev/ttyS0). Shut down the Wine server (wineserver -k in terminal) to apply the serial device properties in registry.

## Used software
* Visual Studio 2022 - 17.6.3
* WinDBGX 1.2306.12001.0
* Wine Staging 8.10
