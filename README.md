# Game UI Replacement Rules

这是一个用于沉淀“游戏工具替换 UI”的规则库。

目标：把不同游戏风格下的 UI 替换规则拆成可维护、可迭代、可审查的工作流知识库，便于后续被设计、Maker 工程、资源生产和 QA 共同使用。

当前临时策略：01 分诊台只做输入整理和 `style_brief` resolution，不再按“面板 / 背景 / 状态”等部位裁剪风格化生成范围。只要用户提出“做成 / 生成 / 套用 XX 风格”，默认输出背景、程序化面板和 1 个固定角部面板饰品。

## 文档目录

| 编号 | 文档 | 用途 |
|---|---|---|
| 00 | [Index](docs/00-index.md) | 文档索引、阅读顺序和职责地图 |
| 01 | [Workflow Dispatch](docs/01-workflow-dispatch.md) | 临时旁路模式下做输入整理、`style_brief` resolution，并把风格化生成统一归一到完整风格输出 |
| 02 | [Style Knowledge](docs/02-style-knowledge.md) | 根据关键词 / 游戏名检索视觉身份、色彩、材质、形状语言、情绪、世界观元素、UI 翻译建议、禁用误区和标签，只输出 `style_brief` |
| 03 | [Production Constraints](docs/03-production-constraints.md) | 定义 Maker 可实现性、全局资源预算、尺寸边界、导出约束和 QA 失败归因 |
| 04 | [Visual Hierarchy Brief](docs/04-visual-hierarchy-brief.md) | 页面级视觉主次、背景强度、组件层级和色彩角色 |
| 05 | [Material Generation](docs/05-material-generation.md) | 消费 `workflow_request`、`style_brief`、生产约束和视觉层级 brief，生成背景、程序化面板、固定角饰和状态规则 |

## 建议使用方式

1. 用 [01 Workflow Dispatch](docs/01-workflow-dispatch.md) 整理用户需求；风格化生成请求默认开启 `style_generation_bypass: enabled`。
2. 用 [02 Style Knowledge](docs/02-style-knowledge.md) 得到或复用 `style_brief`。
3. 用 [03 Production Constraints](docs/03-production-constraints.md) 检查 Maker 能力、预算、尺寸和替换边界。
4. 用 [04 Visual Hierarchy Brief](docs/04-visual-hierarchy-brief.md) 分配页面视觉主次和背景强度。
5. 用 [05 Material Generation](docs/05-material-generation.md) 生成背景、程序化面板、固定角部面板饰品和必要状态规则。
6. QA 失败时先按 [00 Index](docs/00-index.md) 的职责地图归因，再回修对应文件。
7. 每个规则变更建议通过 PR 审阅，并说明影响范围、适用风格和 QA 风险。
