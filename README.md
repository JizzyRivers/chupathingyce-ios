# warthog-ios

An iPhone, iPad and Apple TV port of [ChupathingyCE](https://github.com/ChupathingyCE/chupathingyce),
the Halo: Combat Evolved decompilation port. As with the console ports
(warthog-xbox, warthog-360, warthog-switch, warthog-wiiu), the game lives in its own
repository and this one holds only the port.

You bring your own copy of the game: the app copies the maps from your Xbox disc
image and never downloads Microsoft's files.

## Status

| Phase | What | Where it runs | State |
| --- | --- | --- | --- |
| 1 | The launcher: disc import, map import, then settings, servers | Expo Go on iPhone and iPad | Map zip import works on an iPhone 14 Plus (iOS 27.0) |
| 2 | The game: the engine as a native module, in a development build | iPhone and iPad, sideloaded | The CI build and the memory check, before the engine |
| 3 | A release IPA, and tvOS | Sideloaded (AltStore, SideStore) | Not started |

## Phase 1: the launcher in Expo Go

`app/` is an Expo (SDK 58) app in TypeScript, with Expo Router. Expo Go runs
JavaScript only, so the game itself (C, OpenGL ES, SDL3) cannot run there; the
launcher around it can, and is written so that the development build keeps it.

| Android launcher (`port/android/app/.../decomp/`) | Here |
| --- | --- |
| `XisoExtractor.java` (from `port/linux/src/xiso.c`) | `app/src/lib/xiso.ts`, tested in `app/test/xiso.test.ts` |
| `LauncherActivity.java` | `app/src/app/index.tsx`, `app/src/lib/importDisc.ts` |
| the data folder (`getExternalFilesDir`) | the app's Documents folder, the same layout (`app/src/lib/storage.ts`) |
| `MapImporter.java` | `app/src/lib/mapImport.ts` and `app/src/lib/zip.ts`, tested in `app/test/mapImport.test.ts`. It also takes Xbox maps (into `maps/`), so a zip of your own disc's maps folder can stand in for the disc image |
| (none) | `app/src/lib/download.ts`: downloads a `.zip` of maps, or a `.map`, from an address the player types in, as an iOS background download, then imports it as a picked file. The app has no address of its own built in |
| `config.toml` settings, touch layout | to do |

### Run it

You need Node.js LTS on the computer, and Expo Go from the App Store on the
iPhone or iPad, on the same network.

```sh
cd app
npm install
npx expo start        # scan the QR code with the Camera app
npm test              # the disc extractor's tests (Node, no device)
npm run typecheck
```

### On the device, to check

- **Reading the image where it is.** The picker is asked not to copy the image
  (`copyToCacheDirectory: false`), as a disc image is several GB. If iOS does
  not let the app read it there, the app asks to pick it again and copy it
  first, which needs as much free room again. Which of the two happens for a
  file in iCloud Drive, On My iPhone and an external drive is still to try.
- **Offsets past 4 GB.** A whole disc's `.iso` is about 7 GB.
  `FileHandle.offset` is a JS number, so it should seek there, but no one has
  tried it yet.
- **The background.** iOS pauses the app in the background, so the import
  stops while it is there. The screen is kept awake during the import.
- **Speed.** Expo Go copies in 1 MB chunks through JavaScript. The
  development build can do the same work natively if this is too slow.

## Phase 2: the game

The Android port's native 64-bit build (`ninja android64`,
`port/android/native64/android64_main.c`) is the nearest to iOS. It is 64-bit
code that calls SDL3 and OpenGL ES 3 directly, with no guest image, and
`host/host_touch.c` has the touch controls. iOS keeps OpenGL ES 3 (deprecated,
not removed), and SDL3 and FFmpeg build for iOS and tvOS.

**Memory, with a free Apple account: still to measure.** There are two ways
to lay out the game's memory on iOS:

- The 64-bit builds' layout. The desktop and android64 builds reserve 4 GB of
  address space, aligned to 4 GB, by reserving 8 GB and keeping the aligned
  half (`port/linux/src/xbox_memory.c`). With a free account, iOS may not
  allow that much. Apple's `extended-virtual-addressing` entitlement, which
  raises the limit, needs a paid account.
- A console-style window. The Xbox 360 port (`HALO_WINDOW_TRANSLATED`,
  `source/cseries/xbox_address.h`) uses only the Xbox addresses `0x78000000`
  to `0x88000000`, a 256 MB block. Its base is a 32-bit number, though, so
  the window has to sit below 4 GB. iOS keeps the low 4 GB of every 64-bit
  app (`__PAGEZERO`), so the 360's mode cannot run unchanged. It would need a
  64-bit base, which is a change to the game's shared code, contributed
  upstream.

The development build measures which one the device allows: its launcher has
a **Memory check** (`app/modules/halo-native`), which does what
`xbox_memory.c` does and reports the result. The game builds as a console
port does (`configure.py --console-port`,
[docs/building.md](https://github.com/ChupathingyCE/chupathingyce/blob/main/docs/building.md#console-ports)),
with the game as a submodule.

**Building without a Mac or a paid account.** iOS apps are built on macOS. Here
that means GitHub Actions' macOS runners (`.github/workflows/ios.yml`). They
build the app unsigned (`expo prebuild`, `pod install`, then `xcodebuild` with
`CODE_SIGNING_ALLOWED=NO`) and attach the `.ipa` to the run. AltStore or
SideStore then signs and installs it with a free Apple ID. With a free Apple
ID, an app runs for 7 days before it must be refreshed, and at most 3
sideloaded apps can be installed at once. EAS Build cannot make device builds
without a paid account.

To install a build:

1. Open the repository's **Actions** tab, then the **iOS** workflow. Pushes to
   `main` build the Debug configuration. **Run workflow** lets you choose
   Debug or Release.
2. Download the run's `ChupathingyCE-<configuration>-ipa` artifact, unzip it,
   and open the `.ipa` in SideStore or AltStore on the device.
3. Debug is the development build. Start `npx expo start` in `app` on the
   computer, open the app, and pick the computer's server. JavaScript changes
   then load as in Expo Go, and only native changes need a new build. Release
   carries its JavaScript and runs on its own.

## Phase 3: tvOS, and releases

- Expo Go does not run on tvOS. The Apple TV app is a development build from
  the start, with `react-native-tvos`.
- tvOS has no file picker. The disc's maps (about 1.8 GB) would come over the
  network, for example from a page the Apple TV serves that a phone or
  computer uploads to.
- tvOS may delete an app's data under storage pressure, so the import may need
  doing again.
- The game is GPL-3.0-or-later, which does not sit with the App Store's terms.
  Sideloading is the route.

## License

GPL-3.0-or-later, as the game is ([LICENSE](LICENSE)). The disc extractor is a
port of chupathingyce's `xiso.c`, whose reader is based on the one in
Milenko's Theseus project (GPL-3.0-or-later). This is a fan project, not made
or endorsed by Microsoft, Bungie or 343 Industries.
