# pixieditor (Arch Linux PKGBUILD)

Arch Linux `PKGBUILD` that builds **PixiEditor**, an all-in-one solution for
2D image editing
([pixieditor.net](https://pixieditor.net/),
[PixiEditor/PixiEditor](https://github.com/PixiEditor/PixiEditor)).

This is a continuation of the
[AUR `pixieditor`](https://aur.archlinux.org/packages/pixieditor) package,
kept up to date with newer upstream releases. Full credit for the original
packaging goes to the maintainer listed in the `PKGBUILD` header; this repo only
tracks new upstream versions on top of that work.

## What you get

- `pixieditor` — the application, built from source against the system
  .NET 8 runtime (framework-dependent, not self-contained)
- desktop entry, icon, AppStream metadata and the MIME type for `.pixi` files
- PixiEditor's bundled `ffmpeg` path pointed at the system `ffmpeg`

PixiEditor renders through the GPU (Vulkan by default, OpenGL with
`pixieditor --opengl`), so it needs working graphics drivers.

Source builds get an empty `appsettings.json`: the official binaries ship the
PixiEditor API URL and key there, so features that talk to
`auth.pixieditor.net` are not available in this build.

## Building & installing

```bash
git clone https://github.com/FoxKyong/aur-pixieditor.git
cd aur-pixieditor
makepkg -si
```

The build dependency `dotnet-sdk-8.0-bin` is itself an AUR package
(package base [`dotnet-core-8.0-bin`](https://aur.archlinux.org/pkgbase/dotnet-core-8.0-bin),
a repack of Microsoft's SDK tarball) and has to be built first; an AUR helper
resolves it automatically. NuGet packages are downloaded during `prepare()`
from nuget.org and PixiEditor's own Azure DevOps feed.

## Updating to a new version

Releases are tagged on
[GitHub](https://github.com/PixiEditor/PixiEditor/releases) (tags without a
`v` prefix, e.g. `2.1.2.5`). To bump:

1. Set `pkgver` to the new version.
2. Regenerate the checksum: `makepkg -g` and paste the printed `b2sums=(...)`
   (only the first entry changes; the submodules are `SKIP`).
3. Regenerate metadata: `makepkg --printsrcinfo > .SRCINFO`.
4. Verify it builds: `makepkg -sf`.

Before building, check the release's `.gitmodules` (the four submodules are
wired up by hand in `prepare()`) and the `TargetFramework` in
`src/PixiEditor.Desktop/PixiEditor.Desktop.csproj` (`_sdk` in the PKGBUILD).

`dotnet restore` runs with `-p:Configuration=Release` on purpose: in Debug the
projects also reference `Avalonia.Diagnostics` at the pinned Avalonia CI build,
which PixiEditor's feed does not have, and the restore fails with NU1605.

## License

PixiEditor is licensed under the **LGPL-3.0-or-later**. This repository
contains only the packaging recipe; the source code is downloaded from GitHub
during the build.
