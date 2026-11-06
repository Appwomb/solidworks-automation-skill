# 3. 来源映射与许可证

原则：行为可以从已读代码里学，AGPL 树里的源码、生成物和契约文件不进 MoSolid 仓库。MIT 与 Apache-2.0 的代码只有在实现阶段、并且保留版权声明时才复制。本设计文档不包含这些项目的源码抄录。

## 3.1 来源映射

“方式”一列的含义：

- **已有**：本仓库已经有对应实现，MoSolid 沿用并加门禁。
- **规格**：只把行为写成我们自己的模块规格，代码新写。
- **候选复用**：许可证允许复制，但要保留声明，并且优先重写接口、只在补齐缺口时复制。
- **禁止复制**：不把该文件或该行为放进产品。

| MoSolid 模块 | 来源 | 方式 | 说明 |
|---|---|---|---|
| 能力清单、等级、后端路由 | 本仓库 `capabilities.yaml`、`scripts/core/capability.py` | 已有 | 继续唯一真源 |
| MCP 网关、发现模式、文档身份 | 本仓库 `mcp-server/` | 已有 | 工具数量以代码为准，并加契约测试 |
| 复核门禁、恢复决策、Trace | 本仓库 `scripts/core/` | 已有 | Trace 仍是旁路。关键写操作另走审计门 |
| 产物台账、本轮新鲜度 | 本仓库 `artifact_ledger.py`、交付门禁 | 已有 | 导出验收使用 |
| Policy Gate、逐次批准账本 | 本仓库 `agent_contracts.py`、`mcp-server/approval.py` | 已有 | 模型参数不能自批 |
| CAD Studio 桌面、多模型桥 | 本仓库 `apps/workbench-ui`、`agent_providers.py` | 已有 | 是否改成与 MoAspen 共用壳，见第 4 篇 |
| 安装板 / 孔 / 工程图 / 导出的已验证脚本 | 本仓库 `scripts/sw_*.py`、子技能、Golden 脚本 | 已有 | P2–P4 优先调用这些路径，不重写一套 COM |
| C# Add-in 宿主 | 本仓库 Add-in 路由 | 已有 | 只用于事件与驻留 UI |
| 无头开放格式 | 本仓库 OCCT 路径 | 已有 | 不替代原生验收 |
| STA 单线程队列 | SolidPilot `StaExecutor` 的设计；hjbaard `ComWorker`；python `ComExecutor`；Millwright `com_executor.py` | 规格 | 四处独立实现说明这是 COM 约束，不是某一份代码的独有算法。MoSolid 按第 2 篇的表新写 |
| `operation_id` 幂等、`state_version` | SolidPilot `OperationGuard` 与契约说明 | 规格 | 不复制 `OperationGuard.cs`、`tool-schemas.json` |
| 特征图、确定性编译、引用解析分层 | SolidPilot `master-architecture.md` 描述的分层 | 规格 | 不复制 schema、`pycompiler/`、recipe、适配器 |
| 索引选面/选边、装配变换读回 | SolidPilot 工具描述中的行为 | 规格 | 查询字段按我们自己的验收需要设计 |
| 尺寸回读失败则写回原值 | hjbaard `session.py` `set_dimension` | 候选复用 | MIT。若复制函数，保留 Henk Jan Baard 的版权声明。更稳妥是按规格重写并引用测试用例思想 |
| 完全定义草图断言、Y-up 模板拒绝、模态超时文案行为 | hjbaard | 规格 / 候选复用 | 测试夹具可作 MIT 参考，不原样改名成公共 API |
| 类型化白名单、无任意代码工具 | hjbaard | 规格 | 产品规则 |
| 只附着已运行实例 | hjbaard；本仓库 Golden“不杀已有进程” | 已有 + 规格 | 禁止 alisamsam 那种 `Dispatch` 创建实例 |
| SQLite 会话、检查点、调用记录、导出重放 | python `history_db.py`、SoC | 候选复用 | 见 3.4 的版权人冲突。在冲突查清前只借鉴表的职责划分，不复制文件 |
| `verify_model` 的期望字段形状 | python `analysis.py` | 规格 | 完成条件改成本仓库门禁：verdict 失败则运行失败 |
| live 测试的环境变量隔离 | python `tests/live/conftest.py` | 规格 | 真机测试默认不在无 SolidWorks 的 CI 里收集 |
| 13+ 参数改走 VBA 的知识 | TS `feature-complexity-analyzer.ts` | 规格 | 作为 PIA 失败后的诊断后端，不作为默认执行器 |
| 挤压 API 回退顺序 | alisamsam `features` 模块 | 规格 | 写进版本探测清单。不复制含 `execute_python` 的服务器 |
| 确认卡、审批档位的交互 | Millwright `agent-loop-sidecar.ts`、`ConfirmCard.tsx` | 候选复用 | Apache-2.0。复制时保留文件内已有声明，并在第三方声明中点出 Raylan LIN / Millwright。不复制 `run_shell`，不把 `auto` 做成默认 |
| 破坏性操作前备份 | Millwright | 规格 | 备份失败则 R2/R3 不执行。这是我们的加严，与 Millwright 当前“失败仍继续”不同 |
| 视觉截图 | Millwright `capture_view`；本仓库审查预览 | 规格 | 截图是附加证据，不能代替尺寸和体积 |
| 叙述与真实工具调用不一致时纠偏 | Millwright agent loop | 规格 | 桌面端可做。MCP 客户端我们控制不了对方 UI |
| 多模型 → 审批 → COM → 回读 → `mcp_audit.jsonl` → Tauri | MoAspen `57f9a2e` | 规格 | 审查确认是 Tauri 且没有 LICENSE。本环境未打开文件树，无文件级映射。出现 LICENSE 之前禁止复制 |
| 任意 Python / shell / 默认执行 VBA | alisamsam、Millwright、TS、python 的生成并执行工具 | 禁止复制 | P0–P4 不注册 |

## 3.2 许可证总览

| 项目 | 许可证 | 版权声明以何为准 | MoSolid 可否放入其源码 |
|---|---|---|---|
| 本仓库 | MIT | `LICENSE`：Copyright (c) 2026 SolidWorks Automation Skill Contributors | 可以。本仓库就是底座。发行物保留该 MIT 声明 |
| SolidPilot | AGPL-3.0，另有商业许可说明 | README：Copyright (c) 2025–2026 Çağatay Bakan。仓库含 `CLA.md` | **默认否。** 见 3.3 |
| SolidworksMCP-TS | MIT | Copyright (c) 2025 ESPO Corporation | 可以，但是候选。复制实质部分时保留该声明和 MIT 许可文本 |
| Solidworks-MCP | MIT | Copyright (c) 2026 Samsaam Ali Baig | 同上。当前没有计划复制，因为任意代码工具不进入产品 |
| SolidworksMCP-python | MIT | LICENSE 与 `pyproject.toml` 作者不一致，见 3.4 | 查清前不复制源文件 |
| SolidWorks-MCP-1 | MIT | Copyright (c) 2026 Henk Jan Baard | 可以。保留声明 |
| Millwright | Apache-2.0 | `LICENSE` 全文；`AUTHORS.md` 写 Raylan LIN。树内未找到 `NOTICE` 文件 | 可以。遵守 3.5 |
| MoAspen | `57f9a2e` 没有 LICENSE 文件 | 审查确认。本环境未打开文件树 | 出现 LICENSE 并且评审通过之前不复制、不 submodule。没有许可证时视为保留所有权利 |

本设计 PR 只新增 `docs/mosolid/` 下的原创说明，没有粘贴上述仓库的源码。

## 3.3 AGPL：SolidPilot 只借鉴设计

SolidPilot 的 LICENSE 是 GNU AGPL-3.0 全文。README 说明：若运行修改版并让他人通过网络与之交互，需要向这些用户提供对应源代码；作者同时提供商业/专有许可的联系方式（README 中的邮箱）。贡献受 CLA 约束。

在老板另行书面决定之前，MoSolid 遵守：

1. 不把 `mcp-server-solidworks` 的源码、测试、schema、recipe、markdown 大段、注释或提交说明复制进本仓库或未来的 MoSolid 仓库。
2. 不把其 `bin/Debug` 下的 exe、pdb、dll 当作依赖或再分发。那些二进制和源码同属该许可证。
3. 不把该目录做成 submodule、vendor 或运行时下载。进程隔离不能使“我们发行了他们的程序”变成合规。
4. 不把 C# 机械翻译成 Python 或 TypeScript。翻译仍是衍生作品。
5. 允许写在本目录里的，是我们自己的行为规格：一条 STA 线程、操作 id、状态版本、意图与执行分离。这些思想来自阅读后的独立描述，实现时工程师不得打开 SolidPilot 源文件对照着改名。
6. 若老板希望直接嵌入 SolidPilot 代码，先取得 Çağatay Bakan 的商业许可，或接受整个对外产品按 AGPL-3.0 履行源码提供义务。在该决定写下之前，不开始这项嵌入。
7. 员工或代理在实现 P6 编译器时，若发现自己正在逐函数对照 `pycompiler`，应停下来改走独立设计，而不是“少抄几行”。

AGPL 不禁止我们阅读公开仓库，也不禁止实现相同的工程思想。它禁止的是在未遵从许可证的情况下复制和发行其程序。本方案选择思想、拒绝程序。

## 3.4 MIT 复用

MIT 要求：实质使用该软件的副本或相当部分时，保留原版权声明和许可声明。

执行时：

- 在仓库根或 `THIRD_PARTY/` 放一份第三方声明，按项目列出版权人、许可证和上游 URL。
- 若某个文件是从 MIT 项目复制或改写的，文件头保留原版权行，并加一行改动说明（年份、MoSolid 维护者、改了什么）。
- 不删除原 LICENSE 文本。MIT 不是“保留版权即可去掉免责声明”。
- 重写达到“没有带走其表达、只保留 API 思想”时，不把对方文件标成我们的原创，也不要在文件头写对方版权（那会错误暗示复制了表达）。第三方文档里写“行为参考”，不写“代码来自”。

`SolidworksMCP-python` 的阻塞项：

- `LICENSE` 首行版权是 Copyright (c) 2025 ESPO Corporation。
- `pyproject.toml` 的 `authors` 与 `maintainers` 是 Andrew Bartels。
- README 的反馈链接指向 `github.com/andrewbartels1/SolidworksMCP-python`。
- `SolidworksMCP-TS` 的 LICENSE 同样是 ESPO Corporation，两个仓库的版权行可能是沿用，也可能是共同权利人。本次没有翻完整 git 历史做法律认定。

因此：实现阶段可以读该仓库、按第 2 篇写自己的 SQLite 表；不整文件复制，直到维护者确认 ESPO 与 Andrew Bartels 的声明应以哪一份为准，并在第三方声明里两行都先保留（若最终复制）。多保留一个真实出现过的版权行，不会比删掉它更危险。

hjbaard、ESPO 的 TS 仓库、Samsaam Ali Baig 的仓库若复制，分别保留各自 LICENSE 中的版权人。本仓库贡献者的 MIT 声明在 MoSolid 发行物中继续保留，因为底座代码是他们的。

## 3.5 Apache-2.0：Millwright

Apache-2.0 允许使用、修改和再发行，同时要求：

- 向接收者提供许可证副本。
- 修改过的文件带有显著的修改声明。
- 保留源文件中已有的版权、专利、商标和归属声明。
- 若发行物包含 `NOTICE`，则在 NOTICE 派生文本中保留其中不属于已删除代码的归属。本次树内没有 `NOTICE` 文件；仍在第三方声明中写明项目名 Millwright 与 AUTHORS 中的 Raylan LIN。
- 不能用对方的商标给 MoSolid 背书。界面上不把产品叫做 Millwright。

`com_executor.py` 开头没有版权行。复制该文件时，仍要在第三方声明中给出项目级归属，并在文件头加上“改写自 Millwright，Apache-2.0”以及修改说明。更符合“少搬代码”的做法是按第 2 篇规格新写 STA 队列，只在确认卡 UI 上考虑复制 Apache 代码。

## 3.6 本设计文档的边界

`docs/mosolid/` 是 Appwomb 在本 MIT 仓库中的新文档，描述公开项目的架构事实和我们的产品决定。它不把 SolidPilot 的 schema 或源码纳入本仓库，因此不构成发行 SolidPilot 修改版。若后续 PR 加入任何来自 AGPL 树的文件，应在评审中拒绝，除非第 6 篇的许可证决定已经改口。
