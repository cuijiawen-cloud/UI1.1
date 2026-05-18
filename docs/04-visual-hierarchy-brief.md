# 04 Visual Hierarchy Brief

> 角色：页面设计导演 / 当前状态审计者。位于 `03-production-constraints.md` 之后、`05-material-generation.md` 之前，负责把 02 的风格理解和 03 的生产硬约束转译成页面级视觉主次、背景强度、组件层级和色彩角色规范。

工作流位置：

```text
00 Index
-> 01 Workflow Dispatch
-> 02 Style Knowledge
-> 03 Production Constraints
-> 04 Visual Hierarchy Brief
-> 05 Material Generation
```

本文件不生成素材，不写代码，不改写 01 / 02 / 03 / 05 的职责。04 只在 `visual_hierarchy_scope: required` 时运行，并输出 `visual_hierarchy_brief`；必要时先输出 `current_visual_audit`，供 05 按页面层级执行生成和 QA。

## 一、职责定位

04 解决页面级主次和风格强度问题，重点避免：

- 背景图抢内容。
- 所有矩形都被套成同一种面板。
- 颜色没有主次。
- 按钮、标签、内容容器层级混乱。
- 已有风格迭代时不知道哪些要保留、哪些要削弱。

04 支持两类任务：

- `greenfield_style_application`：从 0 开始应用某个游戏风格。
- `iterative_style_refinement`：基于已有风格结果做迭代优化。

## 二、输入契约

04 必须先取得以下输入。缺失关键输入时，不应直接输出可执行的页面级规范。`target_components.role` 可以由 01 显式提供，也可以由 04 根据 `component_type`、`page_goal`、`panel_candidate_inventory` 和页面上下文推断。

```yaml
visual_hierarchy_inputs:
  workflow_request_ref: docs/01-workflow-dispatch.md output
  style_brief_ref: docs/02-style-knowledge.md#entry-id
  style_brief: provided_by_02_or_reused
  production_constraints_ref: docs/03-production-constraints.md
  target_page_or_screen:
  primary_focus_context:
    required_when: target_components_missing_or_incomplete
    description: page task, expected reading/action focus, protected content areas, affected state or QA target
  target_components:
    - component_name:
      component_type:
      role: provided_by_01_or_inferred_by_04
      width:
      height:
      state_requirements:
  current_visual_state:
    required_when: iterative_style_refinement
    inputs:
      - screenshot
      - existing implementation
      - existing generated assets
      - user feedback
```

输入解释：

- `workflow_request_ref` 用于确认用户意图、路由、目标资产和组件清单来源。
- `style_brief_ref` 和 `style_brief` 用于继承 02 已确认的游戏风格身份，不在 04 中新增具体游戏风格。
- `production_constraints_ref` 用于继承 03 的 Maker 能力、资源预算、图片导出和 fallback 边界。
- `target_page_or_screen` 用于判断页面目标、用户注意力路径和背景服务对象。
- `primary_focus_context` 用于在 `background_generation`、`state_guidance` 或 `qa_gate` 等组件清单不完整的场景下描述主要阅读 / 操作焦点、受保护内容区域、受影响状态或 QA 对象。
- `target_components.role` 建议由 01 提供；如果 01 当前未提供，04 可以从 `component_type`、`page_goal`、`panel_candidate_inventory`、内容承载方式和交互语义推断。仅有外观矩形、圆角或背景色不足以判断是否能套用完整面板；无法可靠推断时必须标为 `unknown`，并保守禁止完整 `programmatic_panel`。
- `current_visual_state` 只在迭代优化时必需，用于审计当前结果中应保留、削弱、移除或调整的视觉方向。

### 目标组件不完整时的处理

当 `target_components` 缺失或不完整时，04 是否阻断取决于 01 的路由和可用上下文：

```yaml
incomplete_target_components_policy:
  background_generation:
    allowed_with:
      - target_page_or_screen
      - primary_focus_context
    output_scope:
      - background
      - color_roles
      - readability_policy
      - forbidden_visual_outcomes
    component_treatment: empty_or_page_level_only
  state_guidance:
    allowed_with:
      - target_page_or_screen
      - primary_focus_context
      - affected_state_or_component_type
    output_scope:
      - state_color_roles
      - readability_policy
      - control_vs_container_distinction
      - forbidden_visual_outcomes
    component_treatment: affected_types_only
  qa_gate:
    allowed_with:
      - current_visual_state
      - target_page_or_screen_or_primary_focus_context
    output_scope:
      - current_visual_audit
      - visual_hierarchy_risks
      - keep_reduce_remove_adjust
    generation_allowed: false
  full_ui_skin:
    requires_complete_target_components: true
  panel_programmatic_draw:
    requires_complete_target_components: true
  panel_demo:
    requires_complete_target_components: true
```

如果是 `background_generation`、`state_guidance` 或 `qa_gate`，且 `target_page_or_screen + primary_focus_context` 足以判断内容焦点和受保护区域，04 可以输出受限 `visual_hierarchy_brief`。该 brief 不得为未知组件生成完整 `component_treatment`，也不得允许未知矩形使用完整 `programmatic_panel`。

如果是 `full_ui_skin`、`panel_programmatic_draw` 或 `panel_demo`，缺少完整 `target_components` 时必须阻断，并要求 01 补充组件上下文、尺寸、角色或 `panel_candidate_inventory`。

## 三、运行模式

```yaml
visual_hierarchy_mode:
  - greenfield_style_application
  - iterative_style_refinement
```

### greenfield_style_application

适用于用户从 0 开始说：

- “换成 XX 游戏风格”
- “生成 XX 风格页面”
- “整体套一版 XX 风格”
- “按 XX 游戏做这个工具 UI”

该模式不需要审计已有视觉结果，但必须基于页面目标、组件角色和 03 的生产约束先分配视觉层级，再允许 05 生成背景 prompt、`panel_render_recipe` 和状态视觉规则。

### iterative_style_refinement

适用于用户基于已有结果说：

- 背景太抢眼。
- 面板太重。
- 矩形全被填充了。
- 层级不清楚。
- 按钮不突出。
- 文字不好读。
- 保留当前方向但弱一点。
- 当前风格不像目标游戏。
- 只调整背景 / 面板 / 按钮 / 色彩层级。

该模式必须先做 `current_visual_audit`，再输出新的 `visual_hierarchy_brief`。不能在未审计当前结果的情况下全部推翻或重新生成。

## 四、当前状态审计

在 `iterative_style_refinement` 下，04 必须先输出 `current_visual_audit`。

```yaml
current_visual_audit:
  audit_id:
  existing_style_summary:
  likely_current_style_sources:
    - background
    - panel
    - button
    - tag
    - typography
    - decoration
  background_strength:
    value: low | medium | high
    evidence:
  panel_density:
    value: low | medium | high
    evidence:
  component_hierarchy_clarity:
    value: clear | mixed | unclear
    evidence:
  color_role_conflicts:
    - description:
      evidence:
  readability_risks:
    - area:
      risk:
      evidence:
  over_styled_components:
    - component_or_area:
      reason:
  under_styled_focus_components:
    - component_or_area:
      reason:
  keep:
    - element:
      reason:
  reduce:
    - element:
      reason:
  remove:
    - element:
      reason:
  adjust:
    - element:
      direction:
```

审计规则：

- `audit_id` 是本次审计的稳定标识，用于 05 或后续 QA 回指具体审计结果。
- `existing_style_summary` 只总结当前可见风格，不重新定义目标游戏风格。
- `likely_current_style_sources` 用于判断风格主要来自背景、面板、按钮、标签、字体还是装饰。
- `background_strength` 看中心细节、中心对比、饱和度、视觉中心和边缘装饰是否压过内容。
- `panel_density` 看完整面板皮肤是否过多、边框 / glow / 阴影是否过厚、内部分组是否被当主容器处理。
- `component_hierarchy_clarity` 看主内容、次级容器、按钮、标签、状态文字是否能一眼区分。
- `color_role_conflicts` 记录主色、装饰色、容器色、状态色、文字色之间的冲突。
- `readability_risks` 必须定位到具体区域，例如正文、表单、列表、按钮文案、状态提示或浮层。
- `over_styled_components` 记录被错误套完整面板、装饰过重或抢焦点的组件。
- `under_styled_focus_components` 记录本应成为行动焦点却不突出的按钮、关键状态或主卡片。
- `keep / reduce / remove / adjust` 必须明确写出，作为迭代模式进入 05 的保护栏。

## 五、页面视觉优先级输出

当 `visual_hierarchy_scope: required` 时，04 必须运行并输出 `visual_hierarchy_brief`；当 `visual_hierarchy_scope: not_required` 时，04 不运行、不输出 brief，也不输出 pass-through。

```yaml
visual_hierarchy_brief:
  brief_id:
  style_brief_ref:
  production_constraints_ref:
  mode:
  page_goal:
  primary_focus:
  secondary_focus:
  background:
    role:
    strength:
    center_detail_level:
    center_contrast_level:
    edge_decoration_level:
    should_recede_behind_content: true
  component_treatment:
    - component_type:
      hierarchy_level:
      treatment:
      use_programmatic_panel: true | false | limited
      use_accent: true | false
      color_role:
      reason:
  color_roles:
    background_palette:
    primary_surface:
    secondary_surface:
    primary_action:
    secondary_action:
    decorative_accent:
    text_primary:
    text_secondary:
    text_on_surface:
    text_on_primary_action:
    text_on_secondary_action:
    text_disabled:
    state_colors:
  text_on_surface_policy:
    page_background:
      allowed_text_roles:
        - text_primary
        - text_secondary
      forbidden_text_roles:
        - decorative_accent
        - surface_highlight
    primary_surface:
      allowed_text_roles:
        - text_primary
        - text_secondary
        - text_on_surface
    secondary_surface:
      allowed_text_roles:
        - text_primary
        - text_secondary
        - text_on_surface
    primary_action:
      required_label_role: text_on_primary_action
      contrast_priority: highest
    secondary_action:
      required_label_role: text_on_secondary_action
      contrast_priority: high
    disabled_control:
      required_label_role: text_disabled
      must_still_signal_disabled_state: true
  readability_policy:
    min_body_text_contrast:
    min_large_text_contrast:
    min_interactive_text_contrast:
    min_disabled_text_contrast:
    content_scrim_allowed:
    fallback_if_contrast_fails:
  style_strength:
    background:
    primary_panel:
    secondary_panel:
    controls:
    decorations:
  forbidden_visual_outcomes:
    - background_competes_with_content
    - every_rectangle_gets_panel_skin
    - action_buttons_look_same_as_containers
    - decorative_color_used_as_main_text_surface
    - interactive_text_too_low_contrast
    - button_label_uses_decorative_accent_as_text
    - enabled_button_looks_disabled
    - control_outline_more_visible_than_label
    - existing_good_style_removed_without_reason
    - violates_03_production_constraints
```

字段规则：

- `brief_id` 是本次页面级规范的稳定标识，供 01 / 05 / QA 通过 `visual_hierarchy_brief_ref` 引用。
- `page_goal` 描述页面要帮助用户完成的任务，不写游戏世界观介绍。
- `primary_focus` 是页面最重要的阅读或操作焦点，通常是主内容、核心表单、主卡片、主按钮或当前决策区域。
- `secondary_focus` 是辅助信息、筛选、补充说明或次级操作。
- `background.role` 必须是服务 UI 的环境层，不得变成插画主视觉、登录海报或角色展示。
- `background.strength` 建议使用 `low | medium_low | medium`，只有非内容型展示页才可考虑更强；工具页默认不应高于 `medium_low`。
- `center_detail_level` 和 `center_contrast_level` 必须低于边缘装饰，用于保护正文、表单、列表和按钮。
- `edge_decoration_level` 可以承载更多游戏识别语义，但不能压过按钮、导航、输入框和状态提示。
- `component_treatment` 必须逐类说明组件层级和处理方式，05 不能只看组件是否是矩形。
- `use_programmatic_panel: true` 表示允许使用完整程序化面板主体；`limited` 表示只允许轻量程序化表面、细描边、弱 tint 或状态 overlay；`false` 表示不得套用程序化面板主体。
- `use_accent` 只表示该组件是否承接 02 / 03 约束下的固定角部局部装饰，不表示可新增通用装饰图；当 03 / 05 已因 `target_assets.values` 包含 `optional_panel_accent` 而要求生成面板装饰时，`use_accent: false` 只能表示“不贴到该组件”，不能取消资产包里的必交付装饰。
- `color_roles` 必须区分背景、容器、行动、装饰、文字和状态，避免所有元素抢同一个主色。
- `text_on_surface_policy` 必须说明文字落在页面背景、内容面、行动按钮和禁用控件上时使用哪个文字角色，避免把装饰高光色当作功能文案色。
- `readability_policy` 必须给出对比底线和失败兜底，通常优先降低背景强度或增加内容 scrim，再调整文字色。
- `style_strength` 用于把风格强度分配到背景、主面板、次面板、控件和装饰，不允许每层都满强度。
- `forbidden_visual_outcomes` 记录页面级禁忌结果，05 和 QA 必须按这些结果检查，而不是只检查单个资产是否合规。

## 六、组件层级规则

04 必须根据组件角色决定风格处理强度，而不是根据外形决定是否套面板。

### 可以使用完整 programmatic_panel 的组件

以下组件可以使用完整程序化面板，但仍需遵守 03 的尺寸、图层和 QA 约束：

- 主内容容器。
- 弹窗主体。
- 重要信息卡。
- 设置面板。
- 页面主卡片。

推荐 treatment：

```yaml
full_programmatic_panel_allowed:
  hierarchy_level: primary | strong_secondary
  treatment:
    - complete_surface
    - readable_center
    - restrained_border
    - optional_single_corner_accent_when_safe
  use_programmatic_panel: true
  use_accent: true
```

### 只能轻量处理的组件

以下组件只允许轻量表面、透明度、细描边、间距、scrim、弱 tint 或状态 overlay：

- 次级容器。
- 内部分组。
- list row。
- form area。

推荐 treatment：

```yaml
light_treatment_only:
  hierarchy_level: secondary | tertiary
  treatment:
    - light_surface_tint
    - simple_border_or_divider
    - no_heavy_corner_accent
    - preserve_content_scanability
  use_programmatic_panel: false | limited
  use_accent: false
```

当这类组件必须与主容器形成层级时，应优先通过透明度、边框轻重、阴影层级、padding、间距和文字权重区分，而不是套同一张完整面板。

### 不能套完整面板的组件

以下组件不能套完整程序化面板，也不能被当作通用 panel skin 对象：

- button
- tag
- pill
- small label
- divider
- status text
- text row
- icon button
- toolbar
- checkbox
- switch
- input
- badge
- avatar

推荐 treatment：

```yaml
full_panel_forbidden:
  hierarchy_level: control | label | state | inline_content
  treatment:
    - control_specific_style
    - state_overlay_or_primitive_only
    - no_panel_skin
    - no_corner_accent
  use_programmatic_panel: false
  use_accent: false
```

按钮必须像按钮，标签必须像标签，输入框必须像输入框。它们可以继承色彩、描边、圆角、hover / pressed / selected 等状态语言，但不能变成和内容容器一样的面板。

### 交互控件文字规则

按钮、标签、pill、input 和状态控件的文案属于功能文本层，不属于装饰层。04 必须为这些控件指定清晰的文字落色关系，05 不得用装饰色、浅高光色、背景氛围色或边框色替代可点击控件的主要文案色。

规则：

- 可点击控件的文字对比优先级高于风格柔和感；风格可以低饱和，但交互文案不能低可读。
- enabled button / tag / pill 的文案必须比控件描边、装饰线和背景氛围更清楚，不能出现“轮廓比文字更显眼”的结果。
- `primary_action` 必须使用 `text_on_primary_action`，并保留页面中最高或接近最高的交互文字对比。
- `secondary_action` 必须使用 `text_on_secondary_action`，可以弱于主按钮，但必须明显高于 disabled 文案。
- disabled 控件必须使用 `text_disabled`，并通过透明度、饱和度、描边或状态说明表达不可用；disabled 仍需可辨认，不得完全消失。
- `decorative_accent`、`surface_highlight`、发光色和浅边框色不得作为 enabled 控件的主要文案色。
- 如果按钮或标签底色很浅，应优先加深文案色或调整底色，而不是靠文字描边、glow、阴影或新增装饰解决。
- 如果风格要求白色、浅金、浅粉或其他浅色文字，只能用于足够深的底色；不能直接落在浅色按钮面、浅色页面背景或浅色半透明表面上。

## 七、色彩角色规则

04 必须为页面分配颜色角色，防止背景、面板和操作控件争夺同一主色。

```yaml
color_role_policy:
  background_palette:
    purpose: atmosphere_and_depth
    strength: weakest_readable_layer
  primary_surface:
    purpose: main_content_legibility
    strength: stable_and_quiet
  secondary_surface:
    purpose: grouping_without_competing
    strength: lighter_than_primary_surface
  primary_action:
    purpose: main_user_action
    strength: strongest_interactive_color
  secondary_action:
    purpose: supporting_action
    strength: below_primary_action
  decorative_accent:
    purpose: style_identity_and_small_highlights
    strength: limited_area_only
  text_primary:
    purpose: main_reading
    strength: highest_legibility
  text_secondary:
    purpose: metadata_or_supporting_text
    strength: readable_but_lower_emphasis
  text_on_surface:
    purpose: readable_text_on_primary_or_secondary_surface
    strength: functional_text_not_decoration
  text_on_primary_action:
    purpose: label_text_on_primary_action
    strength: highest_interactive_legibility
  text_on_secondary_action:
    purpose: label_text_on_secondary_action
    strength: high_interactive_legibility
  text_disabled:
    purpose: disabled_or_unavailable_control_label
    strength: readable_but_clearly_disabled
  state_colors:
    purpose: selected_disabled_warning_error_success_new_reward
    strength: semantic_not_decorative
```

规则：

- 最强颜色优先留给 `primary_action` 或关键状态，不给背景或普通容器。
- `decorative_accent` 只能小面积使用，不能成为大面积文字底或普通面板底。
- `primary_surface` 必须服务内容阅读，不能因为风格化而牺牲正文和表单。
- `secondary_surface` 应比主内容容器更轻，避免所有层级都像主卡片。
- `text_primary`、`text_secondary`、`text_on_surface`、`text_on_primary_action`、`text_on_secondary_action` 和 `text_disabled` 必须是功能文字角色，不得被 `decorative_accent`、浅高光色或背景氛围色替代。
- `text_on_primary_action` 和 `text_on_secondary_action` 必须根据按钮底色反向选择深浅；浅底按钮优先用深色文字，深底按钮才允许用浅色文字。
- enabled 控件文案的可读性必须高于 disabled 文案；如果 enabled 控件看起来像 disabled，判定为色彩角色分配失败。
- 控件边框、外描边、装饰线或 glow 不能比控件文案更抢眼；否则用户会先识别轮廓而不是操作含义。
- warning / error 不能被装饰色或 selected 色混淆。
- 如果背景颜色很有识别度，面板和按钮必须降低同色竞争，让页面留出焦点。

## 八、背景强度规则

背景是页面气氛层，不是主内容层。04 必须把背景强度写入 `visual_hierarchy_brief.background`。

默认建议：

```yaml
background_strength_defaults:
  tool_page:
    strength: low | medium_low
    center_detail_level: low
    center_contrast_level: low
    edge_decoration_level: medium_low | medium
  modal_or_form_heavy_page:
    strength: low
    center_detail_level: very_low | low
    center_contrast_level: low
    edge_decoration_level: low | medium_low
  showcase_or_empty_state:
    strength: medium_low | medium
    center_detail_level: low | medium_low
    center_contrast_level: low | medium_low
    edge_decoration_level: medium
```

背景过强时，优先按以下顺序处理：

1. 降低中心细节。
2. 降低中心对比。
3. 降低饱和度。
4. 降低中心装饰和强光。
5. 将识别语义移到边缘、角落或远景。
6. 必要时允许内容 scrim，但 scrim 不能把页面变成新的重面板。

## 九、迭代时的保留原则

如果当前结果已有可用风格，不要全部推翻。04 必须明确：

- `keep`：保留哪些方向。
- `reduce`：削弱哪些强度。
- `remove`：移除哪些错误套用。
- `adjust`：调整哪些层级或色彩角色。

常见迭代策略：

- 背景抢眼时，优先降低背景中心细节、对比度、饱和度和装饰强度，不先重做面板或替换整体方向。
- 面板全都填充时，优先减少 secondary / tertiary 组件的 `programmatic_panel` 使用，只保留主容器使用完整面板。
- 按钮不突出时，优先把最强色彩留给 `primary_action`，不要让背景或普通面板抢走主色。
- 文字不好读时，优先降低底层干扰、增加内容区稳定面或 scrim，再考虑调整文字色和字重。
- 按钮、标签或 pill 文案不好读时，优先调整 `text_on_primary_action` / `text_on_secondary_action` 与控件底色的对比；不要优先加文字描边、glow、阴影或装饰。
- 风格方向可用但过重时，保留材质、形状和小面积识别语义，削弱 glow、厚边框、强纹理和高饱和大色块。
- 当前风格不像目标游戏时，先判断是 02 风格理解问题、05 prompt 消费问题、背景 identity anchor 位置问题，还是 04 页面强度分配问题，再决定反馈对象。

迭代模式禁止：

- 未写 `keep` 就全量重做。
- 未写证据就删除已有可用风格方向。
- 只因为组件是矩形就继续套同一套面板。
- 为了增加游戏感而牺牲主要内容可读性。

## 十、与 01 的衔接

01 应在 `workflow_request` 中加入以下字段，用于决定是否进入 04；当 `visual_hierarchy_scope: required` 时，这些字段供 04 判断是否需要审计和输出何种页面级 brief：

```yaml
workflow_request:
  visual_hierarchy_scope: required | not_required
  visual_hierarchy_brief_ref: docs/04-visual-hierarchy-brief.md#brief_id | null
  visual_hierarchy_skip_reason:
  visual_hierarchy_mode:
    required_when: visual_hierarchy_scope == required
    value: greenfield_style_application | iterative_style_refinement | null
  current_visual_audit_required: true | false
```

01 接入规则：

- 用户从 0 开始要求套用游戏风格时，设置 `visual_hierarchy_mode: greenfield_style_application`。
- 用户基于已有结果提出“太抢、太重、不清楚、不突出、不好读、弱一点、只调整某层”等反馈时，设置 `visual_hierarchy_mode: iterative_style_refinement`。
- `iterative_style_refinement` 下，`current_visual_audit_required` 必须为 `true`，并附上截图、已有实现、已生成资产或用户反馈中可取得的输入。
- 当请求涉及页面级主次、背景强度、组件层级或当前视觉审计时，01 必须设置 `visual_hierarchy_scope: required`，并进入 04。
- 当请求不涉及页面级主次、背景强度、组件层级或当前视觉审计时，01 必须设置 `visual_hierarchy_scope: not_required`，不进入 04，把 `visual_hierarchy_mode.value` 和 `visual_hierarchy_brief_ref` 设为 `null`，并填写 `visual_hierarchy_skip_reason`。
- 01 仍负责意图路由、style_brief resolution 和 target_components 盘点，不在 01 中写页面视觉规则。
- 01 可以把 `visual_hierarchy_brief_ref` 传给 05，但不应替代 04 输出 brief。

### 跳过 04 的 pass-through

以下场景要求 01 跳过 04，并向 05 传递 pass-through：

```yaml
visual_hierarchy_scope: not_required
visual_hierarchy_brief_ref: null
visual_hierarchy_skip_reason:
```

允许跳过 04 的场景：

- `style_brief_resolution_only`
- `production_constraints_only`
- `qa_only_without_page_hierarchy_question`
- `non_visual_or_metadata_only_request`

当 05 收到 `visual_hierarchy_scope: not_required` 时，05 不应阻塞等待 `brief_id`，也不应自行推断页面级视觉层级；只允许按 01 的路由、02 的 `style_brief` 和 03 的生产约束继续处理其职责范围内的任务。

## 十一、与 05 的衔接

当 `visual_hierarchy_scope: required` 时，05 必须消费 `visual_hierarchy_brief` 后才能生成页面相关物料规则。当 `visual_hierarchy_scope: not_required` 时，05 不应要求 04 输出或等待 `visual_hierarchy_brief`。

05 消费要求：

- 不能只根据矩形外观决定是否套 panel。
- 必须按 `component_treatment` 决定每个组件的处理方式。
- 背景 prompt / 背景参数必须遵守 `background.strength`、`center_detail_level`、`center_contrast_level` 和 `edge_decoration_level`。
- `panel_render_recipe` 必须遵守 `hierarchy_level` 和 `color_roles`，主容器、次级容器、按钮、标签、输入框不能同强度处理。
- 按钮、标签、pill、input 和状态控件必须遵守 `text_on_surface_policy` 与 `readability_policy`，不能让 enabled 控件文案低对比、像 disabled，或让边框比文案更清楚。
- QA 必须检查 `every_rectangle_gets_panel_skin` 是否发生。
- QA 必须检查 `background_competes_with_content` 是否发生。
- 迭代模式下，05 必须保护 `current_visual_audit.keep`，优先执行 `reduce / remove / adjust`，不能无理由清空当前可用风格方向。
- 当 `visual_hierarchy_brief` 与 03 生产硬约束冲突时，05 必须以 03 为硬约束，并把冲突反馈给 04 或 03，不得自行扩大资源预算或突破 Maker 边界。

建议 05 在物料元数据中记录：

```yaml
visual_hierarchy_trace:
  visual_hierarchy_brief_ref: docs/04-visual-hierarchy-brief.md#brief_id
  mode:
  primary_focus_used:
  background_strength_used:
  component_treatment_used:
    - component_type:
      hierarchy_level:
      use_programmatic_panel:
      color_role:
  color_roles_used:
  text_on_surface_policy_used:
  forbidden_visual_outcome_checks:
    background_competes_with_content:
    every_rectangle_gets_panel_skin:
    action_buttons_look_same_as_containers:
    interactive_text_too_low_contrast:
    button_label_uses_decorative_accent_as_text:
    enabled_button_looks_disabled:
    control_outline_more_visible_than_label:
```

## 十二、职责边界

04 负责：

- 页面视觉主次。
- 当前结果审计。
- 背景强度建议。
- 组件风格强度分级。
- 色彩角色分配。
- 迭代时 `keep / reduce / remove / adjust`。

04 不负责：

- 新增或改写具体游戏 `style_brief`，那是 02。
- 定义资源预算、资产白名单、fallback 条件和 Maker 硬约束，那是 03。
- 写 prompt 模板、`panel_render_recipe` 具体实现、metadata 执行和 QA gate，那是 05。
- 最终素材验收由 05 的 QA gate 或后续独立 QA 文档负责。

## 十三、失败归因补充

当页面级问题出现时，优先按以下方向归因：

```yaml
visual_hierarchy_failure_attribution:
  background_competes_with_content:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - 04_should_reduce_background_strength_if_brief_allowed_too_much
      - 05_should_follow_background_strength_if_brief_was_clear
  every_rectangle_gets_panel_skin:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - 04_should_classify_component_treatment_more_strictly
      - 05_should_consume_component_treatment_not_shape_only
  action_buttons_look_same_as_containers:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - reserve_strongest_color_for_primary_action
      - forbid_full_panel_skin_on_buttons
  decorative_color_used_as_main_text_surface:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - reassign_color_roles
      - restore_text_contrast
  interactive_text_too_low_contrast:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - define_or_correct_text_on_surface_policy
      - increase_interactive_text_contrast_before_adding_decoration
  button_label_uses_decorative_accent_as_text:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - move_button_label_to_text_on_primary_or_secondary_action
      - keep_decorative_accent_as_border_or_small_highlight_only
  enabled_button_looks_disabled:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - separate_enabled_and_disabled_text_roles
      - increase_enabled_control_label_contrast
  control_outline_more_visible_than_label:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - make_label_more_legible_than_outline
      - reduce_outline_or_raise_text_role
  existing_good_style_removed_without_reason:
    likely_responsible: docs/04-visual-hierarchy-brief.md | docs/05-material-generation.md
    fix_direction:
      - require_current_visual_audit_keep_reduce_remove_adjust
      - enforce_keep_items_in_iteration
```

如果问题本质是“目标游戏风格理解错误”，反馈给 02。
如果问题本质是“资源预算、图片形态、fallback 或 Maker 能力越界”，反馈给 03。
如果问题本质是“brief 已清楚但 prompt、recipe、metadata 或 QA 没执行”，反馈给 05。

## 十四、输出示例骨架

### Greenfield 输出骨架

```yaml
visual_hierarchy_brief:
  brief_id:
  style_brief_ref: docs/02-style-knowledge.md#entry-id
  production_constraints_ref: docs/03-production-constraints.md
  mode: greenfield_style_application
  page_goal:
  primary_focus:
  secondary_focus:
  background:
    role: low_density_ui_support_background
    strength: low | medium_low
    center_detail_level: low
    center_contrast_level: low
    edge_decoration_level: medium_low
    should_recede_behind_content: true
  component_treatment:
    - component_type: main_content_container
      hierarchy_level: primary
      treatment: complete_programmatic_panel_with_clean_center
      use_programmatic_panel: true
      use_accent: true
      color_role: primary_surface
      reason:
    - component_type: button
      hierarchy_level: control
      treatment: action_control_style_not_panel_skin
      use_programmatic_panel: false
      use_accent: false
      color_role: primary_action | secondary_action
      reason:
  color_roles:
    background_palette:
    primary_surface:
    secondary_surface:
    primary_action:
    secondary_action:
    decorative_accent:
    text_primary:
    text_secondary:
    text_on_surface:
    text_on_primary_action:
    text_on_secondary_action:
    text_disabled:
    state_colors:
  text_on_surface_policy:
    primary_action:
      required_label_role: text_on_primary_action
      contrast_priority: highest
    secondary_action:
      required_label_role: text_on_secondary_action
      contrast_priority: high
    disabled_control:
      required_label_role: text_disabled
  readability_policy:
    min_body_text_contrast: "WCAG AA target or project equivalent"
    min_large_text_contrast: "WCAG AA target or project equivalent"
    min_interactive_text_contrast: "WCAG AA target or project equivalent"
    min_disabled_text_contrast: "readable but visibly disabled"
    content_scrim_allowed: true
    fallback_if_contrast_fails:
  style_strength:
    background:
    primary_panel:
    secondary_panel:
    controls:
    decorations:
  forbidden_visual_outcomes:
    - background_competes_with_content
    - every_rectangle_gets_panel_skin
    - action_buttons_look_same_as_containers
    - decorative_color_used_as_main_text_surface
    - interactive_text_too_low_contrast
    - button_label_uses_decorative_accent_as_text
    - enabled_button_looks_disabled
    - control_outline_more_visible_than_label
    - existing_good_style_removed_without_reason
    - violates_03_production_constraints
```

### Iterative 输出骨架

```yaml
current_visual_audit:
  audit_id:
  existing_style_summary:
  likely_current_style_sources:
    - background
    - panel
    - button
    - tag
    - typography
    - decoration
  background_strength:
    value: low | medium | high
    evidence:
  panel_density:
    value: low | medium | high
    evidence:
  component_hierarchy_clarity:
    value: clear | mixed | unclear
    evidence:
  color_role_conflicts:
    - description:
      evidence:
  readability_risks:
    - area:
      risk:
      evidence:
  over_styled_components:
    - component_or_area:
      reason:
  under_styled_focus_components:
    - component_or_area:
      reason:
  keep:
    - element:
      reason:
  reduce:
    - element:
      reason:
  remove:
    - element:
      reason:
  adjust:
    - element:
      direction:

visual_hierarchy_brief:
  brief_id:
  style_brief_ref: docs/02-style-knowledge.md#entry-id
  production_constraints_ref: docs/03-production-constraints.md
  mode: iterative_style_refinement
  page_goal:
  primary_focus:
  secondary_focus:
  background:
    role:
    strength:
    center_detail_level:
    center_contrast_level:
    edge_decoration_level:
    should_recede_behind_content: true
  component_treatment: []
  color_roles:
    background_palette:
    primary_surface:
    secondary_surface:
    primary_action:
    secondary_action:
    decorative_accent:
    text_primary:
    text_secondary:
    text_on_surface:
    text_on_primary_action:
    text_on_secondary_action:
    text_disabled:
    state_colors:
  text_on_surface_policy:
    primary_action:
      required_label_role: text_on_primary_action
      contrast_priority: highest
    secondary_action:
      required_label_role: text_on_secondary_action
      contrast_priority: high
    disabled_control:
      required_label_role: text_disabled
  readability_policy:
    min_body_text_contrast:
    min_large_text_contrast:
    min_interactive_text_contrast:
    min_disabled_text_contrast:
    content_scrim_allowed:
    fallback_if_contrast_fails:
  style_strength:
    background:
    primary_panel:
    secondary_panel:
    controls:
    decorations:
  forbidden_visual_outcomes:
    - background_competes_with_content
    - every_rectangle_gets_panel_skin
    - action_buttons_look_same_as_containers
    - decorative_color_used_as_main_text_surface
    - interactive_text_too_low_contrast
    - button_label_uses_decorative_accent_as_text
    - enabled_button_looks_disabled
    - control_outline_more_visible_than_label
    - existing_good_style_removed_without_reason
    - violates_03_production_constraints
```
