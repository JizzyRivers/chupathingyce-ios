chupathingyce-ios

halo ce on iphone. native, not emulated. its the ios port of chupathingyce https://github.com/ChupathingyCE/chupathingyce

bring ur own copy, the app has none of the game files. u give it ur xbox disc image

install
- get the ipa from releases
- open it in sidestore or altstore
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

doesnt work
- lan / system link. ios blocks broadcast on free accounts, use invites instead, they work on the same wifi
- upnp, same reason. usually fine anyway
- no touch layout editor / gyro / rumble yet
- ipad untested, apple tv later
- tested on one iphone 14 plus lol. expect jank

if it breaks send me the log from the logs folder. "it crashed" w no log doesnt help me

how its built
basically the android64 build (64 bit, sdl3 + gles3) w a few ios fixes on top. disc extraction is chupathingyces own xiso.c. touch controls are the android ones ported over. github actions builds everything on mac runners so u dont need a mac, push a v tag and it makes a release

app/ is the swift app (xcodegen makes the project from app/project.yml)
port/ios is the game side
patches/chupathingyce is the changes to the game
chupathingyce is the game (submodule)

gpl 3 like the game. disc reader based on milenkos theseus. fan project, not microsoft / bungie / 343. nobody touches ui.map
