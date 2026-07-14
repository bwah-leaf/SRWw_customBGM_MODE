
PyArmor is used during the release build to obfuscate project-owned Python
modules. The generated `pyarmor_runtime` native module is contained in each
executable. PyArmor is not covered by this project's MIT license.

## Python
Anyone rebuilding or redistributing the executables is responsible for using
a PyArmor license that permits the intended build and distribution. Removing
this notice does not remove those obligations.

The single-file executable contains a Python runtime. Python is distributed
under the Python Software Foundation License Version 2 and includes components
under additional compatible licenses.
- License terms: <https://pyarmor.readthedocs.io/en/stable/licenses.html>
- Project: <https://pyarmor.dashingsoft.com/>

- License: https://docs.python.org/3/license.html
- Copyright (c) 2001 Python Software Foundation; All Rights Reserved.
### OpenSSL 3.5.5

## PyInstaller
The Windows executable contains `libcrypto-3-x64.dll` from the Python runtime.
OpenSSL 3.x is licensed under the Apache License 2.0.

PyInstaller is used as a build tool. Its GPL license exception permits generated
executables to be distributed under the application's chosen license, subject
to the licenses of bundled dependencies.
Copyright 1998-2026 The OpenSSL Project Authors. All Rights Reserved.

- License: https://pyinstaller.org/en/stable/license.html
- License: <https://www.openssl.org/source/license.html>
- Apache License 2.0: <https://www.apache.org/licenses/LICENSE-2.0>
- Project: <https://www.openssl.org/>

## PyArmor
### Microsoft Visual C++ Runtime 14.44.35211

The executables contain the unmodified Microsoft runtime files
`VCRUNTIME140.dll` and `VCRUNTIME140_1.dll`. They remain Microsoft software and
are distributed subject to the applicable Microsoft Visual Studio
redistributable terms.

Copyright (c) Microsoft Corporation. All rights reserved.

- Redistribution information: <https://learn.microsoft.com/cpp/windows/redistributing-visual-cpp-files>
- Supported runtime downloads: <https://learn.microsoft.com/cpp/windows/latest-supported-vc-redist>

## External programs required from the user

The following programs are **not** contained in this repository or in either
released executable. The application only starts the separately installed
program selected by the user.

### FFmpeg (external; not bundled)

`SRW_W_BGM_Pack_Maker` invokes a user-supplied `ffmpeg` executable to decode
and convert audio. FFmpeg is generally licensed under LGPL-2.1-or-later, but a
particular build may be GPL-2.0-or-later or include separately licensed codecs
and libraries. The license of the exact build obtained by the user controls.

- Project: <https://ffmpeg.org/>
- License and legal information: <https://ffmpeg.org/legal.html>

PyArmor is used at build time to obfuscate project-owned Python modules. Its
generated runtime is included in the executable. Builders and distributors are
responsible for complying with the PyArmor license applicable to their build
environment and intended use.
Because this project does not redistribute FFmpeg, no FFmpeg binary or FFmpeg
source archive is supplied here.

- License terms: https://pyarmor.readthedocs.io/en/latest/licenses.html
### devkitPro devkitARM (external; not bundled)

## FFmpeg (external, not bundled)
`SRW_W_BGM_ROM_Patcher` invokes the compiler and binary utilities from a
user-installed devkitARM toolchain. No devkitARM, GCC, binutils, libnds, or
Nintendo SDK file is contained in this repository or in the executables.
Individual devkitPro packages have their own licenses; the terms shipped with
the installed packages control.

FFmpeg is invoked as a separate executable selected by the user. No FFmpeg
binary, library, or source is included. Users obtain it separately and are
responsible for that build's LGPL/GPL and codec or patent conditions.
- Project: <https://devkitpro.org/>
- Installation: <https://devkitpro.org/wiki/Getting_Started>
- Package sources: <https://github.com/devkitPro>

- Project: https://ffmpeg.org/
- Legal information: https://ffmpeg.org/legal.html
## Game, platform, and user-provided content

## devkitPro devkitARM (external, not bundled)
Nintendo DS, Super Robot Taisen W, and all related names, trademarks, game
data, artwork, and music belong to their respective owners. They are mentioned
only to describe compatibility.

devkitARM is invoked from the installation directory selected by the user. No
devkitARM, libnds, or proprietary Nintendo SDK file is included.
This distribution contains no ROM, extracted game data, copyrighted music,
official Nintendo SDK component, FFmpeg binary, or devkitARM package. Users
must provide their own lawfully obtained ROM and audio they are authorized to
use. Generated ROMs, BPS patches, and BGM packages may contain or depend on
user-provided copyrighted material; users are responsible for determining
whether they may distribute those outputs.

- Project: https://devkitpro.org/
- Installation: https://devkitpro.org/wiki/Getting_Started
## Rebuild and redistribution note

## Game and platform rights
Before publishing a rebuilt executable, re-check its embedded dependency
versions and update this file. PyInstaller automatically follows the active
Python environment, so rebuilding can change the Python, Tcl/Tk, OpenSSL,
Microsoft runtime, PyInstaller, and PyArmor versions without changing the
application source.

Nintendo DS, Super Robot Taisen W, and related names and assets belong to their
respective owners. This distribution contains no ROM, extracted game data,
music, screenshots, or official SDK. Users must provide their own lawfully
obtained inputs and must not distribute generated content without permission.
This notice is informational and is not legal advice.
