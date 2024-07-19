# Changes in this fork
- Can use the bracelet with the B button without having it equipped
- Disabled heart beeping
- Slightly sped up the menu transition


## About oracles-disasm

This is a complete, documented disassembly of Oracle of Ages and Seasons for the Gameboy
Color. When combined with [LynnaLab](https://github.com/stewmath/lynnalab), it is a level
editing suite.

This repository builds US version ROMS. JP and EU versions are not supported.

[See the wiki](https://wiki.zeldahacking.net/oracle/Setting_up_oracles-disasm)
for detailed setup instructions.


## Current status

The disassembly complete enough to be reassembled with ROM addresses shifted
around arbitrarily. This is fairly well tested through the
[randomizer](https://github.com/Stewmath/oracles-randomizer-ng-webui).

However there is still work to be done:

- Support for Japanese and European version ROMs. This could be done either in
  this repository or with a fork. Documenting the version differences could be
  invaluable for glitch hunting, as Nintendo of Europe had a crack team of
  grizzled QA testers working on these games (that's how it seems anyway...)
- RAM address shifting is not well-tested. There are still some references to
  hardcoded RAM addresses scattered around.
- Documentation and variable/function naming can always be improved; in
  particular, Seasons documentation is relatively lacking compared to Ages. Areas
  to focus on include:
  - Most stuff under object_code/seasons
  - Functions named "seasonsFunc_[...]" or just "func_[...]"
  - Things marked as TODO
- If some genius could figure out how the original compression algorithms worked
  we could get rid of the whole precompressed asset nonsense!


## Branches

I've been extensively (ab)using branches in the main repository
(https://github.com/stewmath/oracles-disasm) for various features and
experiments. Most of these are stale branches for one-off experiments or hacks
that I didn't bother creating separate repositories for.

The following branches are maintained (or at least updated every once in a
while):

- master: Builds vanilla ROMs.
- hack-base: Has some tweaks to make hacking with LynnaLab easier, mainly
  deduplicated tilesets.
- rando/ng: Contains patches used by the
  [randomizer](https://github.com/stewmath/oracles-randomizer-webui). Tracked as
  a submodule.
- hack/crossitems: Ages items available in Seasons and vice-versa. Merged into
  rando/ng (and hack-base eventually).

Other notable branches (not necessarily maintained):

- vwf: Variable-width font hack

In order to keep track of which patches are applied to a particular branch I'm
using certain "tags" prefixed to each commit name, for example:

- HACK-BASE: Patches for hack-base branch
- CROSSITEMS: Patches for hack/crossitems branch
- RANDO: Patches for rando/ng branch
- DEBUG: Debugging features (could be cherry-picked to other branches)
- MUSIC: Music-related patches, mostly new tracks in rando/ng branch

These tags also used in comments for the corresponding commits, which helps to
understand when the code you're looking at has been modified from the master
branch (without needing to use fancy git-fu).


## Required tools to build

* Python 3
* python3-yaml (python module)
* [WLA-DX](https://github.com/vhelin/wla-dx) v10.6
* On windows: Must use MSYS2 or some other linux-y environment.

Note: WLA will not produce an exact matching Seasons ROM due to quirks with how
empty space is handled; however, this has no effect on how the game functions.
If you need to work around this for some reason, [This
branch](https://github.com/Drenn1/wla-dx/tree/emptyfill-banknumber) of WLA can
be used instead (but it needs to be updated!)


## Build instructions

Once the dependencies are installed, running `make` will build both games. To build
a specific game, run `make ages` or `make seasons`.

By default, the rom is built with precompressed assets, so that an exact copy of the
original game is produced. In order to edit text, graphics, and other things,
switch to the "hack-base" branch. Alternatively, run the "./swapbuild.sh" script
in the root of the repository. This will switch the build mode to "modifiable"
instead of "precompressed".

There are 4 build directories in total (for ages and seasons, vanilla or
editable), selected by the makefile at build time. See Makefile for details.

[See the wiki](https://wiki.zeldahacking.net/oracle/Setting_up_oracles-disasm) for detailed
setup instructions.


## Graphics files

(Note: Graphics editing will only work if you're on the "hack-base" branch or
have disabled the use of precompressed graphics)

Graphics are stored as 4-color indexed PNG files. Other formats (RGB) are
supported, as long as you stick to using the original 4 colors. But the indexed
format works particularly well with editors such as Aseprite, which show you the
4-color palette available to you.

Some graphics have a corresponding `.properties` file (ie. `spr_link.png` has
`spr_link.properties`). These are YAML files which specify certain properties
about how they should be converted from PNG format to BIN (raw) format, and
vice-versa.

The following parameters are accepted in `.properties` files:

* width (int): Width of the resulting PNG file, in tiles. Only affects .BIN ->
  .PNG conversion.
* interleave (bool): Whether to treat the image with an 8x16 layout instead of
  an 8x8 layout (mainly for sprites).
* invert (bool): Whether to invert the color order of the PNG palettes. When
  "false", the order is light-to-dark (white = color 0). When "true", the order
  is dark-to-light (black = color 0).
* tile\_padding (int): Number of tiles of padding at the end of the image. This
  many tiles will be truncated before conversion to `.BIN` format, or this many
  tiles will be added during conversion to .PNG format.
* format (string): Set this to "1bpp" for 1 bit-per-pixel files (only the font
  uses this).

Sprite graphics files (files beginning with `spr_` instead of `gfx_`) are
assumed to have the properties `invert: true` and `interleave: true`. However,
these can be overridden by creating a `.properties` file.

If you wish to edit the files in raw .BIN format with an editor such as YY-CHR,
run the following command from the root of the repository (using `spr_link.png`
as an example):

```
python3 tools/gfx/gfx.py auto gfx/common/spr_link.png
```

This will create `gfx/common/spr_link.bin`, a raw 2bpp gameboy image. However
you shouldn't have both a `.bin` and `.png` file with the same name; this will
confuse the Makefile rules. You can simply remove `gfx/spr_link.png`, in which
case the disassembly will read from `gfx/spr_link.bin` instead. Or, you may
convert it back to PNG when you're done, then delete the `.bin` file. The
following command will convert the `.bin` file back to `.png`:

```
python3 tools/gfx/gfx.py png gfx/common/spr_link.bin
```

Both of these commands will check the `.properties` file, if it exists, to
decode and encode the image properly.

## Disclaimer

The reverse-engineered code and assets in this repository belong largely to
Capcom and Nintendo. While I can't really stop you from doing what you want with
it, I strongly disavow its use for any commercial purposes. The purpose of this
project is to research the inner workings of the Zelda Oracle games and
facilitate the creation of non-commercial ROM hacks.

Scripts which do not contain any Nintendo/Capcom code (ie. python scripts in the
"tools/" folder) may be considered "public domain" unless stated otherwise.
