# Super Robot Wars W Custom BGM Tools

[한국어](README.md) | [English](README.en.md)

An unofficial toolset for inserting user-provided music as custom BGM into the Korean Nintendo DS release of **Super Robot Wars W**. Audio packaging and ROM patching are separated into two independent applications, allowing `.bgm.pak` and `.bps` files to be handled as distinct workflow artifacts.

> This project does not include any ROM, game data, music, FFmpeg, devkitARM, or Nintendo SDK component.

## Processing pipeline

```text
MP3 / WAV / FLAC / ...
          │
          ▼
┌──────────────────────────────┐
│ SRW_W_BGM_Pack_Maker.exe     │  + FFmpeg
│ 22.05 kHz / Mono / PCM16     │
└──────────────┬───────────────┘
               │
               ▼
         custom.bgm.pak
               │
               ▼
┌──────────────────────────────┐
│ SRW_W_BGM_ROM_Patcher.exe    │  + devkitARM
│ Verify / build ARM9 / inject │
└──────────────┬───────────────┘
               │
               ├─ Patched .nds
               ├─ Distributable .bps
               └─ build_manifest.json
```

## Applications

| Application | Purpose | Required external tool |
|---|---|---|
| `SRW_W_BGM_Pack_Maker.exe` | Converts audio and packages replacement tracks by BGM ID into a `.bgm.pak` file | [FFmpeg](https://ffmpeg.org/download.html) |
| `SRW_W_BGM_ROM_Patcher.exe` | Combines a `.bgm.pak` with the source NDS ROM and produces `.nds` and `.bps` outputs | [devkitARM](https://devkitpro.org/wiki/Getting_Started) |

Both applications are single-file Windows x64 executables. FFmpeg and devkitARM are not bundled and must be installed separately from their official distribution sources.

## Supported source ROM

The current release supports only a source ROM that exactly matches this SHA-256 digest:

```text
9cbeacfe4fdbeacd6b0ea070b2f2a583c04018f49a47f1e636d02614988c1287
```

The following two filenames contain byte-for-byte identical ROM data and are both supported:

```text
0870 - Super Robot Taisen W (k).nds
0870 - Super Robot Taisen W (kor)_2.0.3.nds
```

If either file is placed next to the ROM patcher, it is selected automatically; `_2.0.3.nds` takes priority when both are present. The filename is not a validation criterion—the actual content is identified by SHA-256.

ROMs from another region or revision, modified dumps, and previously patched ROMs are not supported. Input files are never modified; all results are written separately to the selected output directory.

## 1. Creating a BGM.PAK

### Supported audio formats

The file picker recognizes the following formats:

| Format | Extensions |
|---|---|
| MP3 | `.mp3` |
| WAV | `.wav` |
| FLAC | `.flac` |
| Ogg Vorbis | `.ogg` |
| MPEG-4 Audio | `.m4a` |
| AAC | `.aac` |
| Windows Media Audio | `.wma` |
| Opus | `.opus` |
| AIFF | `.aiff`, `.aif` |

FFmpeg performs the actual decoding. A format not listed above may still be selected through `All files (*.*)` if the installed FFmpeg build can decode it. DRM-protected media is not supported.

Every source track is normalized to the following internal format:

```text
Sample rate : 22,050 Hz
Channels    : Mono
Sample type : Signed PCM 16-bit little-endian
Container   : NWAV (tool-specific stream container)
```

### Maximum combined duration

The conservative combined limit, calculated from the verified source ROM size, the 512 MiB NDS ROM limit, stream alignment overhead, and the `22.05 kHz / Mono / PCM16` format, is:

```text
02:58:09
```

The application probes every selected track and displays the combined duration on a progress bar. The `Create BGM.PAK` button is automatically disabled when the total exceeds the limit. The backend also rejects a package if the converted stream layout would exceed the ROM capacity.

### Procedure

1. Run `SRW_W_BGM_Pack_Maker.exe`.
2. Click `Browse...` and select the installed `ffmpeg.exe`.
3. Select the output `.bgm.pak` path.
4. Select the BGM ID to replace.
5. Click `Assign audio from selected ID` and choose the audio files.
6. Repeat steps 4–5 until all desired IDs have been assigned.
7. Confirm that the combined duration is within the limit, then click `Create BGM.PAK`.

The package stores the BGM IDs, converted stream data, sizes, and integrity metadata. Original source paths and filenames are not stored.

## 2. Creating the NDS ROM patch

`SRW_W_BGM_ROM_Patcher.exe` verifies the package, builds the ARM9 streaming payload, and injects it together with the converted audio into the source ROM.

### Procedure

1. Install devkitARM through [devkitPro](https://devkitpro.org/wiki/Getting_Started).
2. Run `SRW_W_BGM_ROM_Patcher.exe`.
3. Select the source `.nds` matching the supported SHA-256 digest.
4. Select the `.bgm.pak` created by `SRW_W_BGM_Pack_Maker`.
5. Select the output directory and devkitARM installation directory.
6. Click `Create ROM patch`.

### Output files

| File | Description |
|---|---|
| `... [Custom STRM BGM].nds` | Ready-to-run patched ROM |
| `... [Custom STRM BGM].bps` | BPS patch for the exact source ROM; includes the custom BGM difference data |
| `build_manifest.json` | Source/output hashes, sizes, hook addresses, and inserted stream metadata |

Only the patched `.nds` is needed for normal play. Given the `.bps`, an exact matching source ROM, and a BPS patching utility, the same patched ROM can be reconstructed without the `.bgm.pak`, FFmpeg, or devkitARM.

> The BPS patch contains difference data derived from the converted music. Do not redistribute a BPS containing music you are not authorized to distribute.

## Emulator compatibility

Boot, BGM playback, and track transitions were tested in the following environments:

| Environment | Status |
|---|---|
| DeSmuME | Tested |
| no$gba | Tested |
| melonDS for Windows | Tested |
| melonDS for Android | Tested |
| Original Nintendo DS hardware | Not tested |

Playback behavior may vary depending on emulator version, audio synchronization, buffering, and frame-skip settings.

## Known issues

- A short noise or buffered audio tail may occur when a track stops or switches in certain scenes.
- Original Nintendo DS hardware has not been tested.
- Other issues may occur with different emulator versions, audio backends, frame-skip settings, or synchronization options.
- When reporting a problem, include the emulator and version, BGM ID, scene, and exact reproduction steps.

## Release integrity

SHA-256 digests for the current v2.1.1 build:

```text
SRW_W_BGM_Pack_Maker.exe
fb674762ff01d9914bc206e1b16fc6f834254d2a7ae3febd43fd59b58d693994

SRW_W_BGM_ROM_Patcher.exe
acd529faccb4397df07048f1c3db01c2f8fc730fa162eb6da6540d9eca673a98
```

Rebuilding the executables changes their hashes. Prefer the checksums published with the corresponding GitHub Release.

## Legal notice

This is an unofficial fan-made interoperability tool. It is not affiliated with or endorsed by Nintendo, Bandai Namco Entertainment, or Banpresto. All game names, trademarks, ROM data, music, and other third-party materials belong to their respective owners.

Users must provide their own lawfully obtained source ROM and audio they are authorized to use. Neither this repository nor its Releases include any ROM, extracted game data, music, FFmpeg, devkitARM, or Nintendo SDK component.

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for third-party component and external-tool notices.

## Reference

- [Custom BGM mode description and usage example](https://blog.naver.com/casu_incosmo/224346862126)
- Do not attach ROM images or copyrighted audio when reporting an issue.
