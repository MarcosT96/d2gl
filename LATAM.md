# D2 LATAM modifications

This is a fork of [bayaraa/d2gl](https://github.com/bayaraa/d2gl) (GPLv3) based on
upstream `master` at `919df3528de5c5c1b41b326cf68821cd1981c42c`. It is distributed
under the same licence (see `LICENSE.md`); the changes below are marked as
required by GPLv3 section 5(a). Upstream copyright notices are untouched.

Changes (branch `latam/1.14d`):

1. `d2gl/src/option/ini.cpp` — `saveIni()` no longer truncates and rewrites
   `d2gl.ini` on every start. It writes only when the content changed, through
   `<file>.tmp` + `MoveFileExA(MOVEFILE_REPLACE_EXISTING)`. Upstream rewrote the
   file unconditionally, so a second client starting while the first one parsed it
   could read an empty or half-written ini.
2. `d2gl/src/modules/hd_text/variables.h` — the Spanish death-screen texts follow
   the retail Spanish art ("Has muerto", "Pulsa ESC para continuar", "Has perdido
   oro"). Upstream had a German sentence on the first line.
3. New ini key `[Other] lang_override` (`eng`, `esp`, `deu`, `fra`, `por`, `ita`,
   `pol`, `rus`; empty = unchanged upstream behaviour). The HD text tables are
   chosen by the language the game reports, which is the language of the player's
   Windows; a payload that ships one language's menu art under every language
   slot needs the tables to follow the art. Implemented as
   `d2::getLangIdEffective()` (`d2/funcs.cpp`), used by `helpers.cpp` and
   `modules/hd_text.cpp`.

Build: Visual Studio 2019 toolset (`PlatformToolset=v142`), `Release|Win32`,
project `glide3x`. The upstream release was built with v143 (linker 14.38).
