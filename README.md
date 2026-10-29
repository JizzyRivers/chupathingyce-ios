chupathingyce-ios

halo ce on iphone, ipad, apple tv and vision pro. native, not emulated. its the apple port of chupathingyce https://github.com/ChupathingyCE/chupathingyce

bring ur own copy, the app has none of the game files. u give it ur xbox disc image

install
- get the ipa from releases. ChupathingyCE-vX-iOS.ipa is iphone, ipad and vision pro. ChupathingyCE-vX-iPhoneDuo.ipa is iphone duo. ChupathingyCE-vX-tvOS.ipa is apple tv
- iphone / ipad: open it in sidestore or altstore
- apple tv / vision pro: sideloadly (or xcode if u have a mac). sidestore doesnt do tv
- free apple id is fine, refresh every 7 days, 3 apps max (apple rules not mine)
- installing a new build over the old one keeps ur maps and saves

first time
- pick ur disc image (.xiso or .iso). it copies the maps + movies, like 2.2gb, keep the app open while it does it
- or pick a zip of ur maps folder, or paste a link to one
- hit play

pc / ce / halomd maps go in thru import maps, .map or .zip, it sorts them itself
ur files are in the files app, on my iphone > ChupathingyCE (maps, saves, save/config.toml, logs)

playing
- controller works, pair it in bluetooth settings
- touch controls show up in game. stick left buttons right swipe to aim. hide button up top
- they show even w a controller in, set input.touch_controls to auto in config.toml to fix that

online
- invite links work (halo://join/...), tap one and it joins
- server browser works
- say yes to local network when it asks

ipad
- same app as the iphone. touch controls stay phone sized and spread to the corners so u can reach them
- the picture follows the window, so stage manager / slide over sizes letterbox instead of stretching

iphone duo
- grab ChupathingyCE-vX-iPhoneDuo.ipa, not the plain ios one (see below why)
- closed: the outside screen is a normal phone
- open flat: the inside screen is one big screen, picture reshapes to it live, no restart
- half open like a laptop (fold across the screen, like 70 degrees): game on the top half, controller deck on the bottom half. stick + dpad left, abxy + fire right, the empty middle is a big swipe to aim pad
- half open like a book (fold up the middle): normal layout but no buttons on the fold
- why a separate ipa: ios only gives apps the full inner screen + the fold stuff if theyre built w the ios 27.1 sdk. github's mac runners dont have xcode 27.1 yet, so the duo ipa is the same app w its binary marked as 27.1 (apples vtool). once github gets 27.1 the normal ipa does it all and the duo one goes away by itself
- the duo stuff is still early. if the laptop mode does something weird send the log, it says when it sees the fold

apple tv
- controller only, the siri remote doesnt have enough buttons. pair one in settings > remotes and devices
- no files app on tv so the app runs a little upload page. open the address it shows (http://ur-tv-ip:8080) on ur phone or pc on the same wifi and send the disc image or a maps zip
- or download from a link like on the phone
- tvos can wipe app storage when the tv is low on space (thats apple, everything lives in caches on tv). if ur maps vanish send them again. saves live there too so yeah
- disc image upload needs room for the image + the 2.2gb it extracts, then the image gets deleted

vision pro
- runs the ipad app in compatibility mode (a window in ur room). visionos has no opengl es for real visionos apps, so a native one would need a metal renderer, not here yet
- use a controller
- untested, i dont have one. if it doesnt start send the log

doesnt work
- lan / system link. ios blocks broadcast on free accounts, use invites instead, they work on the same wifi
- upnp, same reason. usually fine anyway
- no touch layout editor / gyro / rumble yet
- tested on one iphone 14 plus lol. ipad, tv and vision pro builds are untested on real hw, duo is still early. expect jank

if it breaks send me the log from the logs folder. "it crashed" w no log doesnt help me

how its built
basically the android64 build (64 bit, sdl3 + gles3) w a few ios fixes on top. disc extraction is chupathingyces own xiso.c. touch controls are the android ones ported over. github actions builds everything on mac runners so u dont need a mac, push a v tag and it makes a release

app/ is the swift app (xcodegen makes the project from app/project.yml). app/Sources/Shared is both apps, iOS is the iphone/ipad launcher, tvOS is the tv launcher + upload page
port/ios is the game side, same code for ios and tvos (ninja ios / ninja tvos). the duo fold + hinge stuff is in there too (port/ios/src/ios_touch.m)
patches/chupathingyce is the changes to the game
chupathingyce is the game (submodule)

gpl 3 like the game. disc reader based on milenkos theseus. the apple tv upload page + tv storage idea came from looking at steverice's visr https://github.com/steverice/visr, none of its code is in here. fan project, not microsoft / bungie / 343. nobody touches ui.map
