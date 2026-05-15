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

物料文件必须记录 `style_brief_ref` 或 `style_brief_id`，不保存完整 `style_brief` 副本。允许在 `style_to_image_trace` 和 `panel_style_trace` 中记录本次实际使用到的颜色、材质、边框、发光和局部装饰语义，用于 QA 审计；需要完整风格信息时，运行时回查 `01-style-knowledge.md`。

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
    - ui_translation.panel_base_style_hint
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
  procedural_panel_required:
    - consume colors / materials / shape_language / ui_translation.panel_base_style_hint as primary inputs
    - use ui_translation.panel only as panel-system context or legacy fallback, not as implementation spec
    - translate style into fill, radius, border layers, glow layers, edge lights, fixed geometry, and optional accent semantics
    - output panel_render_recipe before implementation
    - do not generate a full panel PNG or require 9-slice metadata
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
    implementation: programmatic_draw
    renderer_recipe: panel_render_recipe
    optional_accent:
      asset: panel_corner_accent.png | panel_accent.png
      required: false
      max_count: 1
  qa:
    rule: procedural_panel_qa_rule
```

## 总流程

1. 用关键词或游戏名从 `01-style-knowledge.md` 取 `style_brief`。
2. 用 `02-production-constraints.md` 检查 Maker 能力、预算、尺寸和替换范围。
3. 盘点目标组件，标记 card、panel、button、status_bar、list_item、popup 等类型。
4. 先规划 UI primitive 和程序化面板主体，再判断是否需要固定位置的局部面板装饰 PNG，最后规划状态 overlay。
5. 输出 `panel_render_recipe`，用程序化绘制还原面板主体。
6. 如风格需要且不破坏可读性，只生成或复用 `panel_corner_accent.png` 或 `panel_accent.png`；AI 图片生成只允许用于这个局部装饰，不生成整张面板主体。
7. 输出 Maker 配置和物料元数据。
8. QA 失败时按 `02-production-constraints.md` 归因回修；如果失败来自 03 没落实程序化面板策略，归因到 03。

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

## 程序化面板规则

### 核心原则

默认面板主体由代码程序化绘制，不生成整张面板 PNG，也不要求 9-slice metadata。

程序化绘制负责：

- 中心底色。
- 圆角。
- 边框层。
- 发光层。
- 边缘高光。
- 固定几何细节。
- 状态 overlay 的承载底板。

AI 图片生成只允许用于 0-1 个固定位置局部装饰 PNG。该装饰不参与拉伸，不承担按钮、状态、徽章、图标或完整面板语义。

### 输入来源

程序化面板主体优先消费：

- `colors`
- `materials`
- `shape_language`
- `ui_translation.panel_base_style_hint`
- `forbidden_mistakes`

`ui_translation.panel` 只作为完整面板系统上下文或旧条目 fallback，不能作为直接实现规格。`world_elements`、`signature_semantics`、`specific_differentiators` 和 `anti_generic_guardrails` 只能帮助避免误读和过度通用化，不能直接变成面板上的角色、logo、文字、道具、徽章或状态标记。

### panel_render_recipe

生成或实现面板前，必须先输出 `panel_render_recipe`。它是代码还原面板主体的唯一默认方案。

```yaml
panel_render_recipe:
  style_brief_ref: docs/01-style-knowledge.md#entry-id
  renderer: NanoVG | Canvas | CSS | engine_ui_primitives
  base_fill:
    color:
    opacity:
    gradient:
      enabled: false
      type: none | linear | radial
      colors: []
      direction:
  corner:
    radius_px:
  border_layers:
    - color:
      width_px:
      opacity:
      inset_px:
  glow_layers:
    - color:
      blur_px:
      opacity:
      spread_px:
  edge_lights:
    left:
      enabled: false
      color:
      width_px:
      opacity:
    right:
      enabled: false
      color:
      width_px:
      opacity:
    top:
      enabled: false
      color:
      height_px:
      opacity:
    bottom:
      enabled: false
      color:
      height_px:
      opacity:
  fixed_geometry:
    notch:
      enabled: false
      size_px:
    ticks:
      enabled: false
      size_px:
      count:
    corner_brackets:
      enabled: false
      size_px:
      stroke_width_px:
  center_policy:
    texture: none
    particle: none
    stretch_safe: true
    clean_for_content: true
  optional_accent:
    enabled: false
    asset: panel_corner_accent.png | panel_accent.png
    max_count: 1
    placement_type: corner
    preferred_anchor: bottom_right
    allowed_anchors:
      - bottom_right
      - top_right
      - bottom_left
      - top_left
    fixed_pixel_size: true
    scale_with_panel: false
    opacity:
    margin_px:
    source_semantics:
      - string
    forbidden:
      - full_panel_png
      - full_border
      - text
      - logo
      - character
      - icon
      - badge
      - state_marker
      - button
  panel_style_trace:
    colors_used:
      - string
    materials_used:
      - string
    border_semantics_used:
      - string
    glow_semantics_used:
      - string
    local_accent_semantics_used:
      - string
```

`panel_render_recipe` 不保存完整 `style_brief`。只记录 `style_brief_ref` 和本次实际消费到的颜色、材质、边框、发光、局部装饰语义 trace。

### 程序化绘制约束

- `corner.radius_px` 是固定像素值，不随面板尺寸比例缩放。
- `border_layers.width_px` 是固定像素值，不因宽高变化而变粗或变细。
- `glow_layers` 必须由 renderer 绘制，不能使用被拉伸的光晕位图。
- `edge_lights` 只能沿直线段延展，不生成整张边框图片。
- `fixed_geometry` 的 notch、ticks、corner brackets 都必须保持固定像素尺寸。
- `center_policy.texture` 和 `center_policy.particle` 必须为 `none`。
- 中心区域必须干净、无图案、无噪声、无场景、无图标、无文字。
- 状态视觉必须由 overlay、描边、亮度、透明度或已有图标层表达，不能烘焙进面板主体。

### 可选局部装饰 PNG

`panel_corner_accent.png` 或 `panel_accent.png` 是 0-1 个可选角部贴片，只能表达固定位置、不参与拉伸的局部风格点缀。默认优先使用右下角角饰；除非 02 或目标组件明确指定其他角，否则其他角保持干净。不得作为侧边挂件、边中装饰或边框式装饰。

允许：

- 抽象材质片。
- 折角。
- 缎带或蕾丝局部角饰。
- 极简角部线稿。
- 小型表面细节。

禁止：

- 完整卡片。
- 整张面板。
- 全边框。
- 白底画布。
- 阴影面板主体。
- 文字、logo、角色、图标、徽章、状态标记、按钮。
- 多个角落装饰或可被误当作状态/功能控件的图形。
- 侧边挂件、边中装饰、边框式装饰或可拉伸边框替代物。

### 局部装饰 prompt 模板

```text
Generate exactly one transparent PNG fixed corner accent patch.
It is not a full panel, not a 9-slice image, and not a background.
The patch will be anchored to one approved panel corner, preferably bottom-right, and must never be stretched.
Use only abstract material, fold, ribbon, lace, linework, or small surface detail inspired by the style_brief.
No text, logo, character, icon, badge, state marker, button, side pendant, edge accent, border-like decoration, full border, full card, shadowed panel body, or white canvas.
Transparent alpha outside the accent.
```

### 9-slice fallback

`panel_base_9slice.png` 不再是默认必需资产。只有在 02 明确允许的极简 fallback 条件成立时，才可申请使用。

fallback 必须满足：

- 已确认目标运行环境无法用程序化绘制稳定还原基础面板。
- 已尝试调整 `panel_render_recipe` 的 fill、radius、border、glow、edge light 和 fixed geometry。
- QA 归因为程序化绘制能力 / 运行环境不适配，而不是 03 没消费 `style_brief` 或程序化参数写错。
- fallback 源图必须极简、规则、低纹理，只作为面板主体兜底，不承载复杂风格表达。

即使使用 fallback，AI 也不能默认生成整张风格化面板图；`panel_base_9slice.png` 只能是极简、干净、规则的兜底资源。

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

### 程序化面板

```yaml
asset_type: procedural_panel_recipe
style_brief_ref: docs/01-style-knowledge.md#entry-id
constraints_ref: docs/02-production-constraints.md
implementation: programmatic_draw
renderer_recipe: panel_render_recipe
panel_render_recipe:
  renderer:
  base_fill:
    color:
    opacity:
    gradient:
      enabled:
      type:
      colors:
      direction:
  corner:
    radius_px:
  border_layers:
    - color:
      width_px:
      opacity:
      inset_px:
  glow_layers:
    - color:
      blur_px:
      opacity:
      spread_px:
  edge_lights:
    left:
    right:
    top:
    bottom:
  fixed_geometry:
    notch:
      enabled:
      size_px:
    ticks:
      enabled:
      size_px:
    corner_brackets:
      enabled:
      size_px:
  center_policy:
    texture: none
    particle: none
    stretch_safe: true
  optional_accent:
    enabled:
    asset: panel_corner_accent.png | panel_accent.png
    required: false
    max_count: 1
    placement_type: corner
    fixed_pixel_size: true
    scale_with_panel: false
    preferred_anchor: bottom_right
    allowed_anchors:
      - bottom_right
      - top_right
      - bottom_left
      - top_left
panel_style_trace:
  colors_used:
    - string
  materials_used:
    - string
  border_semantics_used:
    - string
  glow_semantics_used:
    - string
  local_accent_semantics_used:
    - string
qa_status:
  procedural_multi_size_test:
  no_full_panel_png_used:
  no_9slice_metadata_required:
last_updated:
```

### 可选局部装饰

```yaml
asset_name: panel_corner_accent.png | panel_accent.png
asset_type: fixed_corner_accent_patch
required: false
max_count: 1
style_brief_ref: docs/01-style-knowledge.md#entry-id
format: transparent_png
placement:
  type: corner
  preferred_anchor: bottom_right
  allowed_anchors:
    - bottom_right
    - top_right
    - bottom_left
    - top_left
  fixed_pixel_size: true
  scale_with_panel: false
  margin_px:
source_semantics:
  - abstract_material
  - fold
  - ribbon
  - lace
  - linework
  - small_surface_detail
forbidden_content_checked:
  full_panel:
  full_card:
  full_border:
  text_or_logo:
  character_or_icon:
  badge_or_state_marker:
  button:
  white_canvas:
qa_status:
  transparent_background:
  not_scaled:
  not_full_panel:
  readability_safe:
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

### 程序化面板

每次输出 `panel_render_recipe` 后，必须进行多尺寸程序化渲染验收。

```yaml
procedural_panel_qa_rule:
  required_sizes:
    - 800x300
    - 300x500
    - 660x440
    - 520x200
  checks:
    - corner_radius_not_scaled
    - border_width_constant
    - glow_not_stretched_as_bitmap
    - center_clean_no_texture
    - straight_segments_extend_only
    - fixed_decorations_keep_pixel_size
    - optional_png_accent_not_scaled
    - no_full_panel_png_used
    - no_9slice_metadata_required
```

必须确认：

- `panel_render_recipe` 消费了 `colors`、`materials`、`shape_language` 和 `ui_translation.panel_base_style_hint`。
- 面板主体由 renderer 绘制，不使用整张面板 PNG。
- 圆角、边框、glow 和固定几何在四个尺寸中保持像素逻辑一致。
- 中心区域没有纹理、粒子、图案、场景、文字、图标或状态。
- 直线边只延展线段，不拉伸位图。
- 不要求 `backgroundSlice`、`slice_config` 或其他 9-slice metadata。

### 可选局部装饰

- 是否最多只有 1 个 `panel_corner_accent.png` 或 `panel_accent.png`。
- 是否固定像素尺寸，不随面板宽高缩放。
- 是否优先锚定右下角；如改用其他角，必须有组件避让原因或 02 明确允许。
- 是否透明背景，无白底、无截图、无 demo sheet。
- 是否只是抽象材质、折角、缎带、蕾丝、线稿或小型表面细节。
- 是否没有侧边挂件、边中装饰、边框式装饰或可拉伸边框替代物。
- 是否没有文字、logo、角色、图标、徽章、状态标记、按钮、全边框、完整卡片或整张面板。
- 是否不干扰正文、数值、按钮文案和输入区域。

### 失败归因

```yaml
procedural_panel_failure_attribution:
  style_brief_not_consumed_by_programmatic_params:
    responsible: docs/03-material-generation.md
  default_full_panel_base_9slice_generated:
    responsible: docs/03-material-generation.md
  ai_accent_became_full_card_or_panel_or_button_or_state_icon:
    responsible: docs/03-material-generation.md
  production_constraints_forbid_old_default_but_03_still_uses_it:
    responsible: docs/03-material-generation.md
```

以下情况必须归因 03 并回修本文件或本次生成配置：

- 程序化主体参数没有消费 `style_brief` 的颜色、材质、形状和 `panel_base_style_hint`。
- 仍在默认生成整张 `panel_base_9slice.png` 作为面板主体。
- AI 局部装饰图变成完整卡片、整张面板、按钮、状态图标、徽章或角色/IP 图形。
- 02 已明确禁止旧默认方案，但 03 的流程、prompt、metadata 或 QA 仍要求旧默认方案。

### 状态

- selected、warning、error 是否能一眼区分。
- disabled 是否仍能看清必要文字。
- hover / pressed 是否不改变布局。
- 状态层是否在基础面板之上。

## 常见失败和修复

### 背景过像插画

修复：降低中心细节、降低对比、移除角色主视觉、清晰文字、假按钮和中心强光。

### 面板出现白色矩形

修复：如果来自程序化主体，检查 renderer 的背景清除、base fill 和外层容器透明度；如果来自 `panel_corner_accent.png` / `panel_accent.png`，重新导出 alpha 透明 PNG，并拒绝白底局部装饰。

### 边框在长条上模糊或变粗

修复：回到 `panel_render_recipe`，让 `border_layers.width_px`、`corner.radius_px` 和 `glow_layers.blur_px` 保持固定像素；不要用被拉伸的边框位图替代程序化绘制。

### 小卡片拥挤

修复：减薄 `border_layers`、降低 glow、关闭 optional accent 或扩大卡片 padding；不要在中心区加入纹理、粒子或复杂固定几何。

### 主容器和卡片完全一样

修复：保持同一套程序化规则，但用 base fill opacity、border layer 数量、shadow/glow、padding、surface tint 和间距区分层级，不新增整张面板图片。

### 仍在生成整张面板 PNG

修复：删除默认整图生成步骤，改为输出 `panel_render_recipe`；只有 02 允许的极简 fallback 才能申请 `panel_base_9slice.png`。

### 局部装饰变成完整卡片或状态图标

修复：重写局部装饰 prompt，只保留抽象材质、折角、缎带、蕾丝、线稿或小型表面细节；禁止完整卡片、整张面板、按钮、徽章、状态标记、文字、logo、角色和图标。

### selected 像 warning

修复：调整 selected 为正向选中语义，warning / error 只保留给风险和错误；必要时增加非红色 selected marker。
