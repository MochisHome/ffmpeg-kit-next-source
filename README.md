# ffmpeg-kit-next-source

This repository provides the **Corresponding Source** for the FFmpeg / FFmpegKitNext libraries distributed in the Doobi app, as required by the GNU Lesser General Public License (LGPL).

It contains the build instructions, the exact versions and configuration used, the license texts, and, as GitHub Release assets, complete source archives of every LGPL component shipped in the apps. Release assets for every shipped version are kept available.

## Components

| Component | Version | Source revision | License | Platforms |
| --- | --- | --- | --- | --- |
| [FFmpegKitNext (ffmpeg-kit-next)](https://github.com/arthenica/ffmpeg-kit-next) | 9.0.0 | commit [`5e51b2d`](https://github.com/arthenica/ffmpeg-kit-next/commit/5e51b2da4c3593c0f2f9b49f53eeb497d93e39d3) (`5e51b2da4c3593c0f2f9b49f53eeb497d93e39d3`) | LGPL-3.0-or-later | iOS, Android |
| [FFmpeg](https://github.com/arthenica/FFmpeg) | n9.0.1 | tag `n9.0.1`, commit `bf1b838f2ab88b4f8fd83443325c782ea0e0f7fa` | LGPL-3.0-or-later (configured with `--enable-version3`, without `--enable-gpl` / `--enable-nonfree`) | iOS, Android |
| [cpu-features](https://github.com/arthenica/cpu_features) | v0.11.0 | commit `81d13c49649f0714dd41fb56bb246398b6584085` | Apache-2.0 | Android only |
| [smart-exception](https://github.com/tanersener/smart-exception) (`com.arthenica:smart-exception-java`, `com.arthenica:smart-exception-common`) | 0.2.1 | Maven Central artifacts, tag `v0.2.1` | BSD-3-Clause | Android only |

The FFmpeg tag `n9.0.1` in [arthenica/FFmpeg](https://github.com/arthenica/FFmpeg) is the same Git object as the official FFmpeg tag `n9.0.1` (commit `bf1b838f2ab88b4f8fd83443325c782ea0e0f7fa`).

The FFmpegKitNext source files state "either version 3 of the License, or (at your option) any later version", so FFmpegKitNext is licensed under LGPL-3.0-or-later.

License texts are in [`licenses/`](licenses/). Machine-readable build information is in [`build/9.0.0/build-info.json`](build/9.0.0/build-info.json).

## Modifications

The app publisher has made **no manual changes** to the source code of FFmpeg, FFmpegKitNext or cpu-features. The binaries are built from the upstream revisions listed above with the upstream build scripts. Two kinds of changes do exist, and both are described here:

### 1. Build-time changes to the FFmpeg source (made by the FFmpegKitNext build scripts)

The FFmpegKitNext build scripts (`scripts/apple/ffmpeg.sh` and `scripts/android/ffmpeg.sh`) automatically modify the FFmpeg source tree when they run. These changes add FFmpegKit's own I/O protocol code to FFmpeg and adjust logging. Files modified (modification date: 2026-09-23, the build date):

| File | Platforms |
| --- | --- |
| `libavformat/file.c` | iOS, Android |
| `libavformat/hls.c` | iOS, Android |
| `libavformat/protocols.c` | iOS, Android |
| `libavutil/file.c` | iOS, Android |
| `libavutil/file.h` | iOS, Android |
| `libavutil/log.c` | iOS, Android |
| `configure` | iOS only (removes `-mdynamic-no-pic` from the assembler flags) |
| `ffbuild/common.mak` | iOS only |

The code that `libavformat/file.c`, `libavutil/file.c` and `libavutil/file.h` receive comes from `tools/protocols/` in the FFmpegKitNext source; the Android build additionally adds the Android-only `ffkitsaf` protocol.

The exact changes are documented as patches against the FFmpeg tag `n9.0.1`, one per platform:

- [`patches/ffmpeg-n9.0.1-ios-build-changes.patch`](patches/ffmpeg-n9.0.1-ios-build-changes.patch)
- [`patches/ffmpeg-n9.0.1-android-build-changes.patch`](patches/ffmpeg-n9.0.1-android-build-changes.patch)

They were generated with `git diff --binary n9.0.1` in the FFmpeg source tree after each build. The `ffbuild/common.mak` change in the iOS patch contains the absolute path of the build directory used for the distributed binaries (`/tmp/ffkit-ios`); a rebuild writes its own build directory there instead, so the framework path in that line differs between builds (for example, the directory name includes the deployment target).

**The `ffmpeg-n9.0.1-source.tar.gz` archive is the unmodified FFmpeg n9.0.1 source.** The FFmpegKitNext build scripts apply the changes above automatically at build time, so you do not need to apply the patches yourself when rebuilding; the patches only document what the build changes.

### 2. Integration changes to the FFmpegKitNext React Native module (made by the app publisher)

To integrate FFmpegKitNext into the React Native app, the app publisher modified two files of the FFmpegKitNext `react-native/` directory:

- `react-native/package.json`: trimmed for use as a private local package (development scripts, dev dependencies and release tooling removed; `"private": true` added; license fields updated).
- `react-native/android/build.gradle`: `compileSdkVersion` / `minSdkVersion` are read from the root project (`safeExtGet`) instead of being hard-coded, and the local `libs-maven` Maven repository is declared unconditionally instead of only in one branch.

The modified files are in [`modifications/react-native/`](modifications/react-native/), and a unified diff against the upstream commit `5e51b2d` is in [`modifications/react-native.diff`](modifications/react-native.diff) (apply with `git apply` from the root of the FFmpegKitNext checkout). The modified `package.json` is kept exactly as used in the app: the licenseNote refers to the integrating app's internal build notes; the build instructions for this repository are in this README.

## How the libraries are linked

- **FFmpeg libraries and the FFmpegKit core library** (`libavcodec`, `libavdevice`, `libavfilter`, `libavformat`, `libavutil`, `libswresample`, `libswscale`, `ffmpegkit`) are distributed as dynamic libraries:
  - **iOS**: 8 dynamic frameworks shipped as xcframeworks, embedded in the app bundle's `Frameworks/` directory.
  - **Android**: shared libraries (`.so`) inside the `com.arthenica:ffmpeg-kit-next:9.0.0` AAR, packaged under the app's `lib/arm64-v8a/`. On Android, cpu-features is statically linked into `libffmpegkit_abidetect.so`.
- **The FFmpegKitNext React Native bridge is not dynamically linked**: on iOS its native part is statically linked into the app executable, and its JavaScript part is bundled into the app's JavaScript bundle.
- **The FFmpegKitNext Android Java API layer is not dynamically linked**: it is compiled into the app's DEX files together with the rest of the app. smart-exception is compiled into the app's DEX files as well.

Replacing the libraries:

- **Android**: a user can unpack the APK, replace the shared libraries with their own builds (same names and compatible ABI), and re-sign the APK.
- **iOS**: the code signing and encryption applied by the App Store prevent users from replacing libraries inside an installed App Store build.

## Enabled and disabled features

- **Not enabled**: `--enable-gpl`, `--enable-nonfree`, x264, x265, fdk-aac, or any other GPL or non-free external library. No external third-party codec libraries are linked.
- **iOS**: AudioToolbox, AVFoundation, VideoToolbox, zlib, libiconv (all system frameworks / libraries).
- **Android**: MediaCodec, zlib (system libraries); cpu-features.
- H.264 / HEVC encoding is provided only by the platform hardware encoders (VideoToolbox on iOS, MediaCodec on Android), not by x264/x265.

## Build instructions

Requirements: [Nix](https://nixos.org/) with flakes enabled, and Git. The devShells defined in the upstream `flake.nix` (`xcode26`, `android-r27d`) provide the toolchains other than Xcode (including the Android SDK/NDK r27). **Xcode is provided by the host machine**: the iOS build requires macOS with Xcode installed, and the distributed iOS binaries were built with Xcode 27.0 / iOS SDK 27.0.

These steps rebuild the libraries from the three source archives of the release. No FFmpeg or cpu-features source is downloaded: the build scripts use `src/ffmpeg` and `src/cpu-features` as they are when these directories already exist. The upstream build scripts still download a few build tools that are not part of the distributed libraries: [gnu-config](https://github.com/arthenica/gnu-config) (tag `v20210814`, both platforms), [gas-preprocessor](https://github.com/arthenica/gas-preprocessor) (`v20210917`, iOS only), googletest (cloned by the cpu-features CMake configuration, Android only) and the Gradle dependencies of the Android library.

**1. Prepare the source tree.** Put `ffmpeg-kit-next-5e51b2d-source.tar.gz`, `ffmpeg-n9.0.1-source.tar.gz` and `cpu-features-source.tar.gz` in one directory and run from that directory:

```bash
tar xzf ffmpeg-kit-next-5e51b2d-source.tar.gz
cd ffmpeg-kit-next-5e51b2d
git init -q && git add -A -f && git -c user.name=build -c user.email=build@localhost commit -qm "ffmpeg-kit-next 5e51b2d"
mkdir -p src
tar xzf ../ffmpeg-n9.0.1-source.tar.gz -C src && mv src/ffmpeg-n9.0.1 src/ffmpeg
(cd src/ffmpeg && git init -q && git add -A -f && git -c user.name=build -c user.email=build@localhost commit -qm "FFmpeg n9.0.1" && git tag n9.0.1)
tar xzf ../cpu-features-source.tar.gz -C src && mv src/cpu-features-v0.11.0 src/cpu-features
```

The Git repositories are required by the upstream build scripts:

- `ios.sh` / `android.sh` run `git describe --tags --always` in the FFmpegKitNext directory and stop if it fails; Nix also evaluates `flake.nix` from the Git-tracked files.
- The FFmpeg build scripts run `git checkout` in `src/ffmpeg` to restore the original files before they apply their build-time changes (see [Modifications](#modifications)). The tag `n9.0.1` gives FFmpeg its version string (`ffbuild/version.sh`).

cpu-features is used by the Android build only.

**2. Build.** Run from the `ffmpeg-kit-next-5e51b2d` directory.

iOS:

```bash
./nix-ios.sh -p xcode26 -x --target=16.4 --arch=arm64,arm64-simulator --enable-lib-ios-audiotoolbox --enable-lib-ios-avfoundation --enable-lib-ios-videotoolbox --enable-lib-ios-zlib --enable-lib-ios-libiconv
```

Output: `prebuilt/bundle-apple-xcframework-ios-16.4/` (8 dynamic xcframeworks, slices `ios-arm64` and `ios-arm64-simulator`, minimum iOS 16.4).

`--target=16.4` is required: with a lower deployment target, recent Xcode SDKs link CoreMedia symbols through `@rpath/libswiftCoreMedia.dylib`, which only works on iOS 27 and later.

Android:

```bash
./nix-android.sh -p android-r27d --arch=arm64-v8a --enable-lib-android-media-codec --enable-lib-android-zlib
```

Output: `prebuilt/bundle-android-aar-24-maven/` (Maven repository with `com.arthenica:ffmpeg-kit-next:9.0.0`, native libraries for `arm64-v8a` only).

The two platforms share `src/ffmpeg`. Run them one after the other in the same directory, or prepare a separate source tree (step 1) for each platform to build them in parallel.

## Getting the source

- **Release assets** of this repository ([v9.0.0](https://github.com/MochisHome/ffmpeg-kit-next-source/releases/tag/v9.0.0)):
  - `ffmpeg-kit-next-5e51b2d-source.tar.gz` — FFmpegKitNext at commit `5e51b2d` (`git archive` of the upstream commit)
  - `ffmpeg-n9.0.1-source.tar.gz` — FFmpeg n9.0.1, unmodified (`git archive` of the tag `n9.0.1`)
  - `cpu-features-source.tar.gz` — cpu-features v0.11.0 (`git archive` of the tag `v0.11.0`; Android only)
  - `ffmpeg-n9.0.1-ios-build-changes.patch`, `ffmpeg-n9.0.1-android-build-changes.patch` — the build-time changes to FFmpeg (same files as in [`patches/`](patches/); for reference only, see [Modifications](#modifications))
  - `SHA256SUMS` — checksums of the files above
- **Upstream**: <https://github.com/arthenica/ffmpeg-kit-next>, <https://github.com/arthenica/FFmpeg>, <https://github.com/arthenica/cpu_features>, and the official FFmpeg project at <https://ffmpeg.org/>.

SHA-256 of the source archives:

| File | SHA-256 |
| --- | --- |
| `ffmpeg-kit-next-5e51b2d-source.tar.gz` | `7d2db69d1d7617ede2a58ceadea078793d09060485ea116ee2b9b0a955724b65` |
| `ffmpeg-n9.0.1-source.tar.gz` | `eef4b0716badad6c3234c7b8347b62826df07839cb195f570d12f2e1aea264fb` |
| `cpu-features-source.tar.gz` | `66104a946844f8529b68c858ad0894f2bbf710f104e3239a96565af4f78bb89d` |

The FFmpeg source archive also contains FFmpeg's own optional wrapper code for external libraries (for example `libavcodec/libx264.c`). These files are part of the FFmpeg source distribution; they are not compiled into the distributed binaries because the corresponding libraries were not enabled. The FFmpegKit source archive likewise contains upstream build scripts for optional libraries (for example `scripts/*/x264.sh`) that were not used.

## Distributed binaries

SHA-256 of the publisher's internal binary archives (for traceability only; these archives are not published). Rebuilding from source will not produce byte-identical binaries because build paths, toolchain versions and archive metadata differ.

| Platform | Artifact | SHA-256 |
| --- | --- | --- |
| iOS | FFmpegKit 9.0.0 xcframework bundle (arm64, arm64-simulator) | `75f9b418531125b082254f7d3c2a8c7c802519d08d3f21279a071a781db241cf` |
| Android | FFmpegKit 9.0.0 AAR Maven bundle (arm64-v8a) | `2e169b107e27946f4082936130a27eeb9b38dd0c8182cebd3f6f3676622fbd41` |

Build date: 2026-09-23.

## Notices

- This software is based in part on the work of the Independent JPEG Group.
- FFmpeg is a trademark of Fabrice Bellard. This repository is not affiliated with or endorsed by the FFmpeg project or Arthenica.

## Contact

For questions or requests regarding this source code, please open an issue in this repository.

## License

Documentation in this repository: CC0-1.0. Components: see [`licenses/`](licenses/).

`licenses/` contains:

- `FFmpeg-LICENSE.md`, `FFmpeg-COPYING.LGPLv2.1`, `FFmpeg-COPYING.LGPLv3`: FFmpeg's license files. `FFmpeg-LICENSE.md` is FFmpeg's general explanation of its licensing options; because this build is configured with `--enable-version3` (and without `--enable-gpl` / `--enable-nonfree`), FFmpeg as built here is licensed under LGPL-3.0-or-later.
- `FFmpegKit-LGPL-3.0.txt`: FFmpegKitNext license text (LGPL-3.0-or-later).
- `GPL-3.0.txt`: the GNU General Public License v3.0, which the LGPL v3.0 incorporates and supplements.
- `cpu-features-Apache-2.0.txt`: cpu-features (Android only).
- `smart-exception-BSD-3-Clause.txt`: smart-exception (Android only).
