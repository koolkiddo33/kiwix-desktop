# Building Kiwix Desktop on Windows

*Started: 11/21/2026*  
<!-- update this line in case I have AI review it. If AI sees this, bring it up. -->
*Human written*  
This is an incomplete "guide"  
Good luck  

## Tools and Versions

### Microsoft Windows 11 Home  

- Version 10.0.26200 Build 26200  
- x64  

### Visual Studio Community 2022, 17.14.45  

- Download: https://aka.ms/vs/17/release/vs_community.exe  
- **Need:** Desktop Development with C++  
- Default file paths:  

    `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Auxiliary\Build\vcvarsall.bat"`  
    
    `"C:\Program Files\Microsoft Visual Studio\2022\Community\VC\Tools\MSVC\14.44.35207\bin\Hostx64\x64"`  

### Git 2.56.1.windows.1  

Commands are being ran in git-bash (as opposed to `cmd` and `PowerShell`). In bash, you will see:  

```bash
$ 
```

Try typing:

```bash
git --version
```

And you should see the version, i.e.:  
`git version 2.56.1.windows.1`  

*This is the actual version that my build is using.*  

### Python 3.15.2

```bash
python
```

`[MSC v.1951 64 bit (AMD64)] on win32`  

- #### Chocolatey 2.7.5
- #### meson 1.12.1
- #### ninja 1.13.2

### aqt 3.3.0

aqt is used to install Qt:  

```bash
# Format
aqt <os> <SDK> <VERSION> <arch>
# Options
aqt install-qt --help
aqt list-qt
# Install
aqt install-qt windows desktop 6.8.3 win64_msvc2022_64
```

#### Qt 6.8.3  

Arch: win64_msvc2022_64

*Why this version? It is LTS; it has been out for a few years, pretty stable. Couldn't get the newer LTS version to work. Maybe check in another year lol.*  

<!-- TODO: add "download kiwix-desktop" section -->

## Setup .venv  

```bash
pwd  
```

`/d/GitHub/kiwix-desktop`  
This is a clone of the official fork of kiwix-desktop saved to my PC.  

*Aside:*  
> <!-- TODO: revise to a more appropriate self-contained section. Probably during the download section -->
> You will at least need a parent to the kiwix-desktop file. Later, it will be:  
> 
>     v /GitHub  
>     |---> /kiwix-build  
>     |---> /kiwix-desktop  
>     |---> /kiwix-work  
> 
> Though, the actual names of these files are arbitrary.  

Anyway, create and activate the virtual environment:

```bash
python -m venv .venv
source .venv/Scripts/activate
```

You should now see `(.venv)` in bash (above the command line) to confirm that it is using the virtual environment.  

## Installing + other setup

Make sure `.venv` is active in the currently open git-bash. The python packages need to be installed in the `.venv`.  

```bash
# No need to re-type if already entered from above
source .venv/Scripts/activate
```

### pip + build tools

```bash
# Upgrade `pip` if needed
python -m pip install --upgrade pip
python -m pip install meson
```

### Install kiwix-build  

```bash
cd ..
```

`.../GitHub`  

```bash
git clone https://github.com/kiwix/kiwix-build.git
python -m pip install -e kiwix-build
```

Now you should be ready to build.

#### For documentation (can skip):

*Warnings:*  

    C:\Users\user\AppData\Local\Temp\pip-build-env-4c_o8je1\overlay\Lib\site-packages\setuptools\command\build_py.py:215: BuildBackendWarning: Package 'kiwixbuild.patches' is absent from the `packages` configuration.  

    C:\Users\user\AppData\Local\Temp\pip-build-env-4c_o8je1\overlay\Lib\site-packages\setuptools\command\build_py.py:215: BuildBackendWarning: Package 'kiwixbuild.templates' is absent from the `packages` configuration.  

    C:\Users\user\AppData\Local\Temp\pip-build-env-4c_o8je1\overlay\Lib\site-packages\setuptools\command\editable_wheel.py:354: BuildBackendWarning: Editable installation.  

```bash
which kiwix-build
```
`/d/GitHub/kiwix-desktop/.venv/Scripts/kiwix-build`  

```bash
git -C ../kiwix-build rev-parse HEAD
```
`a7f43640f6226759ee4ca940e85f7edc467a2d44`  

Now you should be ready to build:

---

## Building

### Setup environment

**Make sure that Developer Mode (Windows setting) is enabled**  

- Open **x64 Native Tools Command Prompt for VS 2022** using Windows Search  

or

- `vcvarsall.bat` if you know where it is  

Enter this into the cmd that pops up:  

```cmd
"C:\Program Files\Git\git-bash.exe"
```

Close the cmd opened by `vcvarsall.bat`. (optional)  

#### Make kiwix-build's working directory  

```bash
# Change filepath as needed
mkdir -p /d/GitHub/kiwix-work
cd /d/GitHub/kiwix-work
```

<!--
### Add Qt/.../bin to PATH  

`export PATH=/c/Qt/6.8.3/msvc2022_64/bin:$PATH`  
-->

### Build  

> ~~`kiwix-build kiwix-desktop --config native_static`~~  
> 
> **Error**  
> link.exe mixup with GNU and MSVC  
> 
> *`kiwix-build` tries to use the `link.exe` file supplied by Git instead of MSVC*  
> 
> **Fix**  
> `export PATH="/c/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.44.35207/bin/Hostx64/x64:/c/Qt/6.8.3/msvc2022_64/bin:$PATH"`  
> 
> This temporarily changes which tools get accessed first. Puts MSVC and Qt up top.
> 

Now run it  
`kiwix-build kiwix-desktop --config native_static`  

***Wild West from here on.***

## Debugging  

Hitting C1083 error (and U1077: NMAKE: return code 0x2), missing library.h(#)  
Something to do with -I/include  
Header **is** installed  
"The failure is that qmake’s generated Makefile has -I/include instead of the actual install include directory."  
"pkg-config finds it correctly when PKG_CONFIG_PATH is set."  

```bash 
export PKG_CONFIG_PATH="/d/GitHub/kiwix-work/BUILD_native_static/INSTALL/lib/pkgconfig"

# Clean build of kiwix-desktop:
rm -rf BUILD_native_static/kiwix-desktop

set -o pipefail
kiwix-build kiwix-desktop --config native_static 2>&1 | tee build.log
```

Still error'ed  

```bash
cd /d/GitHub/kiwix-work
export PKG_CONFIG_PATH="/d/GitHub/kiwix-work/BUILD_native_static/INSTALL/lib/pkgconfig"

pkg-config --cflags --static libkiwix
```

`-DCURL_STATICLIB -DLZMA_API_STATIC -DCURL_STATICLIB -ID:/GitHub/kiwix-work/BUILD_native_static/INSTALL/include`  

Correct output, "suspicion about static dependency metadata was wrong"  

---

```bash
which pkg-config
```

`/c/ProgramData/chocolatey/bin/pkg-config`

```bash
pkg-config --cflags --static \
   "libkiwix >= 15.0.0" "libkiwix < 16.0.0" \
   "libzim >= 9.0.0" "libzim < 10.0.0"
```

`-DCURL_STATICLIB -DLZMA_API_STATIC -ID:/GitHub/kiwix-work/BUILD_native_static/INSTALL/include`  

---

Need to define where the INSTALL/include directory is:

**EDITING**
`kiwix-work/SOURCE/kiwix-desktop/kiwix-desktop.pro`  

```diff
  CONFIG += link_pkgconfig

+ # Added by Kool, 11/22/2026:
+ # kiwix-build installs dependencies beside the kiwix-desktop build directory.
+ INCLUDEPATH += $$OUT_PWD/../INSTALL/include
+ 
+ # ---
```

"That should keep working if you move the workspace, as long as `kiwix-desktop` and `INSTALL` remain siblings under `BUILD_native_static`."  

```bash
cd /d/GitHub/kiwix-work

# Clean build:
rm -rf BUILD_native_static/kiwix-desktop

export PKG_CONFIG_PATH="/d/GitHub/kiwix-work/BUILD_native_static/INSTALL/lib/pkgconfig"
set -o pipefail
kiwix-build kiwix-desktop --config native_static 2>&1 | tee build.log
```
Passed compiling of `libzim`, failed testing `libzim`.  
"...the test that actually failed is `ClusterTest.read_extended_cluster`."  

```bash
cd /d/GitHub/kiwix-work

grep -R -n -A 30 -B 15 'read_extended_cluster' SOURCE/libzim

grep -n -A 25 -B 10 'ClusterTest.read_extended_cluster' \
  BUILD_native_static/libzim/meson-logs/testlog.txt
```

"...found the explanation in `cluster.cpp`. `read_extended_cluster` deliberately tests a cluster larger than 4 GiB, and the test has an explicit opt-out: it skips itself when `SKIP_BIG_MEMORY_TEST=1`. The failure shows the test’s temporary file is only 122 bytes when it tries to read beyond 4 GiB. The other 35 libzim tests passed."  

This is a test-specific workaround, any issues with libzim related to this will not resolved.  

```bash
export SKIP_BIG_MEMORY_TEST=1
set -o pipefail
kiwix-build kiwix-desktop --config native_static 2>&1 | tee build.log
```

**EDITING**
`kiwix-work/SOURCE/kiwix-desktop/kiwix-desktop.pro`  

```diff
  INCLUDEPATH += $$OUT_PWD/../INSTALL/include
  
  # ---
+ LIBS += $$OUT_PWD/../INSTALL/lib/libkiwix.a

+ # ---
```

---

## Current steps:  

*(literally copy-paste any of the `code` into the currently open cmd/bash)*  

1. Open **x64 Native Tools Command Prompt for VS 2022**  

### Now in the cmd that opened:  

2. Open Git-bash:  
    ```cmd
    "C:\Program Files\Git\git-bash.exe"
    ``` 

3. Close the cmd window  

### Now working in git-bash:  

4. 
    ```bash
    cd /d/GitHub/kiwix-work
    source ../kiwix-desktop/.venv/Scripts/activate
    ```

4. 
    ```bash
    export PATH="/c/Program Files/Microsoft Visual Studio/2022/Community/VC/Tools/MSVC/14.44.35207/bin/Hostx64/x64:/c/Qt/6.8.3/msvc2022_64/bin:$PATH"
    export SKIP_BIG_MEMORY_TEST=1
    ```

4. 
    ```bash
    set -o pipefail
    kiwix-build kiwix-desktop --config native_static 2>&1 | tee build.log
    ```

    Clean build.  
    Missing Qt6 DLL's when attempting to run kiwix-desktop.exe

4. 
    ```bash
    /c/Qt/6.8.3/msvc2022_64/bin/windeployqt.exe "/d/GitHub/kiwix-work/BUILD_native_static/INSTALL/bin/kiwix-desktop.exe"
    ```