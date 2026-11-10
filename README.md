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

All dependencies come from the official repositories: the app targets
.NET 8 (`dotnet-runtime-8.0`) but is built with the **.NET 10 SDK**
(`dotnet-sdk-10.0`, also satisfied by `dotnet-sdk`) plus
`dotnet-targeting-pack-8.0`. NuGet packages are downloaded during `prepare()`
from nuget.org and PixiEditor's own Azure DevOps feed.

Why not the official `dotnet-sdk-8.0`: Arch builds it from source, so it is in
the 8.0.1xx feature band with the Roslyn 4.8 compiler, while PixiEditor's
source generators need Roslyn 4.11 (`CS9057`, then dozens of missing
`*.Generated` types). The AUR `pixieditor` package used the AUR
`dotnet-sdk-8.0-bin` (Microsoft's 8.0.4xx) for that reason; building with the
official 10.0 SDK avoids the AUR dependency.

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
`src/PixiEditor.Desktop/PixiEditor.Desktop.csproj` (`_net` in the PKGBUILD).

Three deliberate deviations from a plain upstream build, each commented in
the `PKGBUILD`:

- `dotnet restore` runs with `-p:Configuration=Release`: in Debug the projects
  also reference `Avalonia.Diagnostics` at the pinned Avalonia CI build, which
  PixiEditor's feed does not have, and the restore fails with NU1605.
- `src/global.json` (pinned to SDK 8.0.405) is rewritten to the SDK version in
  `_sdk`.
- The build passes `-p:LangVersion=12`: the projects use `LangVersion=latest`,
  which is C# 14 with the 10.0 SDK, and there `array.Reverse()` binds to the
  in-place `Span` overload and breaks the build. C# 12 is what upstream's
  8.0.4xx SDK compiles with.

`_net` is the target framework/runtime and `_sdk` the SDK used to build; if
upstream moves to a newer `TargetFramework`, update `_net` (and check whether
the `LangVersion` pin is still needed).

## License

PixiEditor is licensed under the **LGPL-3.0-or-later**. This repository
contains only the packaging recipe; the source code is downloaded from GitHub
during the build.
