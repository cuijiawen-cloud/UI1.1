# 01 Workflow Dispatch

> 角色：01 Workflow Dispatch / 分诊台 / Intent Router。负责把用户自然语言需求转换成稳定、可扩展的 `workflow_request`，决定默认是全局页面风格更新还是显式局部元素修改，并完成 `style_brief` resolution、页面上下文采集、组件候选盘点和后续路由。

本文件只负责意图识别、范围判断和输入整理；不负责新增具体游戏风格、定义生产预算、编写 prompt、生成 recipe、指定渲染参数或执行 QA。

## 一、核心原则

### 默认全局风格

任何“换成 / 做成 / 生成 / 套用 / 改成 XX 风格”的视觉风格请求，默认都解释为 `global_page_style`，也就是更改当前页面的完整风格系统，而不是只改用户句子里提到的单个元素。

默认全局风格至少包含：

```yaml
global_page_style_default:
  background:
    required: true
    output: background_tool.png
  panel_system:
    required: true
    output: programmatic_panel
    applies_to: safe_panel_components_from_inventory
  panel_accent:
    required: true
    output: panel_corner_accent.png | panel_accent.png
    max_count: 1
  page_color_system:
    required: true
    output: page_color_rule
    note: non_image_style_rule_not_counted_as_generated_asset
  state_system:
    required_when:
      - user_mentions_state
      - selected_disabled_warning_error_may_conflict_with_new_colors
```

用户只说“面板换成 XX 风格”“背景做成 XX 风格”“按钮状态像 XX 游戏”，仍然默认是“以该元素为重点的全局页面风格更新”。这些表达不等于局部修改。

### 显式限定才局部

只有当用户明确表达“只改某个元素 / 保持其他不变 / 不要动背景或配色 / 单独给某个组件出规则”时，01 才把请求解释为 `element_style_patch`。

典型局部限定词：

```yaml
explicit_scope_markers:
  only_words:
    - 只
    - 仅
    - 单独
    - 局部
    - 只针对
    - 仅针对
  preserve_words:
    - 其他不变
    - 保持背景不变
    - 保持面板不变
    - 不要动背景
    - 不要改配色
    - 不影响其他组件
  asset_only_words:
    - 只出
    - 只生成
    - 单独生成
    - 仅验收
```

没有这些显式限定时，组件名只作为 `requested_focus_elements`，用于确定全局风格更新中的重点区域，不用于缩小输出范围。

### 主链路

```text
00 Index
-> 01 Dispatch
-> 02 Style Knowledge
-> 03 Production Constraints
-> 04 Visual Hierarchy Brief
-> 05 Material Generation
```

01 必须决定：

- 是否是视觉风格请求。
- `style_application_scope.mode` 是 `global_page_style` 还是 `element_style_patch`。
- 是否需要 `02-style-knowledge.md` 重新执行 Style Discovery。
- 是否需要 `04-visual-hierarchy-brief.md` 输出页面视觉层级。
- 是否需要 `current_visual_audit`。
- 进入 05 前是否因为缺少游戏名、背景尺寸、页面上下文或组件盘点而阻断。

## 二、可扩展范围模型

01 不通过不断增加 route 名称来表达新需求，而是用 `style_application_scope` 描述范围，用 `route_to` 维持与 05 的执行兼容。

```yaml
style_application_scope:
  mode:
    - global_page_style
    - element_style_patch
    - qa_only
    - style_brief_only
    - production_constraints_only
    - metadata_only
  defaulted_to_global: true | false
  scope_reason:
  explicit_scope_markers: []
  requested_focus_elements: []
  preserve_surfaces: []
  global_style_surfaces:
    background: required | not_required
    panel_system: required | not_required
    panel_accent: required | not_required
    page_color_system: required | not_required
    state_system: required | conditional | not_required
  element_patch:
    target_elements: []
    allowed_outputs: []
    preserve_other_surfaces: true | false
```

### mode 到 route 的映射

```yaml
scope_to_route:
  global_page_style:
    route_to: full_ui_skin
    global_style_generation_enabled: enabled
    visual_hierarchy_scope: required
  element_style_patch:
    route_to:
      background_only: background_generation
      panel_only: panel_programmatic_draw
      panel_demo_only: panel_demo
      state_only: state_guidance
      color_only: page_color_rule_only
      mixed_element_patch: full_ui_skin
    global_style_generation_enabled: disabled
    visual_hierarchy_scope:
      required_when:
        - target_element_affects_page_hierarchy
        - patch_changes_background_or_page_colors
        - patch_changes_readability_or_focus
        - patch_changes_selected_warning_error_semantics
      not_required_when:
        - isolated_asset_rule_or_export_check
        - isolated_state_semantics_without_page_context
        - non_visual_metadata_only
  qa_only:
    route_to: qa_gate
    global_style_generation_enabled: disabled
  style_brief_only:
    route_to: style_brief_resolution_only
    global_style_generation_enabled: disabled
  production_constraints_only:
    route_to: production_constraints_only
    global_style_generation_enabled: disabled
  metadata_only:
    route_to: metadata_only
    global_style_generation_enabled: disabled
```

`route_to` 的图片生成值仍以 05 当前支持的 `background_generation | panel_programmatic_draw | full_ui_skin | panel_demo | state_guidance | qa_gate` 为准。`page_color_rule_only` 是 05 可输出的非图片规则物料，只能生成 `page_color_rule`，不得触发背景、面板或面板饰品生成。`style_brief_resolution_only`、`production_constraints_only` 和 `metadata_only` 是 01 内部终止类型，不进入 05。

### Surface Registry

新增风格表面时，先加入 surface registry，再决定默认是否属于 `global_page_style_default`。不要为了每个表面增加一套独立分诊流程。

```yaml
surface_registry:
  background:
    owner: docs/05-material-generation.md#背景图规则
    default_in_global_page_style: true
    required_inputs:
      - target_background_dimensions
      - target_page_or_screen
      - style_brief_resolution
  panel_system:
    owner: docs/05-material-generation.md#程序化面板规则
    default_in_global_page_style: true
    required_inputs:
      - panel_candidate_inventory
      - style_brief_resolution
  panel_accent:
    owner: docs/05-material-generation.md#固定局部装饰
    default_in_global_page_style: true
    required_inputs:
      - style_brief_resolution
      - panel_candidate_inventory
  page_color_system:
    owner: docs/04-visual-hierarchy-brief.md#七-色彩角色规则
    default_in_global_page_style: true
    required_inputs:
      - target_page_or_screen
      - primary_focus_context
      - style_brief_resolution
    output_kind: non_image_rule
  state_system:
    owner: docs/05-material-generation.md#状态视觉规则
    default_in_global_page_style: conditional
    required_inputs:
      - state_requirements
      - color_roles_when_available
```

## 三、意图判定

### 先排除非生成请求

以下请求不默认生成新视觉物料：

```yaml
non_generation_intents:
  qa_only:
    examples:
      - 验收这张背景
      - 看看这个素材合不合格
      - 检查现在结果哪里不对
    route_to: qa_gate
  style_brief_only:
    examples:
      - 查询某游戏风格
      - 帮我补 style_brief
      - 只判断这个游戏属于什么风格
    stop_after: style_brief_resolution
  production_constraints_only:
    examples:
      - 这个资源能不能在 Maker 实现
      - 检查是否超预算
      - 只看 9-slice 约束
    stop_after: production_constraints_check
  metadata_only:
    examples:
      - 只改命名
      - 补 metadata
      - 更新说明文字
    stop_after: metadata_update
```

### 默认全局风格请求

以下表达全部默认进入 `global_page_style`：

```yaml
global_style_intents:
  full_style:
    examples:
      - 整体换成 XX 风格
      - 套一版 XX 游戏风格
      - 这个页面做成 XX
  component_mentioned_without_only:
    examples:
      - 面板换成 XX 风格
      - 背景做成 XX 风格
      - 按钮状态改成 XX 风格
      - 卡片像 XX 游戏
    interpretation: global_page_style_with_requested_focus_elements
  state_mentioned_without_only:
    examples:
      - 选中态做成 XX 风格
      - warning 用 XX 游戏感觉
    interpretation: global_page_style_with_state_requirements
  demo_without_only:
    examples:
      - 生成一个 XX 风格面板 demo
    interpretation: full_ui_skin_demo_context
```

默认全局风格必须输出：

```yaml
global_style_required_outputs:
  target_image_assets:
    - background_tool.png
    - panel_corner_accent.png | panel_accent.png
  target_implementations:
    - programmatic_panel
  target_style_rules:
    - page_color_rule
  optional_when_requested_or_conflicting:
    - state_guidance
```

`target_image_assets` 只记录需要生成或导出的图片资产，受 03 的图片白名单和数量预算限制。`programmatic_panel` 属于代码 / 渲染实现产物，必须放在 `target_implementations`，不得混入图片资产预算。`page_color_rule` 是非图片视觉规则，不受 03 的图片白名单数量限制；它用于把 02 的颜色、材质、情绪和 04 的 `color_roles` 转成页面级配色职责，例如背景底色、容器底色、主行动色、辅助色、正文色、弱文字色和状态色。

### 显式局部元素请求

以下表达进入 `element_style_patch`：

```yaml
element_style_patch_intents:
  background_only:
    examples:
      - 只换背景成 XX
      - 背景单独出一张，其他不动
    route_to: background_generation
  panel_only:
    examples:
      - 只改这些面板
      - 仅给弹窗面板出 programmatic panel 规则
    route_to: panel_programmatic_draw
  panel_demo_only:
    examples:
      - 只生成 XX 风格面板 demo
      - 单独验证面板 demo
    route_to: panel_demo
  state_only:
    examples:
      - 只改 selected 状态
      - warning 颜色单独调整，不动背景和面板
    route_to: state_guidance
  color_only:
    examples:
      - 只调页面配色，不出新背景图
      - 保持背景和面板不变，只改色彩角色
    route_to: page_color_rule_only
    global_style_generation_enabled: disabled
    material_output: non_image_page_color_rule
    target_style_rules:
      - page_color_rule
    target_image_assets: []
    target_implementations: []
```

局部请求必须显式记录 `preserve_surfaces`，例如 `background`、`panel_system`、`panel_accent`、`page_color_system` 或 `state_system`。当局部修改仍可能影响可读性、焦点或状态语义时，仍需进入 04。

### 迭代优化请求

如果用户在已有结果上说“太重 / 太花 / 不像 / 层级不清 / 文字看不清 / 保留方向但弱一点”，01 应判断为 `iterative_style_refinement`。

```yaml
iterative_refinement_policy:
  global_by_default:
    applies_when:
      - user_feedback_does_not_explicitly_limit_to_one_element
    mode: global_page_style
    visual_hierarchy_mode: iterative_style_refinement
    current_visual_audit_required: true
  local_when_explicit:
    applies_when:
      - user_says_only_background
      - user_says_only_panel
      - user_says_do_not_touch_other_surfaces
    mode: element_style_patch
    visual_hierarchy_mode: iterative_style_refinement
    current_visual_audit_required: true
```

迭代模式不能跳过 `current_visual_audit` 后直接推翻当前结果。

## 四、style_brief 复用规则

只要进入视觉风格 workflow，必须先做 `style_brief` resolution，但不一定重新执行 02。

### resolution 输出

```yaml
style_brief_resolution:
  requested_game_name:
  requested_alias:
  resolved_style_brief_ref:
  resolution_status: reuse_existing | run_02_style_backfill | run_02_style_discovery | blocked_needs_clarification
  confidence: high | medium | low
  reason:
```

### 可以复用已有 style_brief 的条件

同时满足以下条件时，01 可以复用已有 `style_brief`，不重新执行 02：

- `game_name` 或 `alias` 精确命中 02。
- 当前需求仍是同一游戏。
- 02 规则版本没有变化，或当前任务明确接受现有 `style_brief`。
- `style_brief` 包含程序化面板所需字段，尤其是 `colors`、`materials`、`shape_language`、`ui_translation.panel_base_style_hint`、`forbidden_mistakes`。
- `style_brief` 包含页面配色所需字段，尤其是 `colors`、`mood`、`materials`、`forbidden_mistakes`、`ui_translation.state`、`ui_translation.text`。
- `style_brief` 包含 Style Specificity Contract 所需字段，尤其是 `style_archetype`、`specific_differentiators`、`signature_semantics`、`background_identity_anchors`、`anti_generic_guardrails`、`visual_identity_test`。
- `source.confidence` 或本次 resolution `confidence` 不是 `low`。

### 必须重新执行或回到 02 的条件

出现以下任一情况时，01 不能直接进入 05：

- 游戏名不明确或有歧义。
- 02 中没有该游戏或 alias。
- `style_brief` 缺少 `ui_translation.panel_base_style_hint`，需要回到 02 执行旧条目补全。
- `style_brief` 缺少 `ui_translation.state` 或 `ui_translation.text`，且本次需要页面配色或状态规则，需要回到 02 执行旧条目补全。
- `style_brief` 缺少 Style Specificity Contract 字段，需要回到 02 执行旧条目补全。
- 用户更换了游戏。
- 用户指出风格不像目标游戏。
- 规则版本变化导致旧 `style_brief` 不可信。
- 当前需求需要程序化面板，但现有 `style_brief` 缺少 `colors`、`materials`、`shape_language` 或 `forbidden_mistakes`。

### 复用边界

01 只能引用或要求补全 `style_brief`，不能在本文件内新增具体游戏条目，不能把 Style Discovery 结论直接写成 02 的替代品。

## 五、页面上下文采集

进入 `global_page_style` 时，01 必须采集页面级上下文，因为默认会影响背景、面板装饰和整个页面配色。

```yaml
required_context_for_global_page_style:
  target_page_or_screen:
  primary_focus_context:
    page_goal:
    primary_focus:
    secondary_focus:
    known_content_regions:
      - string
  target_background_dimensions:
    source: user_specified | design_artboard | current_background_node | project_viewport | screenshot_region | unknown
    width_px:
    height_px:
    device_class: mobile | desktop | tablet | unknown
    orientation: portrait | landscape | square | unknown
    reason:
  minimal_visual_context:
    required_for_04: true
    screenshot_ref:
    current_implementation_ref:
    existing_assets_ref:
    user_feedback:
```

当请求会生成 `background_tool.png` 时，必须已有 `target_background_dimensions.width_px` 和 `target_background_dimensions.height_px`。只知道是手机、电脑或平板但没有具体宽高时，不得编造默认尺寸，必须把 `missing_target_background_dimensions` 写入 `blocked_by` 或 `open_questions`。

进入 `element_style_patch` 时，01 只采集该元素需要的上下文；但如果局部修改会影响页面主次、背景强度、文字可读性或状态颜色冲突，仍必须采集足够上下文并进入 04。

## 六、组件类型穷举和识别规则

01 必须在进入 05 前输出 `panel_candidate_inventory`。在 `global_page_style` 中，该 inventory 用于判断哪些组件适合应用程序化面板皮肤，哪些只能走轻量 surface、状态层、局部 accent、UI primitive 或跳过。在 `element_style_patch` 中，该 inventory 只覆盖显式指定的目标元素和必要上下文元素。

### 组件类型 taxonomy

```yaml
component_taxonomy:
  safe_panel_components:
    - medium_panel
    - popup_content_panel
    - settings_panel
    - card_panel
    - item_card
    - character_card
    - selection_card
    - list_container
    - form_panel
    - side_panel
    - dialog_sheet
    - content_surface

  caution_components:
    - compact_card
    - wide_banner_panel
    - nested_panel
    - long_info_panel
    - small_info_card

  not_panel_components:
    - button
    - thin_button
    - icon_button
    - status_bar
    - progress_bar
    - toolbar
    - tab
    - divider
    - pill_label
    - badge
    - icon
    - avatar
    - checkbox
    - switch
    - input
    - text_only_block
    - navigation_item
    - top_strip
    - state_marker
```

### 判断依据

- 命名证据：`panel`、`card`、`dialog`、`modal`、`popup`、`sheet`、`surface`、`container`、`content`、`settings`、`list`、`item`。
- 样式证据：`background`、`border`、`borderRadius`、`shadow`、`padding`、`surface fill`。
- 布局证据：承载正文、列表、表单、卡片内容，而不是单个操作。
- 交互证据：纯容器优先；按钮、状态条、标签、进度条不能当通用面板。
- 尺寸证据：过矮、过长、只承载状态 / 进度的组件不套用普通 panel skin。
- 范围证据：组件名出现但没有“只 / 仅 / 保持其他不变”等限定时，不构成局部范围。
- 不允许只因为有圆角或背景色就判断为 panel。

### panel_candidate_inventory 输出格式

```yaml
panel_candidate_inventory:
  - component_name:
    file_path:
    detected_component_type:
    evidence:
      naming:
      layout_role:
      style_props:
      children_content:
      interaction:
      size_or_aspect_ratio:
      scope_marker:
    safe_to_apply_panel_skin: true | false
    route:
      - programmatic_panel
      - light_surface
      - panel_accent
      - state_overlay
      - color_rule_only
      - skip
    reason:
```

### 识别约束

- `safe_panel_components` 可以进入 `programmatic_panel`，但仍需遵守 03 的尺寸和 QA 要求。
- `caution_components` 必须说明风险；无法确认尺寸、内容密度或层级时，先保守标记为 `light_surface`、`state_overlay` 或 `skip`。
- `not_panel_components` 不得套用通用 panel skin；如用户要求状态变化，只允许进入 `state_overlay` 或由 05 给出状态视觉建议。
- `panel_accent` 只能用于 03 / 05 允许的固定角部局部装饰。全局默认最多生成 1 个；不得把按钮、徽章、状态、图标或 IP 物件塞进面板装饰。

## 七、target_components 和 page_color_rule 输出

当 `style_application_scope.mode == global_page_style` 时，01 必须把安全候选转成 `target_components`，并要求页面级配色输出。

```yaml
target_components:
  - component_name:
    component_type:
    role: primary | secondary | supporting | decorative | unknown
    width:
    height:
    state_requirements:
      - normal
      - hover
      - pressed
      - selected
      - disabled
      - locked
      - warning
      - error
      - success
      - new
      - reward
```

```yaml
page_color_requirements:
  required_when:
    - style_application_scope.mode == global_page_style
    - user_explicitly_requests_color_only
  role_slots:
    - page_background_base
    - background_scrim_or_overlay
    - primary_container
    - secondary_container
    - primary_action
    - secondary_action
    - text_primary
    - text_secondary
    - text_disabled
    - selected
    - warning
    - error
    - success
  must_respect:
    - style_brief.colors
    - style_brief.mood
    - style_brief.ui_translation.state
    - style_brief.ui_translation.text
    - visual_hierarchy_brief.color_roles_when_required
```

如果没有源码或设计稿尺寸，01 只能填 `unknown` 并在 `workflow_request.open_questions` 中记录缺口；不能编造尺寸或实现参数。

当 `style_application_scope.mode == element_style_patch` 时，`target_components` 只包含显式目标元素；必须同时写明 `preserve_surfaces` 和 `allowed_outputs`。

## 八、workflow_request 输出契约

01 的最终产物是初始结构化 `workflow_request`。当 `visual_hierarchy_scope == required` 时，04 根据该请求产出 `visual_hierarchy_brief`，并在进入 05 前补全 `visual_hierarchy_brief_ref`；当 `visual_hierarchy_scope == not_required` 时，01 直接 pass-through 给 05。05 不应重新猜测用户意图。

```yaml
workflow_request:
  request_id:
  original_user_intent:
  normalized_intent:
  style_application_scope:
    mode: global_page_style | element_style_patch | qa_only | style_brief_only | production_constraints_only | metadata_only
    defaulted_to_global: true | false
    scope_reason:
    explicit_scope_markers: []
    requested_focus_elements: []
    preserve_surfaces: []
    global_style_surfaces:
      background: required | not_required
      panel_system: required | not_required
      panel_accent: required | not_required
      page_color_system: required | not_required
      state_system: required | conditional | not_required
    element_patch:
      target_elements: []
      allowed_outputs: []
      preserve_other_surfaces: true | false
  global_style_generation_enabled: enabled | disabled
  route_to: background_generation | panel_programmatic_draw | full_ui_skin | panel_demo | state_guidance | qa_gate | page_color_rule_only | style_brief_resolution_only | production_constraints_only | metadata_only
  target_page_or_screen:
  target_background_dimensions:
    required_when_background_generated: true
    source: user_specified | design_artboard | current_background_node | project_viewport | screenshot_region | unknown
    width_px:
    height_px:
    device_class: mobile | desktop | tablet | unknown
    orientation: portrait | landscape | square | unknown
    reason:
  primary_focus_context:
    page_goal:
    primary_focus:
    secondary_focus:
    known_content_regions:
      - string
  minimal_visual_context:
    required_for_04: true | false
    screenshot_ref:
    current_implementation_ref:
    existing_assets_ref:
    user_feedback:
  style_brief_resolution:
    requested_game_name:
    requested_alias:
    resolved_style_brief_ref:
    resolution_status: reuse_existing | run_02_style_backfill | run_02_style_discovery | blocked_needs_clarification
    confidence: high | medium | low
    reason:
  visual_hierarchy_mode:
    required_when: visual_hierarchy_scope == required
    value: greenfield_style_application | iterative_style_refinement | null
  current_visual_audit_required: true | false
  visual_hierarchy_scope: required | not_required
  visual_hierarchy_brief_ref: null | generated_by_04_before_05
  visual_hierarchy_skip_reason:
  target_image_assets:
    required_when_global_page_style:
      - background_tool.png
      - panel_corner_accent.png | panel_accent.png
    required_when_element_style_patch:
      - derived_image_assets_from_element_patch.allowed_outputs
    qa_gate: []
  target_implementations:
    required_when_global_page_style:
      - programmatic_panel
    required_when_element_style_patch:
      - derived_implementations_from_element_patch.allowed_outputs
    qa_gate: []
  target_style_rules:
    required_when_global_page_style:
      - page_color_rule
    required_when_color_only_patch:
      - page_color_rule
  page_color_requirements:
    required: true | false
    role_slots: []
    preserve_existing_color_roles: true | false
  panel_candidate_inventory: []
  target_components: []
  qa_requirements:
    procedural_panel_required_sizes:
      - 800x300
      - 300x500
      - 660x440
      - 520x200
    qa_only_no_generation: true | false
  assumptions: []
  open_questions: []
  blocked_by:
    - missing_or_ambiguous_game_name
    - missing_style_brief
    - low_confidence_style_brief
    - missing_panel_base_style_hint
    - missing_specificity_fields
    - missing_current_visual_audit
    - missing_target_page_or_screen
    - missing_target_background_dimensions
    - missing_primary_focus_context
    - missing_minimal_visual_context
    - missing_component_inventory
    - missing_required_component_dimensions
```

字段归属：

- 01 设置 `style_application_scope`，并记录是否由默认规则归入全局。
- 01 设置 `global_style_generation_enabled`：全局页面风格请求为 `enabled`；显式局部请求、QA 和非生成请求为 `disabled`。
- 01 设置 `visual_hierarchy_scope`。
- `visual_hierarchy_mode` 仅在 `visual_hierarchy_scope == required` 时必填；`visual_hierarchy_scope == not_required` 时应为 `null`。
- 01 设置 `current_visual_audit_required`。
- 01 采集 `target_page_or_screen`、`primary_focus_context` 和必要的 `minimal_visual_context`，供 04 判断页面视觉层级。
- 当请求会生成 `background_tool.png` 时，01 必须采集 `target_background_dimensions`。尺寸必须来自当前目标界面或背景容器；如果只能判断是手机或电脑但没有具体宽高，不能编造默认尺寸，必须把 `missing_target_background_dimensions` 写入 `blocked_by` 或 `open_questions`。
- 当 `visual_hierarchy_scope == required` 时，01 不生成 `visual_hierarchy_brief_ref`，也不消费它；01 初始输出时该字段应为空、`null` 或标记为待 04 补全。
- 04 在完成 `current_visual_audit`（如需要）和 `visual_hierarchy_brief` 后生成 `visual_hierarchy_brief_ref`。
- 当 `visual_hierarchy_scope == not_required` 时，01 直接输出 `visual_hierarchy_scope: not_required`、`visual_hierarchy_brief_ref: null`、`visual_hierarchy_skip_reason` 给 05。
- 05 消费视觉层级输入时按 `visual_hierarchy_scope` 分支：`required` 时消费 04 生成的 `visual_hierarchy_brief_ref`，`not_required` 时按 01 的 `visual_hierarchy_skip_reason` 跳过视觉层级输入。

## 九、路由到 05 的最低条件

进入 05 前必须具备：

- `style_application_scope.mode` 已确定。
- `route_to` 已确定；如果请求会进入 05 图片生成，必须属于 05 支持的图片生成 route；如果 `route_to == page_color_rule_only`，只允许进入 05 的非图片规则输出分支。
- 如果 `route_to == page_color_rule_only`：只输出非图片 `page_color_rule`，不得触发 05 图片生成，也不得生成 `background_tool.png`、`programmatic_panel` 或面板饰品。
- `style_brief_resolution` 已完成；若为 `run_02_style_backfill`、`run_02_style_discovery` 或 `blocked_needs_clarification`，不得直接生成物料，必须先由 02 输出补全或发现后的 `style_brief`。
- `visual_hierarchy_scope` 已确定。
- 如果 `style_application_scope.mode == global_page_style`：`route_to` 必须为 `full_ui_skin`，`global_style_generation_enabled` 必须为 `enabled`，并且 `target_image_assets` 必须包含 `background_tool.png` 和 1 个 `panel_corner_accent.png | panel_accent.png`，`target_implementations` 必须包含 `programmatic_panel`。
- 如果 `style_application_scope.mode == global_page_style`：`target_style_rules` 必须包含 `page_color_rule`。
- 如果 `style_application_scope.mode == element_style_patch`：必须写明 `explicit_scope_markers`、`target_elements`、`allowed_outputs` 和 `preserve_surfaces`。
- 如果 `visual_hierarchy_scope == required`：`visual_hierarchy_mode` 已确定；若为 `iterative_style_refinement`，必须设置 `current_visual_audit_required: true`。
- 如果 `visual_hierarchy_scope == required`：进入 04 的请求已提供 `target_page_or_screen` 或 `primary_focus_context`；背景、状态、QA 这类依赖当前视觉关系的请求还必须提供 `minimal_visual_context`。
- 如果 `visual_hierarchy_scope == required`：04 已生成 `visual_hierarchy_brief_ref`；该字段不是 01 的产物。
- 如果 `visual_hierarchy_scope == not_required`：01 已输出 `visual_hierarchy_scope: not_required`、`visual_hierarchy_brief_ref: null`、`visual_hierarchy_skip_reason`。
- 如果会生成 `background_tool.png`：必须已有 `target_background_dimensions.width_px` 和 `target_background_dimensions.height_px`；只知道 mobile / desktop / tablet 而没有具体宽高时，不得进入 05 生成背景。
- 面板相关需求已输出 `panel_candidate_inventory`。
- 需要实际替换组件时已输出 `target_components`。
- demo 面板已带上 03 要求的程序化面板 QA 尺寸。
- QA-only 请求已明确 `qa_only_no_generation: true`。

## 十、越权边界

01 不得做以下事情：

- 不新增、改写或替代 02 的具体游戏风格条目。
- 不定义图片数量、资源预算、Maker 能力、尺寸边界或 fallback 条件。
- 不编写背景 prompt、局部装饰 prompt、页面配色数值或 `panel_render_recipe`。
- 不指定颜色数值、半径、描边宽度、glow 参数、切片参数或渲染 API。
- 不把按钮、状态条、标签、进度条、输入框、导航项等误判为通用面板。
- 不因为用户提到某个元素就默认缩小范围；只有显式局部限定才进入 `element_style_patch`。
- 不在 QA-only、style-brief-only、constraints-only 或 metadata-only 请求中默认生成新素材。
