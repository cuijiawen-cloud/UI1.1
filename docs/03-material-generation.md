# 03 Material Generation

> 角色：消费 `style_brief` 和全局生产约束，生成具体物料规则。本文件只定义生成流程和产物规格，不重复存储任何具体游戏风格特征。

## 输入契约

必须先取得：

```yaml
inputs:
  style_brief_ref: docs/01-style-knowledge.md#对应条目
  style_brief: provided_by_retrieval
  production_constraints_ref: docs/02-production-constraints.md
  target_components:
    - component_name
    - component_type
    - width
    - height
    - state_requirements
```

物料文件只允许记录 `style_brief_ref` 或 `style_brief_id`，不允许复制保存颜色、材质、世界观元素、情绪等风格特征。需要风格信息时，运行时回查 `01-style-knowledge.md`。

## 输出包

一次完整生成应输出：

```yaml
material_package:
  background:
    required: true
    asset: background_tool.png
    rule: background_rule
  panel:
    required: true
    asset: panel_base_9slice.png
    rule: nine_slice_panel_rule
  decorations:
    required: optional
    assets: []
    rule: decoration_semantic_rule
  states:
    required: true
    assets: []
    rule: state_visual_rule
  qa:
    rule: material_qa_rule
```

## 总流程

1. 用关键词或游戏名从 `01-style-knowledge.md` 取 `style_brief`。
2. 用 `02-production-constraints.md` 检查 Maker 能力、预算、尺寸和替换范围。
3. 盘点目标组件，标记 card、panel、button、status_bar、list_item、popup 等类型。
4. 先规划 UI primitive，再规划 9-slice，再规划装饰 PNG，最后规划状态 overlay。
5. 生成或复用物料。
6. 输出 Maker 配置和物料元数据。
7. QA 失败时按 `02-production-constraints.md` 归因回修。

## 背景图规则

### 目标

背景图负责建立环境气质和页面深度，但必须弱于工具 UI。它不是海报、插画、角色主视觉或游戏截图。
每套风格化物料必须包含 1 张背景图，不能用纯 UI primitive 代替背景图。

### 生成要求

```yaml
background_rule:
  asset_name: background_tool.png
  required: true
  role: low_density_ui_support_background
  format: png_or_jpg
  center_safe_area: 45%-60%
  information_density: low
  contrast: low_to_medium_low
  saturation: follows_style_brief_but_restrained
  focus: edge_and_corner_atmosphere
  forbidden:
    - clear_text
    - fake_ui
    - button_like_shapes
    - logo
    - avatar_or_character_main_visual
    - screenshot
    - high_contrast_center
    - dense_center_pattern
```

### Prompt 模板

```text
Generate one tool UI background based on the provided style_brief.
Use the style_brief mood, world_elements, colors, materials, and background_visual_language as direction.
Keep the central 45%-60% area clean, low texture, low contrast, and safe for UI content.
Place atmosphere and decorative world elements near edges and corners.
Do not include clear text, logos, fake UI, buttons, avatars, character main visuals, screenshots, or a strong visual center.
This is a support background for an existing tool interface, not a poster or illustration.
```

### 验收

- 中央区域放置正文、表单、列表或卡片后仍可读。
- 边缘装饰不抢按钮、输入框、导航和状态提示。
- 没有清晰文字、假 UI、角色主视觉和截图感。
- 风格来自 `style_brief`，但生成文件不复制保存 `style_brief` 字段。

## 9-slice 面板规则

### 核心原则

9-slice 面板不是普通图片，它必须包含：

```yaml
transparent_png_source_image: panel_base_9slice.png
render_mode: sliced_or_scale9
slice_config:
  top: number_px
  right: number_px
  bottom: number_px
  left: number_px
```

只有同时满足以下条件，才是有效 9-slice：

1. 单张透明 PNG 源图。
2. 有明确切片参数。
3. UI 引擎使用真实 9-slice / scale9 渲染。
4. 可在兼容尺寸内复用，而不是普通整图拉伸。

### 默认策略

```yaml
default_panel_image_budget: 1
required_asset:
  - panel_base_9slice.png
strategy: one_image_plus_multiple_parameters
```

不要默认生成多张面板图，不要为每个尺寸单独生成图片。组件差异优先通过以下方式表达：

- `backgroundSlice`
- opacity
- tint
- padding
- shadow
- border overlay
- selected overlay
- top strip layer
- disabled overlay
- corner accent visibility

### 源图属性

`panel_base_9slice.png` 必须是中性、可复用的面板皮肤。

必须有：

- 透明 PNG。
- 外部 alpha 透明。
- 只包含面板主体。
- 干净可拉伸中心。
- 轻量边框。
- 低对比材质纹理。
- 克制角处理。
- 无强装饰、无状态、无文字。

禁止有：

- 白底画布。
- 截图背景或 mockup 预览。
- 多尺寸示例或 demo sheet。
- 文字、logo、头像、按钮内容。
- 勾选、选中、禁用、警告等状态。
- 顶部分类条或稀有度条。
- 强阴影、大角花、复杂中心图案。

### 生成 prompt 模板

```text
Generate exactly one transparent PNG UI panel asset named panel_base_9slice.png.
Use the provided style_brief only as visual direction; do not include text or copied game marks.
This is a source image for real 9-slice rendering, not a mockup, demo sheet, or preview.
The image must contain only the panel body. The outside area must be fully transparent alpha.
The center area must be clean, low texture, and freely stretchable.
The border must be readable but lower priority than future content.
Corners must be minimal and fit inside the slice area.
Do not include text, logo, avatar, icon, check mark, selected state, top strip, disabled overlay, white background, screenshot background, or multiple size examples.
```

### Maker / UrhoX 配置

```lua
UI.Panel {
  backgroundImage = "image/panel_base_9slice.png",
  backgroundFit = "sliced",
  backgroundSlice = { top = 24, right = 24, bottom = 24, left = 24 },
}
```

已知能力：

```yaml
rendering: true_9slice
backgroundFit: sliced
backgroundSlice_unit: px
supports_transparent_png: true
supports_independent_sides: true
supports_runtime_resize: true
children_overlay: true
```

### 切片指南

```yaml
slice_edge_range: 8%-14% of source image dimension
corners: fully_inside_non_stretch_areas
center: largest_possible_clean_stretch_area
```

示例：

```yaml
source_image: 512x384
card_or_medium_panel:
  top: 24
  right: 24
  bottom: 24
  left: 24
large_container:
  top: 40
  right: 48
  bottom: 40
  left: 48
caution_long_bar_test_only:
  top: 12
  right: 24
  bottom: 12
  left: 24
```

低高度组件必须额外检查：

```yaml
max_top_bottom_ratio_for_low_height: 0.30-0.40
invalid_if:
  component_height_minus_top_minus_bottom: <= 0
  component_width_minus_left_minus_right: <= 0
```

### 使用边界

安全使用：

```yaml
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
height: 96-120
aspect_ratio: 2.8-3.2
requirements:
  - visually_verify_border_clarity
  - verify_corners_are_not_squeezed
  - verify_content_is_not_crowded
  - verify_no_blur
```

禁止使用：

```yaml
aspect_ratio: "> 3.2"
height: "< 96"
component_types:
  - status_bar
  - long_info_bar
  - thin_button
  - toolbar
  - divider
  - pill_label
  - tiny_icon_panel
symptoms:
  - blurred_border
  - heavy_end_caps
  - squeezed_corners
  - flattened_card
```

对禁止使用的长条组件，优先用 UI primitive。只有确认失败后，才请求 `bar_panel_9slice.png`。

## 装饰图规则

### 目标

装饰图用于表达世界观语义、层级或状态提示，不用于填满空间。装饰必须来自 `style_brief.ui_translation.accent` 或 `style_brief.world_elements`，但物料元数据只记录引用关系。

### 类型

```yaml
decoration_types:
  corner_accent:
    use: card_or_panel_identity
    max_per_component: 1-2
  divider_or_tab_marker:
    use: structure_hint
    max_per_group: 1
  badge_or_small_marker:
    use: state_or_category_hint
    max_per_component: 1
  edge_texture_chip:
    use: atmosphere_without_readability_cost
    max_per_screen: 4
```

### 禁止

- 随机堆叠和无语义装饰。
- 把装饰放在正文、数值、按钮文案或输入区域下方。
- 每张卡都使用大角花。
- 装饰比内容更醒目。
- 用装饰替代状态层、标签层或真实图标语义。

### 密度建议

```yaml
decoration_density:
  small_card: none_or_one_small_marker
  medium_card: one_corner_or_one_badge
  large_panel: two_to_four_edge_accents
  full_screen: edge_weighted_not_center_weighted
```

## 状态视觉规则

### 状态集合

```yaml
state_visuals:
  normal: baseline_readable
  hover: subtle_lift_or_highlight
  pressed: slight_inset_or_darker_surface
  selected: positive_selection_marker
  disabled: desaturate_and_opacity_overlay
  locked: lock_marker_or_blocked_surface
  warning: caution_semantics_only
  error: destructive_or_invalid_semantics_only
  success: completed_or_confirmed_semantics
  new: discovery_or_unread_marker
  reward: gain_or_bonus_marker
```

### 视觉通道优先级

1. 色彩语义。
2. 边框或内描边。
3. 亮度、阴影、glow 或 overlay。
4. 小图标或角标。
5. 简单动效。

selected 不等于 warning。除非 `style_brief` 明确要求，否则 selected 不使用警告红；warning / error 不使用普通奖励色或装饰色。

### 状态实现

- 基础面板不烘焙状态。
- selected 使用 overlay、描边、小角标、内高光或轻 glow。
- disabled 使用半透明灰化或降饱和 overlay。
- locked 可以使用锁标识、遮罩或受限纹理，但文字仍需可读。
- warning / error 必须比普通 selected 更明确，避免被误解为“当前选中”。
- hover / pressed 优先使用参数变化，默认不新增图片。

## 物料元数据模板

### 背景

```yaml
asset_name: background_tool.png
asset_type: ui_background
style_brief_ref: docs/01-style-knowledge.md#entry-id
constraints_ref: docs/02-production-constraints.md
center_safe_area: 45%-60%
forbidden_content_checked: true
qa_status:
  readable_center:
  no_fake_ui:
  no_clear_text:
  no_character_main_visual:
last_updated:
```

### 9-slice 面板

```yaml
asset_name: panel_base_9slice.png
asset_type: 9slice_panel
format: transparent_png
style_brief_ref: docs/01-style-knowledge.md#entry-id
source_size:
  width:
  height:
recommended_slice:
  top:
  right:
  bottom:
  left:
safe_usage:
  min_width: 120
  min_height: 120
  aspect_ratio_min: 0.75
  aspect_ratio_max: 2.8
state_policy:
  selected: overlay_not_baked
  disabled: overlay_not_baked
  top_strip: separate_ui_layer
image_budget_policy:
  default: one_image
  fallback: bar_panel_9slice_only_after_confirmed_failure
qa_status:
  transparent_background:
  real_sliced_rendering:
  multi_size_test:
  blur_test:
last_updated:
```

### 装饰

```yaml
asset_name: decoration_*.png
asset_type: semantic_decoration
style_brief_ref: docs/01-style-knowledge.md#entry-id
semantic_role: corner_accent | divider | badge | edge_texture_chip
allowed_positions: []
forbidden_positions:
  - text_area
  - input_area
  - primary_button_text_area
  - dense_list_center
max_instances:
qa_status:
  semantic_match:
  readability_safe:
  density_safe:
last_updated:
```

## QA 检查

### 背景

- 是否服务 UI，而不是完整插画。
- 中央安全区是否干净。
- 是否无清晰文字、假 UI、logo、角色主视觉。
- 是否保留足够对比给真实组件。

### 9-slice

- 是否透明 PNG。
- 是否无白底、无截图、无 demo sheet。
- 是否中心可拉伸。
- 是否使用 `backgroundFit = "sliced"`。
- 是否有 px 级 `backgroundSlice`。
- 是否在安全尺寸内复用。
- 是否没有把 selected、disabled、top strip 烘焙进基础图。

### 装饰

- 是否有语义。
- 是否只在边缘、角、分隔处或状态提示处使用。
- 是否没有干扰正文、数值、按钮和输入。
- 是否密度可控。

### 状态

- selected、warning、error 是否能一眼区分。
- disabled 是否仍能看清必要文字。
- hover / pressed 是否不改变布局。
- 状态层是否在基础面板之上。

## 常见失败和修复

### 背景过像插画

修复：降低中心细节、降低对比、移除角色主视觉、清晰文字、假按钮和中心强光。

### 面板出现白色矩形

修复：重新导出 alpha 透明 PNG；如果源图无法确认透明度，拒绝使用。

### 边框在长条上模糊

修复：不要把 `panel_base_9slice.png` 用在 aspect ratio > 3.2 或 height < 96 的组件上；先用 UI primitive，必要时申请 `bar_panel_9slice.png`。

### 小卡片拥挤

修复：减薄边框、移除大角装饰、减小切片值或扩大卡片 padding；不要在基础图里做复杂中心纹理。

### 主容器和卡片完全一样

修复：保持同一张图，但用 slice、opacity、padding、shadow、surface tint 和间距区分层级。

### selected 像 warning

修复：调整 selected 为正向选中语义，warning / error 只保留给风险和错误；必要时增加非红色 selected marker。
