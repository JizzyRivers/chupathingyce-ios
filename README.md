# warthog-ios

An iPhone, iPad and Apple TV port of [ChupathingyCE](https://github.com/ChupathingyCE/chupathingyce),
the Halo: Combat Evolved decompilation port. As with the console ports
(warthog-xbox, warthog-360, warthog-switch, warthog-wiiu), the game lives in its own
repository and this one holds only the port.

You bring your own copy of the game: the app copies the maps from your Xbox disc
image and never downloads Microsoft's files.

## Status

| Phase | What | State |
| --- | --- | --- |
| 1 | The launcher: disc import, map import, map download | Works on an iPhone 14 Plus (iOS 27.0) |
| 2 | The game, native, with controllers and touch controls | Plays on the iPhone 14 Plus: the campaign, a controller, the on-screen controls |
| 3 | iPad and Apple TV, settings, the touch layout editor | Not started |

## The app

`app/` is a native iOS app in Swift and UIKit. Its Xcode project is made from
`app/project.yml` by [XcodeGen](https://github.com/yonaskolb/XcodeGen), so no
project file is checked in. The launcher (`app/Sources`) imports the game's
data and starts the game, which is linked in as `libhalo.a`.

| Android launcher (`port/android/app/.../decomp/`) | Here |
| --- | --- |
| `XisoExtractor.java` | the engine's own `port/linux/src/xiso.c`, called from `app/Sources/MapImport.swift` |
| `LauncherActivity.java` | `app/Sources/LauncherViewController.swift` |
| the data folder (`getExternalFilesDir`) | the app's Documents folder, with the same layout, shown in the Files app (`app/Sources/App.swift`) |
| `MapImporter.java` | `app/Sources/MapImport.swift` and `app/Sources/Zip.swift`. It also takes Xbox maps (into `maps/`), so a zip of a disc's maps folder can stand in for the disc image |
| (none) | `app/Sources/Download.swift`: downloads a `.zip` of maps, or a `.map`, from an address the player types in, then imports it |
| `TouchControls.java` | `port/ios/src/ios_touch.m`, a UIKit overlay over SDL's view. It doesn't have the layout editor, gyro aim or rumble yet |

## The game

`port/ios` builds the game as the Android port's native 64-bit build does: 64-bit code that
calls SDL3 and OpenGL ES 3 directly, with `HALO_ANDROID` for the mobile
platform layer and `HALO_IOS` where iOS differs. The changes to the game itself are
`patches/chupathingyce/0001-ios-port-hooks.patch`. The game is a submodule
and builds as a console port does (`configure.py --console-port`,
[docs/building.md](https://github.com/ChupathingyCE/chupathingyce/blob/main/docs/building.md#console-ports)).
The movies' FFmpeg is built for the iPhone SDK and goes into `libhalo.a`.

With a free Apple account, iOS lets an app reserve a 4 GB block of address
space but not 8 GB, so the game's Xbox address space is reserved directly as
4 GB, aligned to 4 GB where possible (`port/ios/src/ios_main.c`).

## Building without a Mac or a paid account

iOS apps are built on macOS. Here that means GitHub Actions' macOS runners
(`.github/workflows/ios.yml`). They build the game, generate the Xcode project,
build the app unsigned (`xcodebuild` with `CODE_SIGNING_ALLOWED=NO`) and attach
the `.ipa` to the run. AltStore or SideStore then signs and installs it with a
free Apple ID. With a free Apple ID, an app runs for 7 days before it must be
refreshed, and at most 3 sideloaded apps can be installed at once.

To install a build:

1. Open the repository's **Actions** tab, then the **iOS** workflow. Pushes to
   `main` build the Release configuration. **Run workflow** can build Debug.
2. Download the run's `ChupathingyCE-<configuration>-ipa` artifact, unzip it,
   and open the `.ipa` in SideStore or AltStore on the device.

`.github/workflows/engine.yml` builds only the game, for changes to the port.

## Later: tvOS

- tvOS has no file picker. The disc's maps (about 1.8 GB) would come over the
  network, for example from a page the Apple TV serves that a phone or
  computer uploads to.
- tvOS may delete an app's data under storage pressure, so the import may need
  doing again.
- The game is GPL-3.0-or-later, which does not sit with the App Store's terms.
  Sideloading is the route.

## License

GPL-3.0-or-later, as the game is ([LICENSE](LICENSE)). The disc extractor is
chupathingyce's `xiso.c`, whose reader is based on the one in Milenko's
Theseus project (GPL-3.0-or-later). This is a fan project, not made
or endorsed by Microsoft, Bungie or 343 Industries.
