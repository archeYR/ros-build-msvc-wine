 
# Build ReactOS with MSVC on Wine
This is about building ReactOS using MSVC compiler on Wine. While it's possible, it's not very efficient and not by any means officially supported. Some of these workarounds are also need for ROSBE MinGW build if you want to use it through Wine for some reason.

## Installing Visual Studio 2022
You can install all required dependencies using winetricks:
`winetricks -q d3dcompiler_47 dotnet48 arial cmd`
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
## WinDBG on Wine
To run WinDBG on Wine:
* Run `winetricks -q xmllite`
* add DLL overrides (Native then Builtin) for dbghelp and dbgeng in winecfg.

You can use both usual WinDBG from Windows SDK or WinDBGX from MS "Store" (you can download it from store.rg-adguard.net).
You can specify serial devices in Wine registry; in HKLM\Software\Wine\Ports add string value named `COM[number]` which contains serial device path (eg. /dev/ttyS0).

## Used software
* Visual Studio 2022 - 17.4.4
* WinDBGX 1.2210.3001.0
* Wine Staging 7.22

## Updating Visual Studio
Visual Studio installer is not able to update itself in Wine. To update the installer, simply download the latest version <a href="https://visualstudio.microsoft.com">from here</a> and install it. Now the installer should download and install latest Visual Studio updates.
