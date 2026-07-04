# KK/KKS VideoExport ReShade Output 中文使用说明

> 这是使用者说明，只覆盖安装、使用和排错。开发、编译和实现细节请看英文 README。

这个仓库发布的是 `KK_ScreenshotManagerReshadeOutpuit` 的配套 VideoExport 插件。它本身不负责实现深度捕获算法，只是在 VideoExport 中增加 `Normal + Depth` 截图类型，并调用修改版 Screencap/ScreenshotManager 导出每一帧的 color/depth。

## 使用前提

必须先安装并验证修改版 Screencap/ScreenshotManager：

- KK/KKS Screencap 插件必须提供 `Screencap.ScreenshotManager.ExportOfflineReShadeInputs`。
- KK 需要安装匹配的 `OfflineDepthD3D11Bridge.dll`。
- KKS 不需要 native bridge。

安装 VideoExport 之前，请先只用 Screencap 做一次测试：

1. 进入 Studio。
2. 按 `LeftCtrl + F10`。
3. 确认生成：

```text
UserData\cap\OfflineReShade\coloroutput.png
UserData\cap\OfflineReShade\depthoutput.rfloat
UserData\cap\OfflineReShade\metadata.json
```

如果这一步失败，不要先调 VideoExport，也不要先进入 Offline ReShade app。必须先修好 Screencap 侧导出。

## 安装

1. 安装匹配的 Offline ReShade Screencap/ScreenshotManager fork。
2. 确认 `LeftCtrl + F10` 已经能正确生成 color/depth。
3. 把本仓库对应游戏版本的 `VideoExport.dll` 放入 `BepInEx\plugins`。
4. 打开 VideoExport。
5. 将 ScreenshotManager capture type 设置为：

```text
Normal + Depth
```

## KK 版本选择

VideoExport 使用 Screencap 侧已经安装的 KK 发布版本。

- **KK Stable Path**
  - 优先尝试。
  - 性能更轻。
  - 如果你的环境能稳定生成深度，使用这个版本即可。
- **KK Compatibility Bridge**
  - 如果 stable 不生成 `depthoutput.rfloat`、重启后只有第一张有深度，或 VideoExport 录制时偶发缺 depth，请改用这个版本。

不要混用 stable 和 compatibility 的 DLL。`Screencap.dll` 和 `OfflineDepthD3D11Bridge.dll` 必须来自同一个发布包。

KK 的 `Normal + Depth` 最推荐使用 Studio 镜头/相机视角。如果缺少 depth sidecar，请尝试：

- 从自由视角切换到 Studio 镜头/相机视角；
- 关闭 MSAA；
- 关闭 Optimize in Background；
- 从 stable path 切换到 compatibility bridge。

KKS 不使用 KK 的 native bridge，上述 KK 限制不适用于 KKS。

## 分辨率设置

VideoExport 的帧尺寸来自 ScreenshotManager 的 render screenshot 设置。

如果输出帧分辨率不对，或 color/depth 对不上，请先检查 Screencap/ScreenshotManager：

- render resolution 是否是目标尺寸；
- downscaling/supersampling 是否符合预期；
- `metadata.json` 中的 `width`、`height`、`depthWidth`、`depthHeight` 是否正确。

建议每次修改分辨率后先录一小段，抽查一帧 color/depth，再进行正式录制。

## 输出文件

VideoExport 帧目录会包含：

```text
UserData\VideoExport\Frames\<timestamp>\
  0.png
  0.depth.rfloat
  1.png
  1.depth.rfloat
  metadata.json
```

当前发布版主格式：

- 扩展名：`.rfloat`
- 编码：`rfloat32_device_depth_little_endian`
- 行顺序：`bottom_to_top`
- 数值：D3D/Unity device depth，范围 `0..1`

具体深度捕获算法由 Screencap fork 决定。本插件只是 VideoExport 集成层。

## 性能说明

`Normal + Depth` 一定会比普通 VideoExport 慢，因为每一帧除了 color PNG 之外，还要输出一份 full-resolution depth sidecar。

KK 还需要通过 D3D11 bridge 排队或 flush 深度读回。当前实现使用 staging readback ring，大多数帧可以延迟读回，最后一帧会做一次阻塞 flush，避免尾帧缺 depth。

正常录制时请关闭 Screencap 里的 `D3D11 bridge candidate diagnostics`，它只用于排查问题。

## 验证流程

正式送到 Offline ReShade app 前，建议这样检查：

1. 用 Screencap 单独 `LeftCtrl + F10` 确认 color/depth 正常。
2. VideoExport 录制 5 到 10 帧测试片段。
3. 打开帧目录，确认每个 color PNG 都有同名 `.depth.rfloat`。
4. 抽查至少一帧，确认 depth 可视化后不是空图、不是错分辨率图。
5. 确认通过后，再录制正式序列并进入 app 侧处理。
