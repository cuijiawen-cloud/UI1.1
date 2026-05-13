# Game UI Replacement Rules

这是一个用于沉淀“游戏工具替换 UI”的规则库。

目标：把不同游戏风格下的 UI 替换规则拆成可维护、可迭代、可审查的三段式知识库，便于后续被设计、Maker 工程、资源生产和 QA 共同使用。

## 文档目录

| 编号 | 文档 | 用途 |
|---|---|---|
| 01 | [Style Knowledge](docs/01-style-knowledge.md) | 根据关键词 / 游戏名检索视觉身份、色彩、材质、形状语言、情绪、世界观元素、UI 翻译建议、禁用误区和标签，只输出 `style_brief` |
| 02 | [Production Constraints](docs/02-production-constraints.md) | 定义 Maker 可实现性、全局资源预算、尺寸边界、导出约束和 QA 失败归因 |
| 03 | [Material Generation](docs/03-material-generation.md) | 消费 `style_brief` 和生产约束，生成背景图、9-slice 图、装饰图和状态视觉规范 |

## 建议使用方式

1. 用关键词、游戏名或标签进入 [01 Style Knowledge](docs/01-style-knowledge.md)，得到 `style_brief`。
2. 用 [02 Production Constraints](docs/02-production-constraints.md) 检查 Maker 能力、预算、尺寸和替换边界。
3. 用 [03 Material Generation](docs/03-material-generation.md) 生成背景、9-slice 面板、装饰和状态规则。
4. QA 失败时先按 [02 Production Constraints](docs/02-production-constraints.md) 归因，再回修 `01` 的风格理解、`02` 的硬约束或 `03` 的物料规则。
5. 每个规则变更建议通过 PR 审阅，并说明影响范围、适用风格和 QA 风险。
