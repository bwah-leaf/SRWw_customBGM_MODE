# 슈퍼로봇대전 W 커스텀 BGM 도구

[한국어](README.md) | [English](README.en.md)

Nintendo DS용 **슈퍼로봇대전 W 한국어판**에 사용자가 준비한 음원을 커스텀 BGM으로 삽입하기 위한 비공식 도구입니다. 음원 변환과 ROM 패치를 서로 독립된 두 프로그램으로 분리하여, 음원 제작자와 패치 사용자가 `.bgm.pak` 또는 `.bps` 단위로 작업할 수 있습니다.

> 이 프로젝트는 ROM, 게임 데이터, 음악, FFmpeg, devkitARM 또는 Nintendo SDK를 포함하지 않습니다.

## 처리 구조

```text
MP3 / WAV / FLAC / ...
          │
          ▼
┌──────────────────────────────┐
│ SRW_W_BGM_Pack_Maker.exe     │  + FFmpeg
│ 22.05 kHz / Mono / PCM16 변환 │
└──────────────┬───────────────┘
               │
               ▼
         custom.bgm.pak
               │
               ▼
┌──────────────────────────────┐
│ SRW_W_BGM_ROM_Patcher.exe    │  + devkitARM
│ BGM.PAK 검증 / ARM9 빌드 / 삽입 │
└──────────────┬───────────────┘
               │
               ├─ 패치된 .nds
               ├─ 배포용 .bps
               └─ build_manifest.json
```

## 프로그램 구성

| 프로그램 | 역할 | 외부 필수 도구 |
|---|---|---|
| `SRW_W_BGM_Pack_Maker.exe` | 음원을 변환하고 BGM ID별로 묶어 `.bgm.pak` 생성 | [FFmpeg](https://ffmpeg.org/download.html) |
| `SRW_W_BGM_ROM_Patcher.exe` | `.bgm.pak`과 원본 NDS ROM을 결합하여 `.nds`와 `.bps` 생성 | [devkitARM](https://devkitpro.org/wiki/Getting_Started) |

두 프로그램은 Windows x64용 단일 실행 파일입니다. FFmpeg와 devkitARM은 EXE에 포함되지 않으므로 공식 배포처에서 별도로 설치해야 합니다.

## 지원 원본 ROM

현재 버전은 아래 SHA-256과 정확히 일치하는 원본만 지원합니다.

```text
9cbeacfe4fdbeacd6b0ea070b2f2a583c04018f49a47f1e636d02614988c1287
```

다음 두 파일명은 실제 내용이 바이트 단위로 동일하며 모두 지원됩니다.

```text
0870 - Super Robot Taisen W (k).nds
0870 - Super Robot Taisen W (kor)_2.0.3.nds
```

ROM 패처와 같은 폴더에 둘 중 하나가 있으면 자동으로 선택하며, 두 파일이 모두 있으면 `_2.0.3.nds`를 우선합니다. 파일명 자체는 검증 기준이 아니며 SHA-256으로 실제 내용을 판정합니다.

지역, 리비전, 덤프 상태 또는 기존 패치 적용 여부가 다른 ROM은 사용할 수 없습니다. 입력 파일은 직접 변경하지 않으며 결과물은 지정한 출력 폴더에 별도로 생성됩니다.

## 1. BGM.PAK 만들기

### 지원 음원

파일 선택 창에서 다음 형식을 지원합니다.

| 형식 | 확장자 |
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

실제 디코딩은 FFmpeg가 담당합니다. 목록에 없는 확장자도 FFmpeg가 디코딩할 수 있다면 `모든 파일 (*.*)`로 선택할 수 있습니다. DRM이 적용된 음원은 지원하지 않습니다.

모든 입력 음원은 다음 내부 형식으로 정규화됩니다.

```text
Sample rate : 22,050 Hz
Channels    : Mono
Sample type : Signed PCM 16-bit little-endian
Container   : NWAV (tool-specific stream container)
```

### 최대 재생시간

검증된 원본 ROM의 크기, NDS 최대 ROM 용량 512 MiB, 스트림 정렬 오버헤드와 `22.05 kHz / Mono / PCM16` 규격을 보수적으로 계산한 전체 한도는 다음과 같습니다.

```text
02:58:09
```

프로그램은 선택한 모든 음원의 재생시간을 합산하여 진행 막대로 표시합니다. 합계가 최대 시간을 초과하면 `BGM.PAK 만들기` 버튼이 자동으로 비활성화됩니다. 실제 변환 결과가 용량을 초과하는 경우에도 백엔드 검증 단계에서 생성을 중단합니다.

### 사용 순서

1. `SRW_W_BGM_Pack_Maker.exe`를 실행합니다.
2. `찾기...`를 눌러 설치한 `ffmpeg.exe`를 지정합니다.
3. 출력할 `.bgm.pak` 경로를 지정합니다.
4. 목록에서 교체할 BGM ID를 선택합니다.
5. `선택 ID부터 음원 지정`을 눌러 음원을 배정합니다.
6. 필요한 ID가 채워질 때까지 4~5번을 반복합니다.
7. 재생시간 합계가 한도 이내인지 확인하고 `BGM.PAK 만들기`를 누릅니다.

`.bgm.pak`에는 BGM ID, 변환 데이터, 크기 및 무결성 정보가 저장됩니다. 원본 음원의 경로와 파일명은 저장하지 않습니다.

## 2. NDS ROM 패치 만들기

`SRW_W_BGM_ROM_Patcher.exe`는 앞 단계에서 생성한 `.bgm.pak`을 검증한 뒤, 스트리밍 재생용 ARM9 페이로드를 컴파일하여 원본 ROM에 결합합니다.

### 사용 순서

1. [devkitPro](https://devkitpro.org/wiki/Getting_Started)를 통해 devkitARM을 설치합니다.
2. `SRW_W_BGM_ROM_Patcher.exe`를 실행합니다.
3. SHA-256이 일치하는 원본 `.nds`를 지정합니다.
4. `SRW_W_BGM_Pack_Maker`에서 만든 `.bgm.pak`을 지정합니다.
5. 출력 폴더와 devkitARM 설치 폴더를 지정합니다.
6. `ROM 패치 만들기`를 누릅니다.

### 출력 파일

| 파일 | 설명 |
|---|---|
| `... [Custom STRM BGM].nds` | 즉시 실행할 수 있는 패치 완료 ROM |
| `... [Custom STRM BGM].bps` | 원본 ROM에 적용할 수 있는 BPS 패치. 커스텀 BGM 데이터 포함 |
| `build_manifest.json` | 원본·결과물 해시, 크기, 훅 주소 및 삽입 음원 정보 |

실제 플레이에는 패치된 `.nds`만 있으면 됩니다. `.bps`가 있다면 정확히 일치하는 원본 ROM과 BPS 적용 프로그램을 사용하여 동일한 패치 ROM을 다시 만들 수 있으며, 이때 `.bgm.pak`, FFmpeg와 devkitARM은 필요하지 않습니다.

> `.bps`에는 변환된 음원의 차이 데이터가 포함됩니다. 배포 권한이 없는 음악을 넣은 BPS를 공유하지 마십시오.

## 에뮬레이터 호환성

다음 환경에서 부팅, BGM 재생 및 전환을 확인했습니다.

| 환경 | 테스트 상태 |
|---|---|
| DeSmuME | 확인 |
| no$gba | 확인 |
| melonDS (Windows) | 확인 |
| melonDS (Android) | 확인 |
| Nintendo DS 실기 | 미확인 |

에뮬레이터의 오디오 동기화와 버퍼 설정에 따라 재생 결과가 달라질 수 있습니다.

## 알려진 문제

- 일부 구간에서 곡이 정지하거나 다른 곡으로 전환될 때 짧은 노이즈 또는 소리가 밀려 들어가는 현상이 남아 있습니다.
- 실기 테스트는 수행하지 못했습니다.
- 에뮬레이터 버전, 오디오 백엔드, 프레임 스킵과 동기화 설정에 따라 별도의 문제가 발생할 수 있습니다.
- 문제가 발생하면 사용한 에뮬레이터와 버전, BGM ID, 발생 장면 및 재현 절차를 함께 기록해 주세요.

## 배포 파일 무결성

현재 v2.1.1 빌드의 SHA-256입니다.

```text
SRW_W_BGM_Pack_Maker.exe
fee62a224528c76af4fa849fdc2e3f636b5664e781d7e92b8f0ee6013690508d

SRW_W_BGM_ROM_Patcher.exe
52984ce17e85fe0333359bb9bf7a8a6d20d28e6d29bececa660edfd39f8f6991
```

파일을 다시 빌드하면 해시가 변경됩니다. GitHub Release에 표시된 버전별 체크섬을 우선 확인하세요.

## 법적 고지

이 도구는 비공식 팬 제작 상호운용성 도구이며 Nintendo, Bandai Namco Entertainment 또는 Banpresto와 제휴하거나 보증받지 않았습니다. 게임명, 상표, ROM, 음악 및 기타 제3자 자료의 권리는 각 권리자에게 있습니다.

사용자는 합법적으로 취득한 원본 ROM과 사용 권한이 있는 음원을 직접 준비해야 합니다. 이 저장소와 Release에는 ROM, 게임 추출 데이터, 음악, FFmpeg, devkitARM 또는 Nintendo SDK가 포함되지 않습니다.

제3자 구성요소와 외부 도구에 관한 내용은 [THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md)를 참조하세요.

## 참고

- [커스텀 BGM 모드 설명 및 사용 예시](https://blog.naver.com/casu_incosmo/224346862126)
- 오류 제보 시 원본 ROM이나 저작권 음원을 첨부하지 마세요.
