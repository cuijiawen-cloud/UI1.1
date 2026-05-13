# 02 Production Constraints

> 角色：定义 Maker 生产链路的全局硬约束。本文件不绑定任何具体游戏风格，也不保存风格特征。

## 输入与输出

输入：

- 已确认的工具页面、组件清单和替换范围。
- `01-style-knowledge.md` 输出的 `style_brief` 引用。
- 目标 Maker / UrhoX 工程环境。

输出：

- 可做 / 不可做边界。
- 默认资源预算与 fallback 条件。
- 尺寸、切片、图层和导出硬约束。
- QA 失败归因表和回修顺序。

## Maker 可实现边界

### 可以做

- 在不改业务逻辑的前提下替换视觉皮肤。
- 使用 `backgroundColor`、`borderWidth`、`borderColor`、`borderRadius`、`opacity`、`shadow`、`tint` 等 UI primitive。
- 使用透明 PNG 作为背景和面板；当前不允许新增 `decoration_*.png`。
- 使用 `UI.Panel` 的真实 9-slice / scale9 渲染。
- 使用独立 overlay 表达 selected、disabled、locked、warning 等状态。
- 使用简单 tween：透明度、位移、缩放、轻微呼吸、短时高亮。
- 复用现有字体、图标库和组件结构。

### 不应该做

- 改变用户内容、页面结构、组件数量、组件位置、交互流程或业务语义。
- 把文字、按钮、勾选、头像、状态、品类条直接烘焙到基础图片里。
- 用一张普通拉伸图冒充 9-slice。
- 为每个尺寸生成一张面板图。
- 在背景图中生成清晰文字、假 UI、角色主视觉、强视觉中心或高密度插画。
- 依赖复杂 shader、流体、布料、骨骼动画、大量粒子、视频背景或实时 3D。
- 使用未授权 logo、角色、IP 专属符号或商业素材复刻。

## 默认资源预算

默认预算用于“已有工具 UI 的风格替换”，不是完整游戏界面重做。

```yaml
asset_budget_default:
  background_image: 1
  panel_9slice_base: 1
  panel_9slice_fallback: 0
  decoration_png: 0
  state_image: 0
  icon_new: 0-4
  particle_or_animation_asset: 0
  font_new: 0
```

`decoration_png: 0` 表示当前不允许新增任何 `decoration_*.png`。

优先级：

1. 先使用 UI primitive。
2. 背景图必须保留 1 张，用于承载页面氛围和世界观语义。
3. 再使用一张可复用 9-slice 面板图。
4. 不生成独立装饰 PNG；装饰语义优先由背景图、9-slice 面板、UI primitive 或已有图标承载。
5. 只有确认失败且说明原因后，才允许增加 fallback 资源。

## 资源 fallback 条件

允许新增 fallback 资源必须同时满足：

1. 当前组件超出默认资源的安全使用边界。
2. 已尝试调整切片、padding、opacity、tint、shadow、primitive 组合。
3. 问题被 QA 归因为“资源形态不匹配”，不是风格理解错误或实现参数错误。
4. 新增资源的用途、适用组件和禁用范围已写入物料元数据。

默认允许的第一个 fallback：

```yaml
asset: bar_panel_9slice.png
for:
  - status_bar
  - long_info_bar
  - long_button
condition: confirmed_failure_or_explicit_approval
```

## 图片导出硬约束

所有透明 PNG：

- 外部区域必须是 alpha 透明，不允许白底画布。
- 不允许截图背景、mockup 预览、demo sheet 或多个尺寸示例。
- 不允许清晰文字、logo、头像、按钮文案或交互控件内容。
- 不允许把状态视觉烘焙进基础资源。
- 边缘与角必须干净，避免半透明脏边和压缩噪点。

背景图：

- 服务 UI，不做完整插画主视觉。
- 中央 45%-60% 保持低纹理、低对比、低信息密度。
- 装饰优先放在边缘和角落。
- 不得影响正文、表单、列表、按钮和可点击区域识别。

9-slice 图：

- 必须是单张透明 PNG 源图。
- 必须有显式 `backgroundSlice`。
- 必须使用真实 sliced 渲染，不允许普通 stretch。
- 中心区域必须可拉伸，角和边必须留在非拉伸区。

## 尺寸边界

### 9-slice 安全边界

```yaml
panel_base_9slice_safe_usage:
  min_width: 120
  min_height: 120
  aspect_ratio_min: 0.75
  aspect_ratio_max: 2.8
  component_types:
    - character_card
    - item_card
    - selection_card
    - medium_panel
    - popup_content_panel
    - settings_panel
```

谨慎使用：

```yaml
panel_base_9slice_caution_usage:
  height: 96-120
  aspect_ratio: 2.8-3.2
  required_checks:
    - border_clarity
    - corner_not_squeezed
    - content_not_crowded
    - no_blur
```

禁止使用：

```yaml
panel_base_9slice_forbidden_usage:
  height_below: 96
  aspect_ratio_above: 3.2
  component_types:
    - status_bar
    - long_info_bar
    - thin_button
    - toolbar
    - divider
    - pill_label
    - tiny_icon_panel
```

### 切片边界

- 推荐切片边距为源图尺寸的 8%-14%。
- 角装饰必须完全落在非拉伸区。
- 对低高度组件，`top + bottom` 不应超过组件高度的 30%-40%。
- 必须满足 `component_height - top - bottom > 0`。
- 必须满足 `component_width - left - right > 0`。
- 坐标和尺寸优先整数对齐，避免细边模糊。

## 图层约束

推荐图层顺序：

```yaml
1: background_image_or_page_color
2: sliced_panel_background
3: optional_tint_or_surface_overlay
4: content_image_or_icon
5: text
6: top_strip_or_category_layer
7: selected_overlay_or_border
8: state_marker_or_check
9: disabled_overlay_if_needed
```

状态层必须位于基础面板之上。禁用态 overlay 可以覆盖内容，但不能让关键文字不可读。

## QA 失败归因

QA 失败先归因，再回修。不要在没有归因时直接重生成物料。

| 失败现象 | 优先归因层 | 回修文件 |
|---|---|---|
| 看起来不像目标游戏或关键词 | 风格理解错误 | `01-style-knowledge.md` |
| 背景质量合格，但像通用风格原型，不像目标游戏 | Style Specificity Contract 缺失 | `01-style-knowledge.md` |
| `style_brief` 有差异锚点，但生成 prompt 没有使用 | prompt 消费失败 | `03-material-generation.md` |
| 差异锚点出现了，但破坏 UI 安全区 | identity anchor placement 错误 | `03-material-generation.md` |
| 颜色、材质、形状语言和情绪冲突 | 风格理解错误 | `01-style-knowledge.md` |
| 背景像完整插画，遮挡内容 | 物料生成错误 | `03-material-generation.md` |
| 背景中心太花、对比太强 | 物料生成错误 | `03-material-generation.md` |
| 面板有白底或截图底 | 图片导出错误 | `02-production-constraints.md` |
| 9-slice 被普通拉伸，边框模糊 | 工程实现错误 | `02-production-constraints.md` |
| 长条组件像被压扁的卡片 | 尺寸边界错误 | `02-production-constraints.md` |
| selected 和 warning 都像红色警告 | 状态语义错误 | `03-material-generation.md` |
| 装饰随机堆叠，干扰阅读 | 装饰语义错误 | `03-material-generation.md` |
| 资源数量暴涨、每个尺寸一张图 | 预算错误 | `02-production-constraints.md` |
| 内容、布局、交互被改了 | 替换范围错误 | `02-production-constraints.md` |

## 回修顺序

1. 如果风格判断错，先回到 `01-style-knowledge.md` 修 style_brief。
2. 如果风格判断正确但背景只剩通用 archetype，先补 `01-style-knowledge.md` 的 `specific_differentiators`、`signature_semantics` 和 `anti_generic_guardrails`。
3. 如果 `style_brief` 有差异锚点但 prompt 没消费，回到 `03-material-generation.md` 修背景 prompt 模板。
4. 如果差异锚点破坏 UI 安全区，回到 `03-material-generation.md` 调整 `background_identity_anchors` 的位置和强度。
5. 如果产物超出 Maker 或预算边界，回到本文件收紧约束。
6. 如果风格正确且传递完整，但生成物料不合格，回到 `03-material-generation.md` 修生成规则。
7. 如果实现参数错误，优先修 Maker 配置，不重新生成图片。
8. 只有资源形态被确认不匹配时，才进入 fallback 资源申请。
