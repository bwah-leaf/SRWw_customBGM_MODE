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
│ 22.05 kHz / Mono·Stereo PCM16│
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
translated kor patched ver.2.0.3

https://blog.naver.com/dudaos1123/222622542237 <-- here

```text
9cbeacfe4fdbeacd6b0ea070b2f2a583c04018f49a47f1e636d02614988c1287
```

The following two filenames contain byte-for-byte identical ROM data and are both supported:

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

Every source track is normalized to the following internal format. Channel mode can be selected independently for each track.

```text
Sample rate : 22,050 Hz
Channels    : Mono or Stereo (per-track selection)
Sample type : Signed PCM 16-bit little-endian
Container   : NWAV (tool-specific stream container)
```

### Maximum combined duration

The conservative **mono-equivalent** limit, calculated from the verified source ROM size, the 512 MiB NDS ROM limit, stream alignment overhead, and the `22.05 kHz / Mono / PCM16` format, is:

```text
02:58:09
```

The application displays both the actual combined track duration and the channel-weighted capacity usage. One second of stereo consumes the same space as two seconds of mono. The `Create BGM.PAK` button is automatically disabled when the weighted total exceeds the limit. The backend also rejects a package if the converted stream layout would exceed the ROM capacity.

### Procedure

1. Run `SRW_W_BGM_Pack_Maker.exe`.
2. Click `Browse...` and select the installed `ffmpeg.exe`.
3. Select the output `.bgm.pak` path.
4. Select the BGM ID to replace.
5. Choose Mono or Stereo under `Conversion`, click `Assign audio from selected ID`, and choose the audio files. Multiple files are placed using natural numeric filename order (`2` before `10`), and a `BGM ID <- filename` preview is shown before assignment.
6. To change already assigned tracks, select their rows and click `Apply to selected tracks`. Multiple rows may be changed at once.
7. Repeat steps 4–6 until all desired IDs have been assigned.
8. Confirm that the channel-weighted usage is within the limit, then click `Create BGM.PAK`.

The list is updated only after every selected file passes duration probing. A bad or unsupported file therefore cannot leave a partially assigned ID range.

The package stores BGM IDs, converted stream data, sizes, and integrity metadata; private source paths are not stored in it. The adjacent `.bgm.pak.manifest.json` records only IDs, game titles, and source filenames so the mapping can be reviewed.

## 2. Creating the NDS ROM patch

`SRW_W_BGM_ROM_Patcher.exe` verifies the package, builds the ARM9 streaming payload, and injects it together with the converted audio into the source ROM.

### Procedure

1. Install devkitARM through [devkitPro](https://devkitpro.org/wiki/Getting_Started).
2. Run `SRW_W_BGM_ROM_Patcher.exe`.
3. Select the source `.nds` matching the supported SHA-256 digest.
4. Select the `.bgm.pak` created by `SRW_W_BGM_Pack_Maker`. The most recently created package is selected automatically even when the two executables are kept in different folders.
5. Select the output directory and devkitARM installation directory.
6. Click `Create ROM patch`.

Before patching, the verified track count and included IDs are displayed. Use `View PAK mapping` to review the filename assigned to every ID.

### Output files

| File | Description |
|---|---|
| `... [Custom STRM BGM].nds` | Ready-to-run patched ROM |
| `... [Custom STRM BGM].bps` | BPS patch for the exact source ROM; includes the custom BGM difference data |
| `build_manifest.json` | Source/output hashes, sizes, hook addresses, and inserted stream metadata |
| `*.bgm.pak.manifest.json` | Optional review document containing the package ID/title/filename mapping; not required to patch the ROM |

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

SHA-256 digests for the current v2.3.0 build:

```text
SRW_W_BGM_Pack_Maker.exe
99dcc00739bfd6ac434305b55e086e050a77e16c452a5a13b34d1cc683bbf027

SRW_W_BGM_ROM_Patcher.exe
3c5b5134c2b743b0541f1937982af513fb78cc14f05176a0140e8d33fe615eb9
```

Rebuilding the executables changes their hashes. Prefer the checksums published with the corresponding GitHub Release.

## Legal notice

This is an unofficial fan-made interoperability tool. It is not affiliated with or endorsed by Nintendo, Bandai Namco Entertainment, or Banpresto. All game names, trademarks, ROM data, music, and other third-party materials belong to their respective owners.

Users must provide their own lawfully obtained source ROM and audio they are authorized to use. Neither this repository nor its Releases include any ROM, extracted game data, music, FFmpeg, devkitARM, or Nintendo SDK component.

See [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) for third-party component and external-tool notices.

## Reference

- [Custom BGM mode description and usage example](https://blog.naver.com/casu_incosmo/224346862126)
- Do not attach ROM images or copyrighted audio when reporting an issue.
