# 1. 能力、架构与质量对比

对比对象是 2026-11-06 读到的 fork 树，SHA 见 [README](README.md)。结论来自源码、测试、CI 工作流和验收文档，不来自项目宣传页的转述。Millwright 自己的 `docs/COMPARE-solidpilot.md` 只作为该作者在另一提交上的意见，不作为本矩阵的证据。

## 1.1 总表

| 项目 | 形态 | COM 执行 | 工具面（按代码计数） | 验证 | 审计 / 会话 | 审批 | 真机证据 |
|---|---|---|---|---|---|---|---|
| 本仓库 Skill / MCP / CAD Studio | Skill、stdio/HTTP MCP、Tauri 桌面，共用 `capabilities.yaml` | Python `pywin32` / `comtypes` 进程外自动化；C# PIA 与进程内 Add-in 为路由后端。连接在调用线程 `GetActiveObject`，没有把全部 COM 封送到一条长驻 STA 队列 | `mcp-server/server.py` 在 `9ba848e` 有 **67** 处 `@mcp.tool`，且都能解析出 `name=`。此前写成 66，漏了同一行声明的 `cadstudio_read_artifact`。`docs/architecture/execution-core.md` 仍写 61，文档落后于代码 | 复核是门禁。`VerificationStatus` 聚合后，WARN/FAIL/BLOCKED 和待人工复核不能进入 completed。产物台账记录存在性、大小、SHA-256、`producedThisRun` | Trace 为 `queue/events/*.jsonl`。`scripts/core/execution.py` 的 `_trace` 捕获异常后继续执行。批准账本在本地 JSON | CAD Studio Policy Gate：危险能力进入 `approval_required`，批准原因变化会重新拦截。MCP `approval.py` 是逐次、带摘要和有效期的本地批准；工具参数里的 `approved=true` 无效 | **E1** 部分能力。安装板、夹爪等有入库验收记录。多数能力在清单里是 `pilot` |
| SolidPilot | MCP（Python FastMCP）→ REST → C# .NET Framework 4.8 | `StaExecutor`：一条后台 STA 线程串行全部 COM，包括 `GetActiveObject`。`OperationGuard` 用 `operation_id` 幂等和 `state_version` 乐观并发 | `tool-schemas.json` **46** 个名字。其中 `get_recipe`、`save_analysis`、`rebuild_from_ir`、`submit_feature_graph`、`compare_parts`、`compare_assemblies` 是适配器编排，不进 C# `ToolController` | 拉伸成功路径在特征对象非空时把响应 `Verified` 设为 true（`SolidWorksService.Features.cs` 的 `FeatureExtrusion3` 分支）。README 对 `compare_parts` 的口径才是拓扑一致且体积、面积差不超过 1%。编译器测试用假端口 | `ExecLog` 写 exe 旁的 `execution.log`，失败被吞掉。日志发生在执行过程中，失败不阻止 COM | 未见用户审批门禁 | **E3**。README 与代码注释写 “proven live” / “working prototype”。自动化测试是离线契约和编译器套件。README 指向的 `logs-ir.md` 不在该树中。无 `.github` CI。Appwomb fork 停在 `f9546a9`（2026-07-21）；实现前和上游默认分支重新 diff，不以这一份树代表上游现状 |
| SolidworksMCP-TS | Node MCP + `winax` | 直接 COM；参数多的调用改走生成的 VBA，`RunMacro2` 执行。复杂度分析器把 13 个及以上参数标为宏回退 | `src/index.ts` 实际拼进 `allTools` 的模块工具 **58** 个，另加 5 个内联工具（宏录制 3、设计表 2）；PDM 工具默认不注册。另有约 10 个工具文件没有被 `index.ts` 导入 | 项目自己的 `TESTING.md` 写明：mock 通过不能说明真机可用；集成测试没有进自动 CI | 有 winston 日志、可选 SQL、宏录制状态文件。未见“先记审计再写 CAD” | 未见分级审批 | **E4 / E5**。README 自标 Alpha，并写明多数工具没有对真机验证。`createExtrude` 与 `RunMacro2` 的最近提交说明 2024+ 上修过这一条路径，这条修复本身是真机相关提交，不能外推到全部工具 |
| Solidworks-MCP（alisamsam） | Python MCP。`server.py` 注册 **25** 个工具；README 表格仍写 22 | MCP 入口是 `SolidWorksAutomation._try_connect_com`（`automation/base.py`）：当前线程 `CoInitialize` 后依次尝试 `GetObject`、`Dispatch`、`dynamic.Dispatch`、`GetActiveObject`。`solidworks_com.py` 的 `SolidWorksCOM.connect` 才是另一条路径：`GetObject` 失败后再 `Dispatch`，没有 `GetActiveObject`，也没有 dynamic Dispatch；它被 `solidworks_simulation.py` 使用，不注册这 25 个工具。两条路径都可能用 `Dispatch` 启动新实例，都没有专用 STA 队列 | README 漏记的三个名字是 `get_document_info`、`create_sketch_on_face`、`get_sketch_status`。含 `execute_python`，用 `exec` 在带有 SolidWorks 对象的全局环境里跑任意代码 | 特征失败有回退链（`FeatureExtrusion3` → `FeatureExtrusion2` → `FeatureBossSimple`）。成功返回不要求体积或尺寸回读 | logging 模块。仓库里留有 `backup_20260209_*` 源码副本，不是运行时备份 | 无 | **E4**。README 写 “production-ready”。树内没有可运行的真机测试套件 |
| SolidworksMCP-python | Python FastMCP，可选 UI / agent | `ComExecutor`：专用线程上 `CoInitialize`，所有 COM callable 进队列。断路器包在适配器外 | `src/solidworks_mcp/tools/` 下 **154** 次 `@mcp.tool`。与 README “Discovered 154 tools” 一致。含 10 个 VBA 生成工具和宏录制/执行 | `verify_model` 按期望体积、质量、包围盒、特征数、配合数给出 verdict。工具文档写明：不匹配时工具 `status` 仍可以是 `success`。部分建模工具附带 `measured_change` | SQLite（SQLModel）：`DesignSession`、`PlanCheckpoint`、`ToolCallRecord`、`SoCCheckpoint`、`ModelStateSnapshot`。SolidWorks-as-Code 可导出 Python。默认关闭。`_soc_log` 在调用**之后**写，异常只打 debug | `PlanCheckpoint.approved_by_user` 是仪表字段。`execute_next_checkpoint` 执行时把该字段写成 `True`，执行前不检查它是否已由人批准 | **E2 + E3**。`tests/live/` 有手算断言（例如 100×50×10、四孔板体积 49049.7 mm³），仅当 `SOLIDWORKS_MCP_RUN_REAL_INTEGRATION=1` 才收集。默认 CI 是 Ubuntu `make test`。`.github/workflows/integration-tests.yml` 是 `npm ci` / `npm run build`，该树没有 `package.json`，不能把它当作 Python 真机 CI。README GIF 是作者声称的录屏 |
| SolidWorks-MCP-1（hjbaard） | Python MCP，stdio，不主动联网 | `ComWorker`：一条名为 `solidworks-com` 的 STA 线程；`CoInitialize` 一次；队列一次一个任务。等待超时默认 120 秒，超时后 COM 调用本身停不下来。不泵消息，因此不能在该线程上注册事件 sink | `server.py` 中 **88** 个 `@mcp.tool`。类型化参数，没有任意代码工具 | 建模返回质量属性。`set_dimension` 写 `SystemValue` 后回读，偏差超过阈值则写回原值并报错。零件夹具在结束时断言草图完全定义。集成测试把体积或装配位姿与手算比较 | 无持久审计账本。有 `--selftest` 和分阶段脚本 | 无用户审批。失败即 `{ok:false}` | **E2**。`tests/` 有 30 个文件、519 个 `def test_`；其中 17 个文件整文件标记 `solidworks`，约 316 个测试函数。`conftest.py` 在连不上 SolidWorks 时 skip。CI `publish.yml` 在 `windows-latest` 上只跑 `pytest -m "not solidworks"`，步骤名写明 runner 没有 SolidWorks。README 写 733 tests、测试于 SOLIDWORKS 2026；733 与源码中的 `def test_` 数量不一致，本次没有在 Windows 上 `pytest --collect-only` |
| Millwright | Electron 28 桌面 + Python sidecar | 当前主路径是 sidecar 工具调用。`sw_agent/com_executor.py`（P122）把 COM 收到一条 STA 线程。`docs/ARCHITECTURE.md`（标注 2026-04-29）仍把主路径写成生成 VBA/Python 再执行，与当前 sidecar 代码不一致 | sidecar `@tool(` **87** 处。README 徽章仍写约 50 个原生工具，少于代码。含 `run_shell` | `verify.py` 对体积增减、组件数、配合是否移动做快照。文件头写明：验证是证据，不是门禁 | 会话 JSONL，默认开，写失败被吞掉，可用环境变量关闭。可导出重放步骤 | `strict` / `normal` / `permissive` / `auto`。`auto` 不确认。破坏性工具第一次执行前做懒备份；备份异常只发事件，不阻止调用 | **E4** 为主。Python 测试覆盖验证分类、可靠性逻辑；未见入库的 SolidWorks 真机验收记录。作者在对比文里承认执行层可信度是短板，该文日期早于本 SHA |
| MoAspen | 任务说明：自然语言 / 多模型 → 工具与审批 → Windows COM → 回读，会话，`mcp_audit.jsonl`，Tauri | E0 | E0 | E0 | E0。任务要求 MoSolid 沿用这套形状 | E0 | E0 |

## 1.2 本仓库能力等级

`capabilities.yaml` 是 Skill、MCP、队列和 UI 的能力真源。本次解析到 37 条带 `level` 的能力：

| 等级 | 数量 | 条目 |
|---|---|---|
| `verified` | 8 | `requirements_planning`，`solidworks_addin_host`（2026），`part_and_features`（2024、2026），`holes_and_finishing`（2024、2026），`assembly_and_mates`（清单写 2024），`drawing_dimension_insertion`（2024、2026），`export_delivery`（2024、2026），`geometry_and_delivery_review` |
| `pilot` | 28 | 其余已实现但限制写明需人工复核的能力，包括工程图总包、Pack and Go、钣金、焊件、仿真、齿轮、配置、运动算例 |
| `not_implemented` | 1 | `autocad_basic_drafting` |

`verified` 的含义是清单维护者把该能力标成可无人值守交付的候选，同时许多路径仍要求 review。它不等于“所有参数组合都在所有 SolidWorks 版本上通过”。例如 `assembly_and_mates` 的 `verified_versions` 只有 2024；2026 上的夹爪验收记录在 `docs/architecture/`，不能从清单字段自动升成全装配 `verified`。

已入库、本次未重跑的真机材料包括：

- `golden-workflows.yaml` 的 `mounting_bracket.real_acceptance_2026`：120×80×10 mm，四孔 Ø7，中心 (±48, ±28)，材料 6061-T6，长度容差 0.01 mm，体积容差 0.05 mm³。
- `docs/architecture/real-golden-workflow-acceptance.md`：安装板验收要求从 B-Rep 回读一个实体、六个板面、四个内圆柱面，并核对解析体积与材料条目。
- 夹爪结构、保持行程的验收文档与 `tests/solidworks_golden_*_2026.py`。这些脚本在本环境不能运行。
- `docs/architecture/evidence/` 下的红acted 套件摘要。文档自己写明：一台 SolidWorks 2026 通过不等于发布通过。

Execution Core 的 43 个 deterministic scenario 使用 Fake Handler，证明的是状态机和门禁，不证明 CAD。

## 1.3 分层能力

下表的“有代码”表示该层存在可调用实现。“真机”使用 1.1 的等级，取该层里证据最强的部分，不把一个工具的通过外推到整层。

| 层 | 本仓库 | SolidPilot | TS | alisamsam | python | hjbaard | Millwright |
|---|---|---|---|---|---|---|---|
| 零件草图与拉伸 | 有代码，E1（安装板路径） | 有代码，E3 | 有代码，E4；拉伸宏回退有一条 2024+ 修复提交 | 有代码，E4；圆角依赖先选边或 `execute_python` | 有代码，E2（live 套件里的板） | 有代码，E2（手算体积测试，CI 不跑） | 有代码，E4 |
| 孔 / 槽 / 螺纹 | 孔与收尾 `verified`；真实螺纹在子技能，文档写重建后回读 | 孔主要靠草图切除；Hole Wizard / 真实螺纹不在 46 工具名里 | 草图圆与拉伸，无独立孔向导工具名 | 无独立孔向导 | 有切除与工程图孔标注路径，E2 覆盖四孔板 | 有盲孔、沉孔、Hole Wizard、ISO 螺纹工具，E2 | 有切除，未见 Hole Wizard 工具名 |
| 圆角 / 倒角 | 有 MCP 与 CNC 子技能 | `add_edge_feature`，E3 | 未见与零件圆角对等的已注册工具名 | 有，选边薄弱 | 有 | 有，含全圆角与可变半径描述 | 有 |
| 改驱动尺寸 | `solidworks_update_dimension`，能力等级 `pilot` | `modify_dimension`，E3 | `set_dimension`，E4 | 无独立工具 | `set_dimension`，E2 与否取决于 live 套件是否覆盖该调用 | `set_dimension` 回读并回滚，E2 | `modify_dimension`，验证是证据不是门禁 |
| 装配与配合 | 有，夹爪路径 E1；清单版本字段仍是 2024 | 插入、索引选实体配合、装配分析，E3 | 装配 VBA 模块**未**被 `index.ts` 注册，E5 | 只能新建装配文档 | 有插入与配合，README 演示两板配合，E3（GIF）/ E2（若 live demo 被收集） | 有配合、干涉、运动范围检查，E2 | 有插入、配合、干涉 |
| 干涉 | `solidworks_interference_check` | 装配分析含质量属性；干涉不是独立工具名 | `check_interference` 已注册，E4 | 无 | `check_interference` | `check_interference`，E2 | `check_interference` |
| 工程图 | 出图与尺寸插入有 `verified` 记录；GB/T 总图、GD&T 仍是 `pilot` 且要求目视 | 视图、模型项尺寸、中心标记、孔标注、剖视、展开视图、图纸读回，E3。作者写自动尺寸会漏标或标到隐藏边 | 5 个已注册图纸工具，E4。增强图纸与图纸分析模块未注册，E5 | 无 | 工具多，live demo 检查 A3、尺寸、中心标记、BOM，E2 条件执行 | `make_drawing`，E2 范围小于 python 的图纸包 | 视图、剖视、模型尺寸、BOM、注释 |
| 导出 STEP / PDF / 图 | `export_delivery` 为 `verified`（2024、2026）。交付门禁拒绝本轮未变化的旧文件 | STEP/IGES/STL/PDF/DWG/DXF。导出不增加 `state_version` | 有导出与截图工具，E4 | 无导出工具 | 有 STEP/IGES/STL/PDF/DWG 与图像 | `export` 含 STEP/STL/3MF 与截图，E2 | `export_file` / `export_stl` |
| 质量 / 体积 / 特征树查询 | 有质量属性、包围盒、边列表、审查 | `analyze_model` 模式包括几何、质量、实体、特征、边、面、草图、feature_map | 质量、包围盒、干涉、测距 | `list_features` | 质量、特征树、`verify_model` | 每次建模回质量属性；另有面、边、尺寸、特征列表 | 质量、包围盒、面、边、选择 |
| 特征 IR / 确定性编译 | 无 CAD 中性特征图。VibeCAD 产出的是设计计划，不是可编译 IR | 有。`pycompiler` 离线测试 E4；作者称反向重建在真机上得到 `verified`，E3。参考解析 v0 不能跨编辑存活 | 无 | 无 | 有草图图快照表，不是特征编译器 | 无 | 无 |
| 多模型桌面 | Tauri CAD Studio：Codex / Claude Code / Gemini CLI / OpenCode 桥，Policy Gate，产物台账 | 无桌面。适配器目录只有 Claude | 面向 Claude Desktop 配置 | 面向 Claude | 可选本地 UI 与 LLM 路由 | 无桌面，任何 MCP 客户端 | Electron：Anthropic 与 OpenAI 兼容协议、视觉模型、确认卡 |
| 弹窗 | `solidworks_recover` 能枚举模态窗。`dismiss_dialogs` 默认关闭，因为 `WM_CLOSE` 可能关掉不该关的对话框 | 标注尺寸前临时关闭 `swInputDimValOnCreate`，之后恢复。README 承认分析可能停在等待用户点击 | 无等价恢复工具 | 无 | live 注释记录无界面窗口下草图 API 会 `RPC_E_DISCONNECTED` | 关闭输入尺寸弹窗；超时把模态窗报成可读错误 | 未见系统化弹窗恢复 |
| 断线 | `solidworks_recover` 做诊断。Recovery Decision 默认不自动重试写操作 | 适配器重连会把本地 `state_version` 复位为 0，可能和执行层已前进的版本冲突 | 断路器在适配器层 | 连接失败返回错误 | live conftest 把长时间会话后的 `RPC_E_DISCONNECTED` 写成已知退化，靠测试间关掉文档缓解 | 超时不能中断已发出的 COM 调用 | sidecar 有健康工具；备份失败不阻断 |

## 1.4 质量差异（实现里已经能看见的）

这些差异决定 MoSolid 吸收什么、拒绝什么。

1. **工具成功与任务成功。** 本仓库把复核做成完成条件。Millwright 明确把验证留在证据层。python 的 `verify_model` 允许几何不匹配而工具状态仍为 success。MoSolid 的写操作完成条件采用本仓库的门禁：回读失败、证据缺失、复核未完成，运行状态都不是完成。

2. **审计失败时的行为。** 本仓库 Trace、SolidPilot `ExecLog`、python `_soc_log`、Millwright `session_log.record` 都是观测旁路，写失败不阻止 CAD。MoSolid 对关键写操作要先落审计再调用 COM，审计落盘失败则不执行。这是新门禁，不要改写成“把现有 Trace 变成失败即中断”，以免把已有 eval 的旁路语义搅在一起。

3. **审批是不是真的挡住执行。** CAD Studio 在 worker 里计算审批原因，未批准的任务停在 `approval_required`。MCP 批准账本拒绝模型自带的 `approved=true`。python 的检查点字段在执行函数里被设为已批准。Millwright 的 `auto` 模式跳过确认，备份失败仍继续。MoSolid 默认不提供无人值守的 `auto`。

4. **任意代码。** alisamsam 的 `execute_python`、Millwright 的 `run_shell`、两套 VBA 生成后执行，都让模型逃出类型化工具。hjbaard 明确没有这类工具。MoSolid 的建模路径使用类型化白名单。宏和 shell 若以后存在，单独列为不可逆操作，每次都要人批，不进入 P0 验收路径。

5. **STA。** SolidPilot、hjbaard、python、Millwright 都已经用一条专用线程封送 COM。本仓库的大量脚本在调用线程上连接，短生命周期脚本可以工作，MCP 异步或线程池复用 RCW 时会不稳定。MoSolid 的 COM 进程采用专用 STA，并且 RCW 只在该线程创建。

6. **幂等。** 只有 SolidPilot 把 `operation_id` 与 `state_version` 放进每个变更工具。断线重试时，其余项目可能把同一步再做一遍。MoSolid 采纳这一机制的行为规格，用自己的实现，不复制 SolidPilot 源码。

7. **引用稳定性。** SolidPilot 用面/边索引和几何锚点，并写明锚点经不起上游改尺寸。hjbaard 把尺寸名回给调用方（如 `width@Sketch1`），改尺寸走名字。本仓库有语义实体引用，用来替换 `Face1` 和屏幕坐标。MoSolid 同时保留稳定尺寸名和一次重建内有效的拓扑索引，并在 `state_version` 变化后作废索引。

8. **文档与代码漂移。** 本仓库 MCP 数量、SolidPilot 架构文里的 “38 tools” 与 schema 的 46、Millwright README 的 50 与代码的 87、Millwright 架构文的宏生成主路径与 sidecar 主路径、python 的 npm 集成工作流、hjbaard README 的 733 与 519 个测试函数，都不能直接当现状。MoSolid 的能力真源继续只有一份机器可读清单，文档数字从清单生成。

## 1.5 各项目适合被吸收的部分

| 项目 | 吸收 | 不吸收 |
|---|---|---|
| 本仓库 | 能力清单与等级、Execution Core 门禁、产物台账与本轮新鲜度、Recovery 只决策不自动重试、Policy / MCP 批准、67 个已有工具里已经 E1 的零件/孔/出图/导出、C# Add-in 仅用于必须驻留 UI 的路径、开放格式后端与原生格式阻断分开 | 观测 Trace 失败仍继续执行的语义，不用于关键写操作 |
| SolidPilot | 分层思想：模型只提交意图；确定性编译器降到工具；只有执行层接触 COM。STA、幂等、`state_version`、索引选择、变更前后读回、装配变换与配合读回 | 任何源码、schema 文件、注释、二进制、`server.py` 文案。AGPL 代码不进入本仓库 |
| TS | 13+ 参数走进程内宏这一条**诊断知识**：当 PIA 仍然封送失败时，可以作为版本化后端，而不是默认路径。`RunMacro2` 在 2024+ 的签名差异要写进兼容矩阵 | 未注册的工具模块、默认开启的任意宏执行、把 mock 测试当作完成 |
| alisamsam | 挤压 API 的版本回退顺序，作为探测清单里的一个事实 | `execute_python`、`Dispatch` 可能新开实例的连接顺序、仓库内的历史副本目录 |
| python | SQLite 会话、检查点文件、调用记录、可导出的重放脚本；`verify_model` 的期望值形状；live 测试用环境变量隔离、以及“隐藏窗口会导致草图 RPC 断开”的记录 | 执行时自动把 `approved_by_user` 写成 true；日志失败忽略；工具 status=success 而 verdict=fail；在版权人冲突查清前整文件拷贝 |
| hjbaard | 类型化白名单、每步质量属性、改尺寸回读失败则恢复原值、完全定义草图断言、模态超时、只附着已运行的 SolidWorks、Y-up 模板检查 | 它不覆盖的工程图深度、审计和审批。不把它的 0.19 工具名原样变成 MoSolid 的公共 API |
| Millwright | 桌面多模型、确认卡、审批四档的交互形状、破坏性操作前备份、视觉截图作为**附加**证据、叙述里声称调用了工具但实际没有调用时的纠偏 | `auto` 作为默认、备份失败仍执行、`run_shell`、验证不当门禁、过期架构文里的“生成宏给用户自己跑” |
| MoAspen | 审查确认 `57f9a2e` 为 Tauri 项目。任务要求的形状仍是多模型、工具层、审批、Windows COM、回读、会话、`mcp_audit.jsonl`。本环境打不开该文件树，不引用内部路径 | 没有 LICENSE 之前禁止复制任何文件 |
