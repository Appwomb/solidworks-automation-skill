# MoSolid 设计方案

本目录是 MoSolid 的设计冻结稿。本阶段只定义架构、来源、许可证边界、与 MoAspen 的共享方式和验收标准，不包含功能代码。

MoSolid 的目标是：在多个已有 SolidWorks 自动化项目之上，做成一套比其中任何一个都更完整、并且把“工具返回成功”和“几何真的对”分开的桌面 CAD 助手。质量优先。未读到源码、未在本环境重跑的结论，都按下面的证据等级写明。

## 阅读顺序

| 文档 | 内容 |
|---|---|
| [01-comparison-matrix.md](01-comparison-matrix.md) | 各项目能力、架构、质量对比，以及真机证据等级 |
| [02-target-architecture.md](02-target-architecture.md) | 目标模块、工具分层、STA、特征 IR、回读、审计、审批、恢复 |
| [03-source-mapping-and-license.md](03-source-mapping-and-license.md) | 模块来源映射与许可证合规 |
| [04-moaspen-sharing-and-repo.md](04-moaspen-sharing-and-repo.md) | 与 MoAspen 的共享方式、仓库组织 |
| [05-roadmap-and-acceptance.md](05-roadmap-and-acceptance.md) | P0–P7 路线图与 Windows 真机验收 |
| [06-risks-and-decisions.md](06-risks-and-decisions.md) | 风险，以及需要老板决定的事项 |

## 本次实际读到的版本

审阅日期：2026-11-06。克隆来源是 `Appwomb` 下的 fork。下表 SHA 是当时 `HEAD`，后续设计以这些树为准；上游若继续提交，实现前要重新 diff。

| 仓库 | 审阅 SHA | 提交时间（仓库记录） | 许可证（以仓库内 LICENSE 为准） |
|---|---|---|---|
| `Appwomb/solidworks-automation-skill`（本仓库，底座） | `9ba848e301e996cc64d8078df13f4ccc24416339` | 2026-11-04 | MIT，版权人 “SolidWorks Automation Skill Contributors” |
| `Appwomb/mcp-server-solidworks`（SolidPilot） | `f9546a9e11c86cd03e5cab3277805485907c77a2` | 2026-07-21 | AGPL-3.0，README 版权声明为 Çağatay Bakan，2025–2026 |
| `Appwomb/SolidworksMCP-TS` | `ea5539948b816cd34571a4eda00a3a93f4bd86bb` | 2026-09-19 | MIT，Copyright (c) 2025 ESPO Corporation |
| `Appwomb/Solidworks-MCP`（alisamsam） | `ee8f42a1a919af5e0fa8d1dcd24270c9983ce027` | 2026-03-23 | MIT，Copyright (c) 2026 Samsaam Ali Baig |
| `Appwomb/SolidworksMCP-python` | `5451e8ebe5b09bcada9c95b6be7a1d987630ba08` | 2026-11-05 | MIT；LICENSE 写 Copyright (c) 2025 ESPO Corporation，`pyproject.toml` 作者为 Andrew Bartels |
| `Appwomb/SolidWorks-MCP-1`（hjbaard） | `ddcefc0cfa056002ff8eb8bdb186818ad52dc2f1` | 2026-11-03 | MIT，Copyright (c) 2026 Henk Jan Baard |
| `Appwomb/Millwright` | `0e7f9b5332b3d908fa8872eb9958803d6171fc1d` | 2026-10-02 | Apache-2.0；`AUTHORS.md` 记载创始人 Raylan LIN |
| `Appwomb/MoAspen` | 未读到 | — | 未知。私有仓库，当前凭据返回 404 |

本环境没有 Windows，也没有 SolidWorks。文档里的“真机”均指仓库中已有的测试、验收记录或作者声明，不是本次重新执行的结果。

## 证据等级

后续表格使用这些等级，避免把 README 口号写成已验证事实。

| 等级 | 含义 |
|---|---|
| E1 | 仓库里有面向真实 SolidWorks 的验收脚本或验收记录，并写明读回了尺寸、体积、材料或文件。本次未重跑。 |
| E2 | 仓库里有真机测试代码，默认 CI 不跑，或 runner 明确没有 SolidWorks。本次未执行。 |
| E3 | 作者在 README、注释或 GIF 说明中声称真机通过。仓库内没有可复跑证据，或声称的证据文件不在树里。 |
| E4 | 代码实现存在，未见真机测试。 |
| E5 | 文档描述与当前接线不一致，或模块存在但服务器没有注册。 |
| E0 | 源码未读到。只记录任务说明里的约束，不把它写成已核对的实现。 |
