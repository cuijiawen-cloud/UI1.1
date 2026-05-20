# 00 Index / 文档索引

`docs/00-index.md` 只是文档索引，用来维护文档目录、核心阅读顺序、职责边界和 QA 回修路径。

它不是分诊台，不负责解析用户意图。真正的用户意图入口是 `docs/01-workflow-dispatch.md`。

当前 01 处于临时旁路模式：它仍负责采集输入和 `style_brief` resolution，但不再用细分分诊裁剪风格化生成范围。凡是“做成 / 生成 / 套用 XX 风格”的请求，默认进入完整风格输出，至少包含背景、程序化面板和 1 个固定角部面板饰品。

## 核心阅读顺序

关键词 / 用户需求
-> `docs/01-workflow-dispatch.md` 分诊台解析意图
-> `docs/02-style-knowledge.md` 风格理解或复用 `style_brief`
-> `docs/03-production-constraints.md` 生产硬约束
-> `docs/04-visual-hierarchy-brief.md` 页面设计规范 / 当前状态审计
-> `docs/05-material-generation.md` 物料生成与实现
-> QA 验收与归因

## 核心文件职责表

| 顺序 | 文档 | 负责内容 | 不负责内容 |
|---|---|---|---|
| 00 | `docs/00-index.md` | 文档目录、阅读顺序、职责地图 | 意图分诊、风格理解、生产约束、设计规范、生成执行 |
| 01 | `docs/01-workflow-dispatch.md` | 用户意图识别、`style_brief` resolution、`style_brief` 复用判断、组件候选盘点、路由到后续文档、输出 `workflow_request` | 具体游戏风格知识、生产硬约束、页面设计规范、prompt、生成执行 |
| 02 | `docs/02-style-knowledge.md` | 游戏风格理解、`style_brief`、颜色 / 材质 / 形状语言、programmatic panel 风格输入边界 | 资源预算、组件处理策略、页面层级、生成执行、QA 硬约束 |
| 03 | `docs/03-production-constraints.md` | Maker 可实现边界、资源预算、`programmatic_draw` 默认策略、资产白名单、fallback 条件、多尺寸 QA 硬约束、失败归因 | 具体游戏风格、页面视觉主次、prompt 或 recipe 执行 |
| 04 | `docs/04-visual-hierarchy-brief.md` | 页面级视觉主次、当前状态审计、背景强度、面板层级、颜色角色、组件风格强度分级、迭代时 keep / reduce / remove / adjust | 具体游戏风格知识、资源预算、prompt、recipe、素材生成 |
| 05 | `docs/05-material-generation.md` | 消费 02 / 03 / 04 输出，生成 background rule、`panel_render_recipe`、optional accent prompt、metadata、QA 执行规则 | 存储具体游戏风格、不重定义生产硬约束、不决定页面级设计主次 |

## QA 回修路径

1. 意图路由错、组件盘点错、`style_brief` 复用错：回修 `docs/01-workflow-dispatch.md`。
2. 游戏风格理解错、`style_brief` 缺字段、风格不像目标游戏：回修 `docs/02-style-knowledge.md`。
3. 超出 Maker、预算、资产白名单、fallback 或硬约束：回修 `docs/03-production-constraints.md`。
4. 背景抢眼、所有矩形都套面板、页面层级混乱、颜色主次不清：回修 `docs/04-visual-hierarchy-brief.md`。
5. prompt / recipe / metadata / QA 执行没落实：回修 `docs/05-material-generation.md`。
6. 素材或最终画面验收失败：进入 QA gate 后再归因。
