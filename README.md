# binaries-avif

A repository with static and dynamic binaries for libavif.

It offers x64 & arm64 versions for macOS, Linux, MinGW and Windows.

## 📦 Release assets

| Asset                       | Contents                                                                              |
|-----------------------------|---------------------------------------------------------------------------------------|
| `static_<os>_<arch>.zip`    | libavif and its dependencies as separate static libraries, with all their headers     |
| `dynamic_<os>_<arch>.zip`   | the same libraries as shared libraries (`.dylib`, `.so`, `.dll`)                       |
| `bundle_<os>_<arch>.zip`    | ONE self-contained static library with private symbols prefixed (see below)           |

`<os>` is `osx`, `linux`, `mingw` or `windows` (MSVC); `<arch>` is `x64` or `arm64`.

### Bundle

The bundle is meant for programs that statically link libavif **and** another copy of one of its dependencies, typically
a full FFmpeg build, which bundles its own dav1d, SVT-AV1 and fastfeat. With the separate static libraries, two copies of
the same library end up in the same link: either the linker fails with duplicate symbols, or it silently mixes code from
two different versions.

`bundle_<os>_<arch>.zip` contains:

- `include/avif/`: libavif's public headers only.
- `lib/libavif.a` (`lib/avif.lib` on Windows/MSVC): libavif merged with the dependencies it needs: dav1d, SVT-AV1,
  fastfeat and libyuv. Only the parts libavif can reach are merged, as a linker would pick them, so libyuv's MJPEG
  decoder is left out, and with it libjpeg/libturbojpeg, which nothing else uses.

Every symbol in that library except libavif's public C API (`avif*`) is renamed with the prefix `avifrs_`: dav1d,
SVT-AV1, fastfeat, libyuv and C++ template instantiations (`dav1d_version` → `avifrs_dav1d_version`). The bundle
therefore can't clash with any other copy of those libraries, whatever its version. The `avif*` API is unchanged, so
code using libavif's headers needs no change. Link it alone, plus the system and C++ runtime libraries (e.g. `-lc++` on
macOS, `-lstdc++ -lpthread -ldl -lm` on Linux, `-lstdc++ -lpthread` on MinGW). On MSVC the runtime comes from the
library's `/DEFAULTLIB` directives (`/MD`), plus `kernel32.lib` and `advapi32.lib`.

The bundle is produced by [scripts/bundle.sh](scripts/bundle.sh) from the same stripped libraries as the static zip.
Each build verifies it: no unprefixed private symbol is defined or referenced, a small program encodes an image with
SVT-AV1, decodes it with dav1d and converts it to RGB with libyuv, and the same program links cleanly with the bundle
**and** the original unprefixed dav1d, SVT-AV1, fastfeat and libyuv forced in together.

## 👨🏾‍💻 Author

Vinicius Egidio ([vinicius.io](http://vinicius.io))
