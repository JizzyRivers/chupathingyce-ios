# chupathingyce-ios

ok so. halo ce. on ur iphone. natively. not streamed not emulated not some cloud thing, the actual game running on the actual phone in ur actual hand, like a sopranos character holding a warthog he stole from a parking garage. this is the ios / ipados port of [ChupathingyCE](https://github.com/ChupathingyCE/chupathingyce), the halo ce decomp port, and it was called warthog-ios while i was building it bc thats what the console ports are called (warthog-xbox, warthog-360, warthog-switch, warthog-wiiu) and honestly the warthog is the main character of halo and everyone knows it

**bring ur own copy.** the app ships with zero of the games files. u give it ur xbox disc image and it pulls the maps out itself. it never downloads anything from microsoft and it never will, capisce

## grab it

go to [releases](../../releases), get the newest `ChupathingyCE-<version>.ipa`, open it in **SideStore** or **AltStore** on ur phone. thats it. no mac, no paid apple dev account, no app store (the game is gpl and the app store and gpl do not get along, its like two families at a wedding)

free apple id rules, these are apples not mine:
- the app runs 7 days then sidestore / altstore has to refresh it. just refresh it
- 3 sideloaded apps max on a free account
- installing a new build over the old one keeps ur maps and saves, same bundle id

## first launch

1. open it, it says the games data isnt here yet. tap **pick the disc image** and pick ur halo ce xbox image (`.xiso` or a full disc `.iso`, any version). it reads it right where it is so it doesnt eat another 7gb copying it first
2. it copies out the maps + the bink movies, about 2.2gb with room to spare. **keep the app open**, ios pauses apps in the background and the copy stops
3. no disc image? pick a `.zip` of ur discs maps folder instead, or paste a link to ur own zip in **download maps**
4. **play**

halo pc / custom edition / halomd maps go in through **import maps** (`.map` files or `.zip`s of them). it reads each maps header and puts it in the right folder by itself, maps / maps_ce / maps_pc / maps_md, crc checked so a busted zip gets skipped instead of blowing up in ur face

ur stuff lives in the files app under **On My iPhone → ChupathingyCE**: maps, saves, `save/config.toml`, logs. all of it

## playing it

- **controller**: xbox, dualsense, mfi, whatever. pair it in ios settings → bluetooth and it just goes
- **touch**: the on screen controller shows up once ur in a level. stick bottom left, buttons right, swipe anywhere empty to aim, drag a button ur holding to aim and shoot with one thumb like a lunatic. **hide** up top puts it away. it hides itself over menus and cutscenes, those just take taps
- the controls show even with a controller plugged in bc `input.touch_controls` is `on`. set it to `auto` in `config.toml` and they hide whenever a controller is connected

## internet play

- **invites** work. a `halo://join/...` link in messages or safari opens the app and it joins. game not running yet? the launcher says an invite is waiting and it joins when u hit play
- server browser and online games go thru the same stuff the pc and android builds use
- first time online ios asks for **local network** access. say yes

## what doesnt work (yet, or ever, its apple)

- **lan / system link**: no. finding lan games is udp broadcast and ios blocks broadcast unless the app has apples multicast entitlement, which free accounts cannot get. use invites, they work on the same wifi too
- **upnp**: same reason. the game falls back to punching thru ur router directly, which works on most home internet but strict mobile carriers can still say no
- **touch layout editor**: not yet. no moving / resizing buttons, no gyro aim, no phone rumble. android has all of it, its next
- **ipad**: should work, nobody has actually tried it yet
- **apple tv**: later. tvos has no file picker so the maps gotta come over the network somehow
- this was played on exactly one iphone 14 plus on ios 27. expect jank. it is a test build. it is a guy in a garage

## when it breaks

send the log. files app → **On My iPhone → ChupathingyCE → logs**. "it crashed" with no log is like telling the mechanic the car makes a noise. what noise. where. i need the noise

## how the sausage got made

the whole thing stands on the other ports, they did the hard parts and i just drove the warthog thru the hole they already blew in the wall:

- **android64** is the base. native 64 bit, talks to sdl3 + opengl es 3 straight up, no guest loader. ios has both so the game builds as that with `HALO_ANDROID` for the phone stuff and `HALO_IOS` where ios is weird. it even says "android arm64" in the log and to other players. i left it, the games platform limits might key off that name and i aint poking that bear yet
- **xiso.c**, the disc reader (based on milenkos theseus), extracts the disc. the swift launcher just calls the engines own c
- **macos** already fought apples gl so most of the apple weirdness was solved before i showed up
- the ios specific fights:
  - ios is opengl es 3.0, no `glDrawElementsBaseVertex`, and the game used to quietly give up on its window over it. music played, screen black, very spooky
  - no framebuffer 0 on ios, the game draws into sdls
  - controllers: the game ran inside a main queue block that never returned and the gamecontroller framework talks thru that queue so sdl never heard about any controller. it starts from a run loop block now
  - ios keeps its root certificates where apps cant read them so the app carries mozillas (curls copy) for https
  - free accounts get a 4gb address space reservation but not 8, so the xbox address space is reserved as 4gb directly
- the touch controls are androids `TouchControls.java` ported to a uikit overlay (`port/ios/src/ios_touch.m`)

### the layout

| where | what |
| --- | --- |
| `app/` | the native app, swift + uikit. the xcode project comes from `app/project.yml` via [XcodeGen](https://github.com/yonaskolb/XcodeGen) so theres no project file to fight with |
| `app/Sources` | the launcher: disc import (`MapImport.swift` → `xiso.c`), map import + zip reading (`Zip.swift`, the systems zlib), downloads, starting the game |
| `port/ios` | the game for ios: its build (`build.py`), entry point (`ios_main.c`), gl helpers, touch controls, the games own stack |
| `patches/chupathingyce` | the changes to the game itself, applied to the submodule at build time |
| `chupathingyce` | the game, as a submodule |

### building it urself

u dont need a mac, github does it. its macos runners build everything in `.github/workflows/ios.yml`: sdl3 for ios, the game with `configure.py --release --console-port`, ffmpeg for the bink movies, then xcodegen + `xcodebuild` unsigned, and the `.ipa` lands on the run. push a `v<version>` tag and it becomes a release on its own. `.github/workflows/engine.yml` builds just the game when only the port changed

the game builds as a console port does, see chupathingyce's [docs/building.md](https://github.com/ChupathingyCE/chupathingyce/blob/main/docs/building.md#console-ports)

## license

GPL-3.0-or-later, same as the game ([LICENSE](LICENSE)). the disc extractor is chupathingyces `xiso.c`, its reader based on the one in milenkos theseus project (GPL-3.0-or-later). fan project, not made or endorsed by microsoft, bungie or 343 industries. nobody touches ui.map
