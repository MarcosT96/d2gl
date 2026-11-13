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
   Upstream's `saveIni()` printed `shader_preset` with `%d` while passing
   `App.shader.preset.c_str()`, i.e. it wrote the string's *address* as a number
   (`shader_preset=47428344`) on every start and relied on `upscaler.cpp` to
   overwrite it a moment later with the real name; during that window another
   client could read a bogus preset. Fixed to `%s`.
   `saveValue()` also skips the write when the stored value already matches
   (`WritePrivateProfileString` rewrites the whole file for an unchanged value,
   and `upscaler.cpp` saves `shader_preset` on every start).
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

4. `glide3x/src/wrapper.cpp/.h` — `grLfbLock()` accepted only
   `GR_LFBWRITEMODE_8888` and returned `FXFALSE` for everything else, so the lobby
   banner (which asks for another write mode) was never drawn. It now accepts
   565/555/1555/888/8888 for `GR_ORIGIN_UPPER_LEFT` and converts to BGRA in
   `grLfbUnlock()`, the format the movie texture uploads. Same defect class as the
   D2DX RGB565 banner fix (`d2dx/README.md`, 2026-10-31).
5. `d2gl/src/helpers.cpp` — two small functions, `__std_find_trivial_4` and
   `__std_count_trivial_1`, compiled only for `_MSC_VER < 1930`. The vendored
   `glslang.lib` was built with MSVC 14.38 and its STL calls them; the v142
   (14.2x) import library does not export them, so the link fails (LNK2001).
   Signatures were read from the call sites inside `glslang.lib` (`__stdcall`,
   three 4-byte arguments, result in EAX).

Build: Visual Studio 2019 toolset (`PlatformToolset=v142`), `Release|Win32`,
project `glide3x`. The upstream release was built with v143 (linker 14.38).
