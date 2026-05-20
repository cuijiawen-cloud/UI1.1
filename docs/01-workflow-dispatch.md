# 01 Workflow Dispatch

> 角色：01 Workflow Dispatch / 分诊台 / Intent Router。负责解析用户自然语言意图，决定是否需要进入 `02-style-knowledge.md`，复用同一游戏已有 `style_brief`，盘点目标组件，并把结构化 `workflow_request` 交给后续链路。

本文件只负责路由和输入整理，不负责写入风格知识、生产硬约束、prompt 模板、生成 recipe 或具体实现参数。

## 一、入口流程

所有风格化需求先进入 01。当前临时关闭细分分诊：01 仍负责采集输入和完成 `style_brief` resolution，但不再因为用户只提到“面板 / 卡片 / 背景 / 状态”而缩小生成范围。

只要用户提出“换成 / 做成 / 生成 / 套用 XX 风格”这类风格化生成请求，除非用户明确说只做 QA、只验收、只查询规则或只改非视觉 metadata，否则统一视为完整风格输出，并要求背景和面板饰品一起进入后续链路。

主链路固定为：

```text
00 Index -> 01 Dispatch -> 02 Style -> 03 Constraints -> 04 Visual Hierarchy -> 05 Material Generation
```

01 只决定是否需要 02、`visual_hierarchy_scope` 是否为 `required`、以及是否必须做 `current_visual_audit`。在临时关闭细分分诊期间，所有风格化生成请求都输出 `style_generation_bypass: enabled`，并把 `route_to` 归一为 `full_ui_skin`。只有 `visual_hierarchy_scope == required` 时，才需要设置 `visual_hierarchy_mode` 并进入 04；当 `visual_hierarchy_scope == not_required` 时，01 直接 pass-through 给 05。

### 临时分诊旁路

```yaml
style_generation_bypass:
  status: enabled
  applies_when:
    - user_requests_make_or_apply_style
    - user_mentions_panel_card_popup_background_state_but_intent_is_style_generation
  route_to: full_ui_skin
  required_assets:
    - background_tool.png
    - programmatic_panel
    - panel_corner_accent.png | panel_accent.png
  optional_assets:
    - state_guidance
  do_not_apply_when:
    - qa_only
    - production_constraints_only
    - style_brief_resolution_only
    - non_visual_or_metadata_only_request
```

旁路启用时，旧的 `panel_programmatic_draw`、`background_generation`、`panel_demo` 和 `state_guidance` 只作为历史路由语义或 QA 归因线索，不作为默认输出范围裁剪依据。

### 意图路由

```yaml
intent_routing:
  panel_style_change:
    user_says: "把面板换成 XX 风格"
    steps:
      - resolve_style_brief
      - identify_target_page_or_screen
      - collect_primary_focus_context
      - build_panel_candidate_inventory
      - output_workflow_request
    route_to: full_ui_skin
    target_assets:
      - background_tool.png
      - programmatic_panel
      - panel_corner_accent.png | panel_accent.png

  background_style_change:
    user_says: "背景换成 XX 风格"
    steps:
      - resolve_style_brief
      - identify_target_page_or_screen
      - collect_minimal_visual_context
      - output_workflow_request
    route_to: full_ui_skin
    target_assets:
      - background_tool.png
      - programmatic_panel
      - panel_corner_accent.png | panel_accent.png

  full_game_style_skin:
    user_says: "整体换成 XX 游戏风格"
    steps:
      - resolve_style_brief
      - identify_target_page_or_screen
      - collect_primary_focus_context
      - build_panel_candidate_inventory
      - output_workflow_request
    route_to: full_ui_skin
    target_assets:
      - background_tool.png
      - programmatic_panel
      - panel_corner_accent.png | panel_accent.png
      - state_guidance

  state_style_change:
    user_says: "把选中态 / 禁用态 / warning / error 换成 XX 风格"
    steps:
      - resolve_style_brief
      - identify_state_requirements
      - decide_visual_hierarchy_need
      - collect_minimal_visual_context_if_04_required
      - output_workflow_request
    route_to: full_ui_skin
    target_assets:
      - background_tool.png
      - programmatic_panel
      - panel_corner_accent.png | panel_accent.png
      - state_guidance
    default_generation: true
    requires_04_when:
      - state_visibility_depends_on_page_background_or_panel_hierarchy
      - selected_warning_error_may_conflict_with_existing_color_roles
      - user_feedback_mentions_readability_focus_or_hierarchy
    skip_04_when:
      - explicitly_isolated_state_semantics_only_without_style_generation
      - no_page_or_screen_context_available
      - no_visual_hierarchy_or_readability_judgement_requested
    direct_pass_through_to_05:
      visual_hierarchy_scope: not_required
      visual_hierarchy_brief_ref: null
      visual_hierarchy_skip_reason:

  panel_demo:
    user_says: "生成 XX 风格面板 demo"
    steps:
      - resolve_style_brief
      - define_demo_target_page_or_screen
      - define_demo_primary_focus_context
      - build_demo_target_components
      - output_workflow_request
    route_to: full_ui_skin
    target_assets:
      - background_tool.png
      - programmatic_panel
      - panel_corner_accent.png | panel_accent.png
    required_qa_sizes_from_03:
      - 800x300
      - 300x500
      - 660x440
      - 520x200

  qa_only:
    user_says: "用户上传素材让验收"
    steps:
      - identify_submitted_assets
      - identify_relevant_rules
      - decide_visual_hierarchy_need
      - collect_current_visual_context_if_04_required
      - output_qa_request
    route_to: qa_gate
    default_generation: false
    requires_04_when:
      - qa_scope_includes_page_hierarchy
      - qa_scope_includes_background_strength
      - qa_scope_includes_component_focus_or_readability
      - user_feedback_mentions_too_heavy_too_weak_unclear_or_not_prominent
    skip_04_when:
      - qa_scope_is_asset_export_only
      - qa_scope_is_transparency_size_naming_or_budget_only
      - submitted_asset_is_not_shown_in_page_context
    direct_pass_through_to_05:
      visual_hierarchy_scope: not_required
      visual_hierarchy_brief_ref: null
      visual_hierarchy_skip_reason:
```

### 01 的默认判断

- 用户只要提出“换成 / 做成 / 生成 / 套用 XX 风格”且不是 QA-only 或 metadata-only，默认按 `full_ui_skin` 分诊，并输出 `style_generation_bypass: enabled`。
- 用户即使明确提到“背景 / 面板 / panel / 卡片 / 弹窗面板 / 面板主体 / 状态”，只要意图是做风格化生成，也不再裁剪生成范围；`target_assets` 必须包含 `background_tool.png`、`programmatic_panel` 和 1 个 `panel_corner_accent.png | panel_accent.png`。
- 01 不再把 `panel_programmatic_draw` 作为风格请求的默认 route；只有后续 QA 归因、历史兼容或明确非旁路任务才可引用该 route。
- 背景和面板饰品是风格承载的默认资产，不需要用户额外说“整体 / 全套 / 背景也一起”。
- 用户只要求验收、检查、QA、看看合不合格时，进入 `qa_gate`，不默认生成新素材。
- 用户要求状态、选中态、禁用态、warning 或 error 且同时要求风格化生成时，仍按 `full_ui_skin` 输出，并把状态需求写入 `target_components.state_requirements`。
- 用户要求 demo 时，也按完整风格输出背景和面板饰品，但必须把 03 的程序化面板多尺寸 QA 列入请求，不把 demo 当成最终业务组件替换。
- 所有进入 04 的路由，必须至少采集 `target_page_or_screen` 或 `primary_focus_context`；背景、状态和 QA 路由还应采集最小视觉上下文，避免 04 在没有页面主次依据时判断背景强度、状态冲突或可读性。
- `state_style_change` 和 `qa_only` 可以跳过视觉层级判断；跳过时由 01 直接 pass-through 给 05，并且只输出 `visual_hierarchy_scope: not_required`、`visual_hierarchy_brief_ref: null`、`visual_hierarchy_skip_reason`。一旦涉及页面层级、背景强度、焦点突出或可读性判断，就必须进入 04 的正常 brief 流程。

## 二、style_brief 复用规则

只要进入风格化 workflow，必须先做 `style_brief` resolution，但不一定重新执行 02。

### resolution 输出

```yaml
style_brief_resolution:
  requested_game_name:
  requested_alias:
  resolved_style_brief_ref:
  resolution_status: reuse_existing | run_02_style_discovery | blocked_needs_clarification
  confidence: high | medium | low
  reason:
```

### 可以复用已有 style_brief 的条件

同时满足以下条件时，01 可以复用已有 `style_brief`，不重新执行 02：

- `game_name` 或 `alias` 精确命中 02。
- 当前需求仍是同一游戏。
- 02 规则版本没有变化，或当前任务明确接受现有 `style_brief`。
- `style_brief` 包含程序化面板所需字段，尤其是 `colors`、`materials`、`shape_language`、`ui_translation.panel_base_style_hint`、`forbidden_mistakes`。
- `style_brief` 包含 Style Specificity Contract 所需字段，尤其是 `style_archetype`、`specific_differentiators`、`signature_semantics`、`background_identity_anchors`、`anti_generic_guardrails`、`visual_identity_test`。
- `source.confidence` 或本次 resolution `confidence` 不是 `low`。

### 必须重新执行或回到 02 的条件

出现以下任一情况时，01 不能直接进入 05：

- 游戏名不明确或有歧义。
- 02 中没有该游戏或 alias。
- `style_brief` 缺少 `ui_translation.panel_base_style_hint`。
- `style_brief` 缺少 Style Specificity Contract 字段。
- 用户更换了游戏。
- 用户指出风格不像目标游戏。
- 规则版本变化导致旧 `style_brief` 不可信。
- 当前需求需要程序化面板，但现有 `style_brief` 缺少 `colors`、`materials`、`shape_language` 或 `forbidden_mistakes`。

### 复用边界

01 只能引用或要求补全 `style_brief`，不能在本文件内新增具体游戏条目，不能把风格 discovery 结论直接写成 02 的替代品。

## 三、组件类型穷举和识别规则

01 必须在执行 05 前输出 `panel_candidate_inventory`。该 inventory 用于判断哪些组件适合应用程序化面板皮肤，哪些只能走状态层、局部 accent、UI primitive 或跳过。

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
    safe_to_apply_panel_skin: true | false
    route:
      - programmatic_panel
      - panel_accent
      - state_overlay
      - skip
    reason:
```

### 识别约束

- `safe_panel_components` 可以进入 `programmatic_panel`，但仍需遵守 03 的尺寸和 QA 要求。
- `caution_components` 必须说明风险；无法确认尺寸、内容密度或层级时，先保守标记为 `skip` 或仅 `state_overlay`。
- `not_panel_components` 不得套用通用 panel skin；如用户要求状态变化，只允许进入 `state_overlay` 或由 05 给出状态视觉建议。
- `panel_accent` 只能用于 03 / 05 允许的固定角部局部装饰。临时旁路启用时默认生成 1 个；不得把按钮、徽章、状态、图标或 IP 物件塞进面板装饰。

## 四、target_components 输出

当 `route_to` 为 `full_ui_skin`，或当前请求需要实际替换组件时，01 必须把安全候选转成 `target_components`。临时旁路启用后，原本会落到 `panel_programmatic_draw` / `background_generation` / `panel_demo` / `state_guidance` 的风格化生成请求都应先归一为 `full_ui_skin`。`qa_gate` 可按 05 输入契约省略或传空，并在 `open_questions` / `assumptions` 中记录原因。

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

如果没有源码或设计稿尺寸，01 只能填 `unknown` 并在 `workflow_request.open_questions` 中记录缺口；不能编造尺寸或实现参数。

## 五、workflow_request 输出契约

01 的最终产物是初始结构化 `workflow_request`。当 `visual_hierarchy_scope == required` 时，04 根据该请求产出 `visual_hierarchy_brief`，并在进入 05 前补全 `visual_hierarchy_brief_ref`；当 `visual_hierarchy_scope == not_required` 时，01 直接 pass-through 给 05。05 不应重新猜测用户意图。

```yaml
workflow_request:
  request_id:
  original_user_intent:
  normalized_intent:
  style_generation_bypass: enabled | disabled
  route_to: background_generation | panel_programmatic_draw | full_ui_skin | panel_demo | state_guidance | qa_gate
  target_page_or_screen:
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
    resolution_status: reuse_existing | run_02_style_discovery | blocked_needs_clarification
    confidence: high | medium | low
    reason:
  visual_hierarchy_mode:
    required_when: visual_hierarchy_scope == required
    value: greenfield_style_application | iterative_style_refinement | null
  current_visual_audit_required: true | false
  visual_hierarchy_scope: required | not_required
  visual_hierarchy_brief_ref: null | generated_by_04_before_05
  visual_hierarchy_skip_reason:
  target_assets:
    required_by_route:
      background_generation:
        - background_tool.png
        - programmatic_panel
        - panel_corner_accent.png | panel_accent.png
      panel_programmatic_draw:
        - background_tool.png
        - programmatic_panel
        - panel_corner_accent.png | panel_accent.png
      panel_demo:
        - background_tool.png
        - programmatic_panel
        - panel_corner_accent.png | panel_accent.png
      full_ui_skin:
        - background_tool.png
        - programmatic_panel
        - panel_corner_accent.png | panel_accent.png
        - state_guidance
      state_guidance:
        - background_tool.png
        - programmatic_panel
        - panel_corner_accent.png | panel_accent.png
        - state_guidance
      qa_gate: []
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
    - missing_primary_focus_context
    - missing_minimal_visual_context
    - missing_component_inventory
    - missing_required_component_dimensions
```

字段归属：

- 01 设置 `visual_hierarchy_scope`。
- `visual_hierarchy_mode` 仅在 `visual_hierarchy_scope == required` 时必填；`visual_hierarchy_scope == not_required` 时应为 `null`。
- 01 设置 `current_visual_audit_required`。
- 01 采集 `target_page_or_screen`、`primary_focus_context` 和必要的 `minimal_visual_context`，供 04 判断页面视觉层级。
- 当 `visual_hierarchy_scope == required` 时，01 不生成 `visual_hierarchy_brief_ref`，也不消费它；01 初始输出时该字段应为空、`null` 或标记为待 04 补全。
- 04 在完成 `current_visual_audit`（如需要）和 `visual_hierarchy_brief` 后生成 `visual_hierarchy_brief_ref`。
- 当 `visual_hierarchy_scope == not_required` 时，01 直接输出 `visual_hierarchy_scope: not_required`、`visual_hierarchy_brief_ref: null`、`visual_hierarchy_skip_reason` 给 05。
- 05 消费视觉层级输入时按 `visual_hierarchy_scope` 分支：`required` 时消费 04 生成的 `visual_hierarchy_brief_ref`，`not_required` 时按 01 的 `visual_hierarchy_skip_reason` 跳过视觉层级输入。

### 路由到 05 的最低条件

进入 05 前必须具备：

- `route_to` 已确定。
- `style_brief_resolution` 已完成；若为 `run_02_style_discovery` 或 `blocked_needs_clarification`，不得直接生成物料。
- `visual_hierarchy_scope` 已确定。
- 如果 `visual_hierarchy_scope == required`：`visual_hierarchy_mode` 已确定；若为 `iterative_style_refinement`，必须设置 `current_visual_audit_required: true`。
- 如果 `visual_hierarchy_scope == required`：进入 04 的请求已提供 `target_page_or_screen` 或 `primary_focus_context`；背景、状态、QA 这类依赖当前视觉关系的请求还必须提供 `minimal_visual_context`。
- 如果 `visual_hierarchy_scope == required`：04 已生成 `visual_hierarchy_brief_ref`；该字段不是 01 的产物。
- 如果 `visual_hierarchy_scope == not_required`：01 已输出 `visual_hierarchy_scope: not_required`、`visual_hierarchy_brief_ref: null`、`visual_hierarchy_skip_reason`。
- 面板相关需求已输出 `panel_candidate_inventory`。
- 需要实际替换组件时已输出 `target_components`。
- demo 面板已带上 03 要求的程序化面板 QA 尺寸。
- QA-only 请求已明确 `qa_only_no_generation: true`。

## 六、越权边界

01 不得做以下事情：

- 不新增、改写或替代 02 的具体游戏风格条目。
- 不定义图片数量、资源预算、Maker 能力、尺寸边界或 fallback 条件。
- 不编写背景 prompt、局部装饰 prompt 或 `panel_render_recipe`。
- 不指定颜色数值、半径、描边宽度、glow 参数、切片参数或渲染 API。
- 不把按钮、状态条、标签、进度条、输入框、导航项等误判为通用面板。
- 不在 QA-only 请求中默认生成新素材。
