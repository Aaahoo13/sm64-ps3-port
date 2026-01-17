# Super Mario 64 Port - PS3 Port

- The PS3 port is still in development. Expect issues.
- This repo contains a full decompilation of Super Mario 64 (J), (U), and (E) with minor exceptions in the audio subsystem.
- Naming and documentation of the source code and data structures are in progress.
- Efforts to decompile the Shindou ROM steadily advance toward a matching build.

This repo does not include all assets necessary for compiling the game.
A prior copy of the game is required to extract the assets.

## Using Docker

0. Ensure Git and Docker are installed on your system.
1. Check out repo, submodules, etc.:
```
git clone https://github.com/Aaahoo13/sm64-ps3-port.git -b ps3 --recursive && cd sm64-ps3-port
```
2. Clone the repo ps3toolchain:
```
git clone https://github.com/ps3dev/ps3toolchain.git
```
3. Add 3 lines in ps3toolchain's script "009-ps3libraries.sh" to update the links in order to download `libxml2-2.7.8`, use `tar.xz` in lieu of `tar.gz`, and download `freetype-2.4.3`:
```
awk '
/rm -Rf ps3libraries && mkdir ps3libraries && tar --strip-components=1 --directory=ps3libraries -xvzf ps3libraries.tar.gz && cd ps3libraries/ {
    print
    print "sed -i '\''s|http://xmlsoft.org/download/libxml2-2.7.8.tar.gz|https://download.gnome.org/sources/libxml2/2.7/libxml2-2.7.8.tar.xz|g'\'' scripts/012-libxml2-2.7.8.sh"
    print "sed -i '\''s|xfvz libxml2-2.7.8.tar.gz|xfvJ libxml2-2.7.8.tar.xz|g'\'' scripts/012-libxml2-2.7.8.sh"
    print "sed -i '\''s|http://download.savannah.gnu.org/releases/freetype/freetype-old/freetype-2.4.3.tar.gz|https://download-mirror.savannah.gnu.org/releases/freetype/freetype-old/freetype-2.4.3.tar.gz|g'\'' scripts/004-freetype-2.4.3.sh"
    next
}
{ print }
' ps3toolchain/scripts/009-ps3libraries.sh > /tmp/ps3lib && mv /tmp/ps3lib ps3toolchain/scripts/009-ps3libraries.sh
```
4. Copy in your `baserom.<region>.z64`, where &lt;region> can be us, jp, or eu:
```
cp /path/to/baserom.<region>.z64 .
```
5. Build Docker image:
```
docker build . -t sm64_ps3
```
6. Compile using your Docker image.  
You can precise the region with `VERSION=<region>`, where &lt;region> can be us, jp, or eu (us is default).  
You can also add `-j4` if ou have 4 cores, for instance, or `-j$(nproc)` if you want to use all the cores and you base your Docker image on Debian instead of Ubuntu.
```
docker run --rm -ti -v $(pwd):/sm64 sm64_ps3 make VERSION=<region> -j4
```
The resulting .self will be in `build/<region>_ps3/`.

7. In order to produce the .pkg file, you can have the default PSL1GHT icon without music nor background picture:
```
docker run --rm -ti -v $(pwd):/sm64 sm64_ps3 make build/<region>_ps3/sm64.<region>.f3dex2e.pkg
```
To avoid having the default PSL1GHT icon, copy the correct icon (320×176) with the name "ICON0.PNG" into the repository's root directory, and change the Makefile:
```
sed -i 's/ICON0[[:space:]]*:=/ICON0     ?=/g' Makefile
```
```
docker run --rm -ti -v $(pwd):/sm64 sm64_ps3 make build/<region>_ps3/sm64.<region>.f3dex2e.pkg ICON0=ICON0.PNG
```
To add the possibility of having a music and background picture, you'll have to copy the audio file (ATRAC3 / ATRAC3+) and picture (1920×1080), then modify the Makefile:
```
sed -i '/ICON0[[:space:]]*?=/a\
  PIC1      ?= PIC1.PNG\
  SND0      ?= SND0.AT3' Makefile
```
```
sed -i '/cp $(ICON0)*/a\
\tcp $(PIC1) $(BUILD_DIR)/pkg/PIC1.PNG\
\tcp $(SND0) $(BUILD_DIR)/pkg/SND0.AT3' Makefile
```
You can then produce the .pkg file with the desired icon, music and background picture. Add `-j4` to improve build speed (hardware dependent based on the amount of CPU cores available):
```
docker run --rm -ti -v $(pwd):/sm64 sm64_ps3 make build/<region>_ps3/sm64.<region>.f3dex2e.pkg ICON0=ICON0.PNG -j4
```

## Manually under Linux (WSL and MSYS2 not tested)

0. Ensure Git, GCC, GNU Make and Python 3 are installed on your system :
```
# for example on Ubuntu
sudo apt install git build-essential python3
```
1. Check out repo, submodules, etc.: 
```
git clone https://github.com/Aaahoo13/sm64-ps3-port.git -b ps3 --recursive && cd sm64-ps3-port
```
2. Clone the repo ps3toolchain:
```
git clone https://github.com/ps3dev/ps3toolchain.git
```
3. Same step as the step 3 in the Docker version to update the links in order to download `libxml2-2.7.8`, use `tar.xz` in lieu of `tar.gz`, and download `freetype-2.4.3`.
4. Ensure PSL1GHT is installed on your system and the environmental variables `PS3DEV` and `PSL1GHT` are defined and PSL1GHT is in your `PATH`.
You can follow the installation instructions in the [ps3toolchain repo](https://github.com/ps3dev/ps3toolchain).
5. Install [Cg Toolkit](https://developer.nvidia.com/cg-toolkit-download).
6. Copy in your `baserom.&lt;region>.z64`, where &lt;region> can be us, jp, or eu:
```
cp /path/to/baserom.<region>.z64 .
```
7. Follow the same last 2 steps as the Docker version (6 & 7) without `"docker run --rm -ti -v $(pwd):/sm64 sm64_ps3"` at the beginning of the commands.

## Project Structure

```
sm64
├── actors: object behaviors, geo layout, and display lists
├── asm: handwritten assembly code, rom header
│   └── non_matchings: asm for non-matching sections
├── assets: animation and demo data
│   ├── anims: animation data
│   └── demos: demo data
├── bin: C files for ordering display lists and textures
├── build: output directory
├── data: behavior scripts, misc. data
├── doxygen: documentation infrastructure
├── enhancements: example source modifications
├── include: header files
├── levels: level scripts, geo layout, and display lists
├── lib: SDK library code
├── rsp: audio and Fast3D RSP assembly code
├── sound: sequences, sound samples, and sound banks
├── src: C source code for game
│   ├── audio: audio code
│   ├── buffers: stacks, heaps, and task buffers
│   ├── engine: script processing engines and utils
│   ├── game: behaviors and rest of game source
│   ├── goddard: Mario intro screen
│   ├── menu: title screen and file, act, and debug level selection menus
│   └── pc: port code, audio and video renderer
├── text: dialog, level names, act names
├── textures: skybox and generic texture data
└── tools: build tools
```

## Contributing

Pull requests are welcome. For major changes, please open an issue first to
discuss what you would like to change.

Run `clang-format` on your code to ensure it meets the project's coding standards.

Official Discord: https://discord.gg/7bcNTPK
