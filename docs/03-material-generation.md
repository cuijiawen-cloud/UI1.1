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

物料文件必须记录 `style_brief_ref` 或 `style_brief_id`，不保存完整 `style_brief` 副本。允许在 `style_to_image_trace` 中记录本次实际使用的差异锚点、位置和强度，用于 QA 审计；需要完整风格信息时，运行时回查 `01-style-knowledge.md`。

## style_brief 消费规则

`03-material-generation.md` 必须消费完整 `style_brief`，不能把具名游戏压缩成通用风格原型。

### 必须读取的信息

```yaml
style_brief_consumption:
  use_fields:
    - style_archetype
    - archetype_shared_traits
    - specific_differentiators
    - signature_semantics.must_include
    - signature_semantics.should_include
    - signature_semantics.should_reduce
    - signature_semantics.must_avoid
    - background_identity_anchors.foreground_forbidden
    - background_identity_anchors.edge_or_corner_allowed
    - background_identity_anchors.far_background_allowed
    - background_identity_anchors.center_safe_area_allowed
    - anti_generic_guardrails
    - visual_identity_test
    - visual_identity
    - genre_presentation
    - game_features.core_gameplay
    - world_elements
    - colors
    - materials
    - shape_language
    - background_visual_language
    - ui_translation.panel
    - ui_translation.accent
    - forbidden_mistakes
  background_prompt_required:
    - use style_archetype only as base direction
    - include specific_differentiators
    - include signature_semantics.must_include
    - include signature_semantics.should_reduce
    - include signature_semantics.must_avoid
    - include anti_generic_guardrails
    - place anchors according to background_identity_anchors
```

`style_archetype` 只能帮助选择方向，不能替代差异锚点。背景 prompt 如果只包含 archetype、通用氛围、通用颜色和安全区要求，视为失败。

如果现有 `style_brief` 条目尚未显式包含 Style Specificity Contract，必须先按 `01-style-knowledge.md` 的旧条目补全规则生成临时 specificity 字段，再进入背景 prompt 组装。

### 压缩失败归因

如果产物“好看、低密度、可读，但不像目标游戏”，优先归因为：

```yaml
failure_type: style_specificity_loss
responsible_step: 01_to_03_transfer
fix:
  - 回到 01 补充 style_archetype / specific_differentiators / signature_semantics / anti_generic_guardrails
  - 在 03 prompt 中显式消费 must_include / should_reduce / must_avoid / background_identity_anchors
  - 不先重生成或微调单张图片
```

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
    required: false
    assets:
      "decoration_*.png": 0
    rule: decoration_semantic_policy
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
4. 先规划 UI primitive，再规划 9-slice，再规划状态 overlay；默认不生成独立装饰 PNG。
5. 生成或复用物料。
6. 输出 Maker 配置和物料元数据。
7. QA 失败时按 `02-production-constraints.md` 归因回修。

## 背景图规则

### 目标

背景图负责建立环境气质和页面深度，但必须弱于工具 UI。它不是海报、插画、角色主视觉或游戏截图。
每套风格化物料必须包含 1 张背景图，不能用纯 UI primitive 代替背景图。
背景图必须保留具名游戏的可识别锚点，不能只生成通用题材背景。

### 生成要求

```yaml
background_rule:
  asset_name: background_tool.png
  required: true
  role: low_density_ui_support_background
  style_anchor_policy: preserve_specificity_contract
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
Generate one low-density tool UI background based on the provided style_brief.

Use the style_archetype only as the base direction, but do not stop at the generic archetype.
Preserve the specific_differentiators and signature_semantics so the result does not become a generic style background.

Must include, in a subtle and UI-safe way:
{{signature_semantics.must_include}}

Should include when it can stay low-density and UI-safe:
{{signature_semantics.should_include}}

May include near edges, corners, or far background only:
{{background_identity_anchors.edge_or_corner_allowed}}
{{background_identity_anchors.far_background_allowed}}

Keep the central 45%-60% area clean, low texture, low contrast, and safe for UI content.
The center may only contain:
{{background_identity_anchors.center_safe_area_allowed}}

Reduce or de-emphasize:
{{signature_semantics.should_reduce}}

Must avoid:
{{signature_semantics.must_avoid}}
{{anti_generic_guardrails}}

Do not promote these to foreground or main visual:
{{background_identity_anchors.foreground_forbidden}}

Do not include clear text, logos, fake UI, buttons, avatars, character main visuals, screenshots, or a strong visual center.
This is a support background for an existing tool interface, not a poster, key art, login page, game screenshot, or full illustration.
```

### 验收

- 中央区域放置正文、表单、列表或卡片后仍可读。
- 边缘装饰不抢按钮、输入框、导航和状态提示。
- 没有清晰文字、假 UI、角色主视觉和截图感。
- prompt 中能看到 `specific_differentiators` 和 `signature_semantics.must_include`。
- 至少 2 个 `signature_semantics.must_include` 以低干扰方式出现在边缘、角落或远景。
- `signature_semantics.should_reduce` 被弱化，不能成为主视觉。
- `signature_semantics.must_avoid` 和 `anti_generic_guardrails` 进入实际 prompt。
- 语义位置符合 `background_identity_anchors`，强识别元素不能破坏中央安全区。
- 风格来自 `style_brief`，但物料元数据只记录 `style_brief_ref`，不复制保存完整 `style_brief` 字段。

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

当前固定不生成任何 `decoration_*.png`，数量必须为 0。装饰语义用于指导背景图、9-slice 面板边角、UI primitive、已有图标和状态 overlay 的视觉选择，不作为新增图片资产输出。

```yaml
decoration_asset_policy:
  asset_pattern: decoration_*.png
  decoration_png: 0
  max_count: 0
  new_decoration_assets_allowed: false
  allowed_carriers:
    - background_tool.png
    - panel_base_9slice.png
    - UI_primitives
    - existing_icon_library
    - state_overlay
```

### 语义类型（不新增 PNG）

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

- 新增独立 `decoration_*.png`。
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
  genericness_check:
  traceability_check:
style_to_image_trace:
  visual_identity_used:
  archetype_used:
  differentiators_used:
    - differentiator:
      image_expression:
      location: edge | corner | far_background | center_safe_area
      strength: low | medium | high
  must_include_coverage:
    - semantic:
      present: true | false
      where:
  should_reduce_check:
    - semantic:
      reduced: true | false
      note:
  must_avoid_check:
    - rule:
      violated: true | false
      note:
  final_specificity_score: 1-5
  final_ui_safety_score: 1-5
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
asset_type: semantic_decoration_policy
style_brief_ref: docs/01-style-knowledge.md#entry-id
decoration_png: 0
new_decoration_assets_allowed: false
semantic_carriers:
  - background_tool.png
  - panel_base_9slice.png
  - UI_primitives
  - existing_icon_library
  - state_overlay
qa_status:
  no_new_decoration_png:
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
- 是否至少有 2 个 `signature_semantics.must_include` 以低干扰方式出现在边缘、角落或远景。
- 是否没有只停留在 `style_archetype` 的通用表达。

### Genericness Check

```yaml
genericness_check:
  question: "这张图是否可以不修改地套用到 3 个以上同原型游戏？"
  fail_if:
    - yes
    - target_game_signature_semantics_not_visible
    - result_only_contains_archetype_shared_traits
  pass_if:
    - center_safe_area_is_clean
    - no_fake_ui_or_text
    - at_least_2_signature_semantics_are_visible_in_edges_or_far_background
    - must_avoid_rules_are_not_violated
```

如果 `genericness_check` 失败，即使 UI 安全区、低干扰和无假 UI 都合格，也必须回修风格特异性。

### Traceability Check

每次生成背景图后，必须输出可审计映射：

```yaml
style_to_image_trace:
  visual_identity_used:
  archetype_used:
  differentiators_used:
    - differentiator:
      image_expression:
      location: edge | corner | far_background | center_safe_area
      strength: low | medium | high
  must_include_coverage:
    - semantic:
      present: true | false
      where:
  should_reduce_check:
    - semantic:
      reduced: true | false
      note:
  must_avoid_check:
    - rule:
      violated: true | false
      note:
  final_specificity_score: 1-5
  final_ui_safety_score: 1-5
```

如果 `final_specificity_score < 3`，即使背景好看，也应回修 `01-style-knowledge.md` 或重新生成 prompt。

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
