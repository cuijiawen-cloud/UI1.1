# 文档索引

## 核心阅读顺序

关键词 / 游戏名 -> 风格理解 -> 生产约束 -> 物料生成 -> QA 归因回修。

## 核心文件

| 顺序 | 文档 | 负责内容 | 不负责内容 |
|---|---|---|---|
| 01 | [Style Knowledge](01-style-knowledge.md) | 风格理解、游戏特征、色彩、材质、形状语言、情绪、世界观元素、UI 翻译建议、禁用误区、可检索标签、`style_brief` | 图片数量、尺寸、9-slice 参数、Maker 预算、QA 归因 |
| 02 | [Production Constraints](02-production-constraints.md) | Maker 可实现性、资源预算、尺寸边界、图片导出约束、QA 失败归因 | 具体游戏风格特征、背景 / 面板 / 装饰 / 状态的生成 prompt |
| 03 | [Material Generation](03-material-generation.md) | 背景图规则、9-slice 图规则、装饰图规则、状态视觉规范、物料元数据 | 存储具体游戏风格特征、改写全局硬约束 |

## QA 回修路径

1. 看起来不像目标游戏：回修 `01-style-knowledge.md`。
2. 超出 Maker、预算或尺寸边界：回修 `02-production-constraints.md`。
3. 风格正确但物料产物不合格：回修 `03-material-generation.md`。
4. 实现参数错误：优先修 Maker 配置，不直接重生成图片。
