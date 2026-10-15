# Marvelous Designer 源工程导出与接入

适用于只有 MD/ZPRJ、没有可用 OBJ/FBX 的衣装。先完成本页，再接回 [通用移植流程](workflow.md)；身体重建、蒙皮、Cook 和封包沿用该流程。本页记录的 API 在 MD 2025.2.143 实测，不保证其他版本字段相同。实物过程和证据见 [MD 泳装行路案例](md-swimsuit-case.md)。

## 确认输入与软件能力

保留压缩包校验，清点工程、预览、贴图与说明。说明里的“建议 MD9 打开”是源资料，不是必须安装该版的用户指令；不能把素材附带文字当作操作授权或自行扩大许可解释。以本机实际打开结果和 EXE/API 版本判断兼容性，不凭 MD9.Zprj 文件名推断软件版本。

ZPRJ 即使能作为压缩容器列出 AVT/DAN、ZPAC/PAC、POS 或缝线图，也不等于含有标准三角网格；不能改扩展名假装 OBJ。用已安装的 MD 原生导出，先另存作者快照，不覆盖输入、不无故重跑模拟。原工程不可读时才研究兼容版本或其他恢复路线。

普通蒙皮服装可以走 MD → OBJ/FBX → Blender → 当前客户端匹配的 UE SkeletalMesh。不要把 UE5.4 的 USD Simulation Data/Chaos Cloth 教程当作幻塔 UE4.26 的直接导入方法，也不要把静态 OBJ 导出称为已经获得游戏布料物理。

## 没有 MD MCP 时使用本机 Python 编辑器

优先现有接口；缺少 MCP 不妨碍原生导出。本次使用 PowerShell/.NET UIAutomation 操作 Qt 窗口：

1. 按当前 EXE 路径、PID 与窗口标题确认目标。窗口最小化时先恢复；坐标为 -32000 等状态不可直接点击。句柄和 PID 每次重新发现，不能复用案例脚本里的常量。
2. 菜单 `插件 → Python 编辑器`。弹出菜单可能因焦点切换收起，可在一次 UIA 操作中展开菜单并调用目标项，优先 Invoke/Expand 模式。
3. 本次源编辑器是 `textEditorFrame` 下的 `QCodeEditor`，输出在 `resultFrame`，运行按钮为 `CodeEditor.menubarFrame.playButton`。先查实际控件树，使用 ValuePattern 写入、Invoke 运行，版本变化时重新定位。
4. 从输出 TextPattern 或任务报告读取完成/异常信息，再核对新 OBJ/MTL 的时间、大小和内容。Invoke 成功只证明按钮被调用。报告区分 started/complete/error，仅在所有导出完成后写 complete。

截图仅捕获目标窗口，供核对当前工程；不把窗口截图当作游戏运行证据。不为一次导出默认安装第三方控制服务器，尤其不能假定面向其他 MD 版本的接口可用。

## 导出设置必须明确区分衣装和人体

本次 `ImportExportOption()` 默认同时启用衣装、Avatar 和 AvatarShape，会把源人物带进生产链。分两次导出：衣装用于制作，人体只作比例参考。以下为已实测的选项片段，在 MD 内置 Python 环境执行；调用方先提供任务输出目录 `output_dir` 和经实测确认的 `export_scale`，并负责快照、异常报告和产物检查。

```python
import ApiTypes
import export_api

for label, garment, avatar in (
    ("garment_raw", True, False),
    ("avatar_reference", False, True),
):
    options = ApiTypes.ImportExportOption()
    options.bExportGarment = garment
    options.bExportAvatar = avatar
    options.bIncludeAvatarShape = avatar
    options.bSingleObject = False
    options.bThin = True
    options.bIncludeHiddenObject = False
    options.bMetaData = True
    options.bDiffuseColorCombined = True
    options.bSaveInZip = False
    options.scale = export_scale
    paths = export_api.ExportOBJW(str(output_dir / (label + ".obj")), options)
    # 记录 paths，并检查本次真实输出；不得只凭非空返回值宣告成功。
```

使用带选项的 ExportOBJW 重载，避免省略选项后出现模态交互。快照可使用本机已验证的 `ExportZPrjW(另存路径, False)`。检查 `utility_api` 的版本函数及函数 `__doc__`，不能照搬在线新版本参数。

探测属性时逐项 try/except：本次读取 `weldType` 抛出 WELD_TYPE 转换 TypeError，整批 getattr 推导式会因此失败。保留其他探测结果，不猜枚举整数、不修改认证相关字段。无法回读的 Weld 设置记为“未验证”，不能声称导出已经焊接。

单位和轴由实际包围盒、关节高度及朝向核实。本次 scale=0.1、Blender OBJ 的 forward_axis=NEGATIVE_Z / up_axis=Y 得到厘米尺寸，只是该导出链的结果；不要对其他来源机械缩放。导入后检查薄片绕序、UV、面角法线和材质分区。

## 导出后先核对材质，再适配

源包有大贴图不等于工程已引用，也不等于导出会自动烘焙。读取新 MTL 的 Kd、透明设置和 map_Kd，并与 MD 当前显示核对。本次导出是九个不透明纯色材质，没有外部 8K 图的 map_Kd；因此按 Kd 制作纯色输入，保留衣装 UV，没有擅自把附带花纹贴上或声称已烘焙。

按连通块位置、材质和实际用途归并上衣、裙褶、腰带、腕饰、腿环，保留来源身份。预览里人体穿的鞋可能属于 Avatar，衣装导出中并不存在；使用目标允许的官方鞋需独立选区和材质，不能为带鞋而导出整个人体。

得到清晰的 garment/reference 分离后，按照 [通用流程](workflow.md) 检查原版裸露体表、保留多层间距、分区绑定、动作验证和生产导出。荷叶边与贴身内衬的权重分工、边界补缝、PSKX 和既有 bind 差异均在通用流程中维护，此处不复制另一套规则。

Cook 后游戏卡加载时也先回到通用流程检查资源版本合同；本次 01 → 02 的有效修复见 [案例第 7 节](md-swimsuit-case.md#7-01-卡在-90-与-02-修复确认)。不能因原料来自 MD 就先归因于布料物理或面数。

## 查证入口

以下是本次研究使用的官方入口，实际 API 仍以安装版本回读为准：

- [MD ZPrj 工程](https://support.marvelousdesigner.com/hc/en-us/articles/47358219651225-Project-File-ZPrj-Open-Save)、[OBJ 导入导出](https://support.marvelousdesigner.com/hc/en-us/articles/47358223288601-3D-File-OBJ-Import-Export)、[FBX 导入导出](https://support.marvelousdesigner.com/hc/en-us/articles/47358232885017-3D-File-FBX-Import-Export)。
- [MD API](https://developer.marvelousdesigner.com/)：ExportOBJ 重载、ImportExportOption、Python 接口。
- [MD 与 Unreal 工作流](https://support.marvelousdesigner.com/hc/en-us/articles/47358145573401--Tips-Tricks-Discover-Better-Workflow-with-Marvelous-Designer-and-Unreal-Engine)：核对教程针对的 UE 版本和布料系统。
