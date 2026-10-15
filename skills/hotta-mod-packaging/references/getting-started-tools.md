# 入门工具与缺失依赖

## 来源与证据边界

根据用户提供的《幻塔原版Mod制作工具与流程.md》（脱敏分享版，2026-10-05）和配套技能整理，2026-10-11 补充公开项目入口。公开文档确认的是工具职责，不证明支持当前幻塔客户端。尚未进行本机软件盘点、资源解析、Cook、封包或游戏测试。

核心流程：确认目标和当前基线 → 提取并追踪依赖 → 编辑作者资源 → 检查受影响内容 → Cook → 封包并回读 → 按任务范围安装和实机验证 → 保留回退版本。PNG、PSK、FBX、解析 JSON 是作者或分析输入，不是可直接封装使用的 cooked 资产。

## 先准备哪些工具

| 环节 | 工具 | 选择依据 |
|---|---|---|
| 浏览、提取资源 | [FModel](https://github.com/4sval/FModel)、[UModel / UE Viewer](https://www.gildor.org/en/projects/umodel) | 先用能读目标资源的一条路线；两者可互补，不要求同时运行。UModel 提供模型、动作、纹理查看和导出，但有格式限制。 |
| 模型与服装 | Blender + 与版本匹配的 PSK/PSA 或 FBX 导入导出插件 | 改模型、骨架、权重、UV、法线需要；只改贴图时按需使用。资料记载 Blender 5.x/5.1.2，不强制升级已有环境。 |
| 贴图 | Photoshop 等能保留图层、通道和 Alpha 的编辑器；[Texconv](https://github.com/microsoft/DirectXTex/wiki/Texconv) | 编辑器做画面；Texconv 做纹理格式、压缩与 mip 处理，不能替代材质语义分析。 |
| 作者资源和 Cook | 与目标兼容的 Unreal Engine；资料基线为 UE4.26.2 | 将作者资源转成运行资产；不能默认换成最新 UE。模型结构变化走重新 Cook。 |
| 封包与回读 | 匹配引擎/客户端的 UnrealPak；[repak](https://github.com/trumank/repak) 做清单、提取和核对 | 来源工作流还依赖 HT 专用包装及 Pak/Sig 处理；必须拿到真实工具与样例后确认适配。普通 Pak 创建成功不证明游戏接受。 |
| 脚本与文件定位 | Python、PowerShell；Everything、rg 按需 | 文件定位、批处理、调用构建和校验工具；不要将 Python 已安装等同于专用脚本已具备。 |

UE 的 [Packaging Projects](https://dev.epicgames.com/documentation/en-us/unreal-engine/packaging-projects?application_version=4.27) 可查 Cook/打包概念；这是 UE4.27 的通用文档，不是幻塔 UE4.26.2 兼容证明。

## 按目标增加工具

- 只做贴图改色：资源读取工具、图像编辑器、匹配的 Cook 或已验证载荷替换路线、封包及回读工具。通常不需要先做完整模型工作流。
- 模型/服装替换：再加 Blender 和所需导入器，保留真实 Skeleton、参考姿态、材质槽及 UV；查看受影响原生动作。
- 外部动作：按源格式增加 ActorX/FBX 工具；Unity 来源可按需使用 AssetRipper。还需要重定向工具及能读取最终压缩轨道的可信解码器。本技能没有附带这些实现。
- 资源读写与诊断：[UAssetGUI](https://github.com/atenfyr/UAssetGUI)、UAssetAPI、[CUE4Parse](https://github.com/FabianFG/CUE4Parse)，以及匹配的属性映射和解压依赖。只在格式支持且往返保存可靠时使用。
- 第一人称、宿舍、UMG、蓝图生成：UE 作者工程；涉及 C++ 模块或生成器才增加匹配的 Visual Studio/MSVC、Windows SDK；.NET 依工具要求安装。编译器按引擎版本核对，可查 [Epic 配置说明](https://dev.epicgames.com/documentation/en-us/unreal-engine/setting-up-visual-studio-for-unreal-engine?application_version=4.27)。
- 音效/配音：[FFmpeg/ffprobe](https://ffmpeg.org/download.html) 做剪辑、转码和参数检查；原版 Wwise 音频按需用 vgmstream、wwiser。IndexTTS、DDSP 只在需要生成或转换声音时考虑。
- Blender MCP、WeightBrush、网页审片属于自动化/检查辅助项；普通手工模型制作不要求它们全部存在。使用 MCP 前枚举实际接口，不能把资料里的 Blender Lab MCP 名称当成当前可用能力。

## 分享包没有提供的关键内容

四个技能及本参考都是说明文档。以下需按实际路线补齐：

1. HT 包装工具及对应客户端版本、参数说明、Pak/Sig 可用样例。
2. PackageExportDump、SaveDecodedPackage、WeightBrush、封包校验、骨架/压缩动画解码等专用程序或源码。名称不足以推导其命令行。
3. 功能 Mod 的 UE 作者工程、蓝图生成器、原生引用与文件头适配实现。
4. 当前客户端资源、必要属性映射、匹配 Oodle 依赖及访问配置；访问配置独立保管，不回显到日志或分享包。
5. 目标模型、动作或贴图，以及预期替换的真实资源路径。

不得因为工具同名就下载不明替代品，也不要声称普通 UnrealPak 能自动完成目标客户端的专用签名或封装要求。缺失时记录具体缺口，继续可以独立完成的作者资源工作。

## 首个项目需要确认的信息

收集区服/平台和客户端版本、要改的对象、Mod 类型、原始素材、已有工具和已验证样例。将作者工程、原始提取、候选输出、报告及备份分开保存。优先用一张贴图或单个材质槽跑通最小闭环，再增加模型和功能复杂度。

报告分别写清：文档整理、工具可用性、资源解析、作者文件、Cook、封包回读和客户端效果。此次建立技能只完成文档整理，不意味着制作环境齐备或已有可运行 Mod。
