# In Falsus 3DMigoto 配置说明

本文档说明同目录下的 [`d3dx.ini`](./d3dx.ini) 中与 In Falsus 相关的设置、BGA 隐藏规则和 TOKONAGI 噪点 shader 修改。其余未提到的章节主要是随 3DMigoto 提供的通用配置模板，具体选项说明仍以 `d3dx.ini` 内的注释和 3DMigoto 文档为准。

## 运行环境

- 目标进程：`infalsus.exe`
- 注入模块：`3dmigoto.dll`
- Shader hash 模式：`3dmigoto`
- Shader 覆盖目录：`ShaderFixes`
- Regex shader 编译缓存：`ShaderCache`
- 默认 BGA skip 开关：`$bga = 1`（启动时启用）

请通过此目录中的 `3DMigoto Loader.exe` 启动或注入游戏，并确保它与 `3dmigoto.dll`、`d3dx.ini` 位于同一 loader 目录。不要把 `ShaderCache` 当作原始配置备份；需要备份的主要文件是 `d3dx.ini`、本说明和任何实际使用的 `ShaderFixes`。

## 常用按键

| 按键 | 功能 |
| --- | --- |
| `F7` | 开关 BGA skip 规则。默认开启；按下后 `$bga` 在 1 和 0 之间切换。 |
| `NUMPAD 0` | 开启或关闭 3DMigoto hunting overlay。当前 `hunting=2`，初始为软关闭。 |
| `F8` | 抓取下一帧的 3DMigoto FrameAnalysis。 |
| `F10` | 重新加载配置和 fixes。部分设置仍可能需要重启游戏。 |
| `F9` | 暂时显示原始画面，用于比较修改前后的效果。 |

Hunting overlay 打开后，可用 `NUMPAD 1/2` 切换 pixel shader，用 `NUMPAD 3` 标记当前 shader；按 `NUMPAD +` 结束 hunting 并恢复 shader。当前 `marking_mode=skip`，标记 shader 会暂时将其跳过，完成检查后应结束 hunting。

F8 FrameAnalysis 配置为导出 render target、贴图、constant buffer 文本和资源描述，并优先使用 mono 与 JPG/JPS 格式以控制文件大小。捕获结果通常写入 `FrameAnalysis-*` 目录。

## BGA skip 规则

所有下列 `ShaderOverrideBga*` 规则都受 `$bga` 控制：只有 `$bga == 1` 时才执行 `handling = skip`。因此 F7 可在隐藏这些元素与恢复它们之间切换。

当前配置共有 16 条 skip 规则：

| 配置名称 | 3DMigoto shader hash |
| --- | --- |
| `ShaderOverrideBgaLane` | `4fd89bf240d522e3` |
| `ShaderOverrideBgaNotesHud` | `316ed1fdc61b00d1` |
| `ShaderOverrideBgaNoteA` | `a442298b6f4226c0` |
| `ShaderOverrideBgaNoteB` | `6af59731df759ef0` |
| `ShaderOverrideBgaText` | `7590e112dc23d2cc` |
| `ShaderOverrideBgaLaneTex` | `d7ddadc18c8d77d8` |
| `ShaderOverrideBgaBeam` | `149b80b4e999b262` |
| `ShaderOverrideBgaBarLetters` | `6eebb195b8e1a45c` |
| `ShaderOverrideBgaLines3` | `4df9e103f2e35029` |
| `ShaderOverrideBgaCentreStage` | `3ea88fb8034b707e` |
| `ShaderOverrideBgaBands` | `b3a392045985a973` |
| `ShaderOverrideBgaLines5` | `81ca1da9f42bf884` |
| `ShaderOverrideBgaSkyNoteA` | `517e1b5a21a44c05` |
| `ShaderOverrideBgaSkyNoteB` | `6dda9a88068c9c42` |
| `ShaderOverrideBgaSkyNoteBody` | `9b90b45b412d4907` |
| `ShaderOverrideBgaTokonagiHexAtlas` | `f73b4e715a508828` |

The TOKONAGI noise regex rules below are separate from the BGA skip switch. F7 does not disable or enable these shader edits.

## TOKONAGI 噪点区域规则

这些规则旨在扩大噪点/干扰效果的空间覆盖范围，同时保留 shader 原本的随机噪点、贴图采样和强度计算。每个 RenderDoc capture 中的 EID 是该次抓帧的事件编号；不同 capture 的 EID 不可直接当作固定 shader ID。

| 3DMigoto pixel shader hash | 对应抓帧 / RenderDoc 事件 | 规则行为 |
| --- | --- | --- |
| `e58392c134a8e0a7` | TOKONAGI capture；此前对应的候选 noise pass | 将最终空间遮罩值设为 1，保留遮罩平滑计算之前的原始强度与其余效果。 |
| `b3373a0f3d3d9ecd` | `TOKONAGI-frame47903.rdc`，EID 1936 | 用原有随机因子 `r2.w` 替代位置区域因子。 |
| `2643171e8ad9cbdc` | 最新 `TOKONAGI.rdc`，EID 1929（draw 145） | 对应另一寄存器分配的 shader 变体；用原有随机因子 `r3.x` 替代位置区域因子。 |

RenderDoc EID 1936 和 EID 1929 的 pass 都读取主画面以及 `256×256` 贴图，并把结果写入新的全尺寸 render target。对应的 3DMigoto hash 通过抓帧中的输入纹理、输出和 shader 运算序列对照确认。

### 匹配方式与适用范围

这几段使用 3DMigoto `ShaderRegex`，通过 pixel shader 汇编中的指令序列匹配，而不是在 regex section 中直接按 hash 限定。文档中列出的 hash 是抓帧分析得到的对应 shader；regex 实际是否应用，取决于目标 shader 汇编是否包含相应序列。因此，游戏或 shader 编译器更新后应重新抓帧并检查匹配结果。

这些修改在配置中没有绑定 F7，也没有设置专用启用键。只要 3DMigoto 加载 `d3dx.ini` 且 shader 指令匹配，patch 就会自动应用。完成配置修改后可用 F10 重载；若要确认视觉效果，仍需在游戏中重新录制并检查。

## 分析与备份说明

- `FrameAnalysis-*` 是 3DMigoto 抓帧输出，可用来核对 draw 顺序、输入贴图、constant buffer 与 shader hash；单独备份时体积可能较大。
- RenderDoc `.rdc` 文件是独立的 capture。EID 只有结合相应的 `.rdc` 才有意义。
- `ShaderCache` 中的文件由 3DMigoto 自动生成或缓存；本目录中的 `d3dx.ini` 才是 regex 规则的配置来源。
- 当前已完成静态抓帧对应与配置规则记录；噪点覆盖扩展后的最终视觉效果仍需游戏内核验。
