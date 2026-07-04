[中文使用说明](README.zh-CN.md) | English

# KK/KKS VideoExport ReShade Output

This fork publishes the VideoExport changes needed to export color/depth frame
sequences through the modified ScreenshotManager/Screencap plugin. It is the
matching VideoExport companion for `KK_ScreenshotManagerReshadeOutpuit`; the
actual color/depth capture algorithm lives in the Screencap fork.

## What This Adds

- A `Normal + Depth` ScreenshotManager capture type in VideoExport.
- Per-frame color PNG plus depth sidecar output.
- Final delayed depth flush so the native D3D11 staging queue can finish after
  the last frame.

## Install

1. Install the matching Offline ReShade ScreenshotManager/Screencap fork first.
   VideoExport calls `Screencap.ScreenshotManager.ExportOfflineReShadeInputs`;
   without that method, `Normal + Depth` cannot work.
2. Before installing or testing VideoExport, press `LeftCtrl + F10` with the
   Screencap plugin alone and confirm that
   `UserData\cap\OfflineReShade\coloroutput.png`,
   `UserData\cap\OfflineReShade\depthoutput.rfloat`, and
   `UserData\cap\OfflineReShade\metadata.json` are generated correctly.
3. Install this fork's `VideoExport.dll` for KK or KKS into
   `BepInEx\plugins`.
4. In VideoExport, set ScreenshotManager capture type to `Normal + Depth`.

For KK, the Screencap-side `OfflineDepthD3D11Bridge.dll` must also be installed
next to `Screencap.dll` or configured through ScreenshotManager's
`Offline ReShade Export > D3D11 bridge DLL path`.

Use the same KK Screencap release variant that works for normal `LeftCtrl + F10`
captures:

- **KK Stable Path**: try first. It is lighter and fastest when the environment
  can keep D3D11 depth hooks stable.
- **KK Compatibility Bridge**: use if stable does not generate
  `depthoutput.rfloat`, only works for the first capture after restart, or
  intermittently misses depth during VideoExport.

Do not mix stable and compatibility DLLs. `Screencap.dll` and
`OfflineDepthD3D11Bridge.dll` must come from the same package.

KK `Normal + Depth` is most reliable from a Studio camera/lens view. If depth
sidecars are missing, switch from free view to a camera view, disable MSAA, and
disable Optimize in Background before recording again. KKS does not use the
native bridge and is unchanged by these KK-specific notes.

VideoExport uses ScreenshotManager's render screenshot settings for output
resolution. If a frame sequence has the wrong size, fix the ScreenshotManager
render resolution/downscaling settings first, then record again. Always inspect
one exported frame pair before sending the sequence to the Offline ReShade app.

## Output

VideoExport frame folders contain:

```text
UserData\VideoExport\Frames\<timestamp>\
  0.png
  0.depth.rfloat
  1.png
  1.depth.rfloat
  metadata.json
```

Depth files use the format reported by `metadata.json`. The current main path
for KK and KKS is:

- extension: `.rfloat`
- encoding: `rfloat32_device_depth_little_endian`
- row order: `bottom_to_top`
- value: D3D/Unity device depth in `0..1`

The actual depth capture algorithm lives in the Screencap fork. This plugin is
only the VideoExport integration layer.

## Performance Notes

`Normal + Depth` is slower than normal ScreenshotManager color capture because
each frame writes an additional full-resolution depth sidecar and, on KK, waits
for the D3D11 bridge to queue or flush delayed readbacks. The current bridge
uses a staging readback ring so most frames save depth asynchronously; the last
frame performs a blocking final flush.

---

# HSPlugins
A collection of useful studio plugins.

Make sure you download the version for your game (the first part before _ is the initials of the game, e.g. HS2 = HoneySelect2).

You can get the latest nightly builds of all plugins from the [CI workflow](https://github.com/IllusionMods/HSPlugins/actions/workflows/ci.yaml). Open the latest successful run and download the build from the Artifacts section.

Main changes in the fork:
- Added KKS support to some of the plugins
- Removed HS and IPA support to simplify the codebase
- Fixed compiling with VisualStudio, no longer require external scripts or dlls
- Code refactoring, created separate projects for each game
- Use BepInEx config instead of custom xml config files
- Changed default user content folders to be inside UserData
- Some new features like studio toolbar buttons

## How to use
1. Install latest BepInEx5 and BepisPlugins.
2. Download the latest release zip for your game from the releases page.
3. Pick the plugins that you want to install from the release zip and extract them into your game directory (the .dll files should end up inside the BepInEx\plugins folder).

To see hotkeys used to use the plugins check plugin settings or the config files in BepInEx\config.

## Credits
This is a fork of [HSPlugins](https://bitbucket.org/Joan6694/hsplugins/src/master/) by [Joan6694](https://joan6694.bitbucket.io/). Main reason for the fork is that Joan disappeared and the plugins needed to be ported to KKS. Support for games older than KK was dropped to simplify the codebase. Some no longer used plugins are not able to build and are excluded from the solution. Use the legacy sln to build those if needed.
