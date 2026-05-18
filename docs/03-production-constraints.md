# 03 Production Constraints

> 角色：03 Production Constraints / 生产硬约束。定义 Maker 生产链路的全局硬约束。本文件不绑定任何具体游戏风格，也不保存风格特征。

## 输入与输出

输入：

- 已确认的工具页面、组件清单和替换范围。
- `02-style-knowledge.md` 输出的 `style_brief` 引用。
- 目标 Maker / UrhoX 工程环境。

输出：

- 可做 / 不可做边界。
- 默认资源预算与图片 fallback 禁止条件。
- 尺寸、图层、导出和资产类型硬约束。
- QA 失败归因表和回修顺序。

## Maker 可实现边界

### 可以做

- 在不改业务逻辑的前提下替换视觉皮肤。
- 使用 `backgroundColor`、`borderWidth`、`borderColor`、`borderRadius`、`opacity`、`shadow`、`tint` 等 UI primitive。
- 使用代码程序化绘制面板主体。
- 使用透明 PNG 作为背景和不参与拉伸的固定角部面板装饰。
- 使用程序化绘制替代 AI 生成的整张 9-slice / panel base 图片。
- 使用独立 overlay 表达 selected、disabled、locked、warning 等状态。
- 使用简单 tween：透明度、位移、缩放、轻微呼吸、短时高亮。
- 复用现有字体、图标库和组件结构。

### 不应该做

- 改变用户内容、页面结构、组件数量、组件位置、交互流程或业务语义。
- 把文字、按钮、勾选、头像、状态、品类条直接烘焙到基础图片里。
- 把 AI 生成的整张面板图作为默认通用底板。
- 用一张普通拉伸图冒充 9-slice。
- 为每个尺寸生成一张面板图。
- 依赖图片拉伸实现复杂边框、发光、玻璃感、角装饰、侧边凹槽或边中结构。
- 在背景图中生成清晰文字、假 UI、角色主视觉、强视觉中心或高密度插画。
- 依赖复杂 shader、流体、布料、骨骼动画、大量粒子、视频背景或实时 3D。
- 使用未授权 logo、角色、IP 专属符号或商业素材复刻。

## 默认面板实现方案

默认面板方案为 `programmatic_draw`，不允许 AI 生成整张 `panel_base_9slice.png`。

```yaml
default_panel_strategy: programmatic_draw
code_responsibilities:
  - panel_fill
  - border_radius
  - main_border
  - inner_stroke
  - outer_shadow
  - inner_glow
  - subtle_gradient
  - scalable_straight_segments
  - fixed_pixel_polyline_nodes
  - fixed_pixel_notches
  - fixed_pixel_ticks
ai_png_responsibilities:
  - non_scaling_corner_accents_only
```

代码负责面板主体，包含底色、圆角、主边框、内描边、外阴影、内发光、轻微渐变、可缩放直线段，以及固定像素折线节点、凹槽和刻度。

AI / PNG 只允许负责不参与拉伸的角部贴片：

- 四角小装饰。
- 右下角或指定角落折角。
- 角部抽象材质片、线条或轮廓装饰。

`panel_base_*.png` 不属于允许的 image asset。默认不要求，也不允许 AI 生成整张 9-slice 面板底图。

## 默认资源预算

默认预算用于“已有工具 UI 的风格替换”，不是完整游戏界面重做。

## 图片资产与非图片输出边界

`full_ui_skin` 可以有多个交付项，但不等于可以生成多个图片资产。03 必须区分 image asset 和 non-image output。

默认只允许生成以下 image assets：

```yaml
image_assets_allowed:
  background_tool.png:
    max_count: 1
    purpose: 页面氛围与世界观语义背景
  "panel_corner_accent.png | panel_accent.png":
    max_count: 1
    optional: true
    purpose: 固定位置面板角部抽象装饰，不参与拉伸
```

`panel_corner_accent.png` 和 `panel_accent.png` 二选一，默认最多生成 1 张。除上述白名单外，不允许生成其他 image assets。

以下交付项是 non-image outputs，不计入图片资产数量，也不得导出为 PNG：

```yaml
non_image_outputs:
  programmatic_panel:
    output: panel_render_recipe only
    image_asset: false
  state_guidance:
    output: state_visual_rule only
    image_asset: false
```

逻辑产物名必须先映射到实际产物类型，不能把未知逻辑名直接当作新增图片资产：

- `optional_panel_accent`：可选面板角饰逻辑产物。
- `allowed_image_assets`：允许落地的真实图片文件名。
- `allowed_when`：允许出现的路由。

```yaml
logical_output_mapping:
  programmatic_panel:
    type: non_image_output
    output: panel_render_recipe
    png_allowed: false
  state_guidance:
    type: non_image_output
    output: state_visual_rule
    png_allowed: false
  optional_panel_accent:
    type: optional_image_asset
    allowed_image_assets:
      - panel_corner_accent.png
      - panel_accent.png
    max_count: 1
    allowed_when:
      - panel_programmatic_draw
      - panel_demo
      - full_ui_skin
  background_tool.png:
    type: image_asset
    max_count: 1
    allowed_when:
      - background_generation
      - full_ui_skin
```

05 调用图片生成前必须先执行 asset mapping gate。05 只能从 `image_assets_after_mapping` 中生成图片，不能直接把 `workflow_request.target_assets` 当作图片资产清单：

```yaml
asset_mapping_gate:
  required_before_generate_image: true
  input:
    - workflow_request.route_to
    - workflow_request.target_assets.values
  output:
    image_assets_after_mapping:
      - string
    non_image_outputs_after_mapping:
      - string
    blocked_assets:
      - string
```

硬规则：

- `image_assets_after_mapping` 为空时，禁止调用 `generate_image`。
- `programmatic_panel` 只能进入 `non_image_outputs_after_mapping: panel_render_recipe`。
- `state_guidance` 只能进入 `non_image_outputs_after_mapping: state_visual_rule`。
- `background_tool.png` 只有 `background_generation` / `full_ui_skin` 可进入 `image_assets_after_mapping`。
- `optional_panel_accent` 只有 `panel_programmatic_draw` / `panel_demo` / `full_ui_skin` 可映射为 `panel_corner_accent.png` 或 `panel_accent.png`。

禁止生成以下 image assets：

```yaml
forbidden_image_assets:
  - divider_*.png
  - panel_base_*.png
  - "bg_landscape_*.png + bg_main_*.png dual_background"
  - state_*.png
  - button_*.png
  - icon_*.png
  - badge_*.png
  - generated_button_*.png
  - generated_icon_*.png
  - generated_badge_*.png
```

```yaml
asset_budget_default:
  image_assets:
    background_tool.png: max 1
    panel_corner_accent_or_panel_accent.png: max 1 optional
    all_other_png: 0
  non_image_outputs:
    programmatic_panel: panel_render_recipe only
    state_guidance: state_visual_rule only
```

`programmatic_panel` 只能输出 `panel_render_recipe`，不能输出 `panel_base_*.png`。
`state_guidance` 只能输出 `state_visual_rule`，不能输出 `state_*.png`。
`icon_new: 0` 表示当前不允许新增任何独立图标资源；风格语义只能由背景图、程序化面板、固定角部面板装饰、UI primitive 或已有图标库承载。

优先级：

1. 先使用 UI primitive。
2. 仅当 `route_to` 为 `background_generation` 或 `full_ui_skin` 时，允许 / 需要输出 1 张 `background_tool.png`；`panel_programmatic_draw`、`panel_demo`、`state_guidance`、`qa_gate`、`style_discovery_only` 不默认生成背景图。
3. 面板主体默认使用 `programmatic_draw`，输出 `panel_render_recipe`，不输出面板 PNG。
4. 面板装饰最多使用 1 张固定角部 PNG，且不参与拉伸。
5. 状态视觉只输出 `state_visual_rule`，不生成 `state_*.png`。
6. 不生成新按钮、图标、徽章或分割线 PNG；装饰语义优先由背景图、程序化面板、固定角部面板装饰、UI primitive 或已有图标库承载。

## 图片资产 fallback 条件

默认不允许通过 fallback 扩大 image asset 白名单。

如果程序化绘制能力或运行环境不适配当前面板需求，回修 `panel_render_recipe`、Maker 参数或实现能力边界，不申请 `panel_base_*.png`、`divider_*.png`、`state_*.png`、按钮 / 图标 / 徽章 PNG 作为替代。

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

禁止的整图 / 切片面板 PNG：

- 不允许生成 `panel_base_*.png`，包括 `panel_base_9slice.png`。
- 不允许生成 `divider_*.png` 作为分割线。
- 不允许生成 `state_*.png` 作为 selected、disabled、locked、warning 等状态资源。
- 不允许生成按钮、图标、徽章 PNG。
- 不允许同时生成 `bg_landscape_*.png` 和 `bg_main_*.png` 形成双背景。

面板角部装饰 PNG：

- 只允许作为固定位置贴片，不参与拉伸。
- 必须保持固定像素尺寸，不随面板宽高比例缩放。
- 只允许放在角部，不允许作为侧边挂件或边中装饰。
- 默认最多 1 个角落装饰。
- 如果存在默认位置，优先使用右下角。
- 其他角保持干净。
- 不允许文字、logo、角色、徽章、状态图标或具体 IP 物件。
- 不允许跨进可缩放区域后被拉伸。
- 不允许边中复杂装饰依赖图片拉伸。

## 尺寸边界

### 程序化面板多尺寸验证

所有程序化面板主体必须至少验证以下尺寸：

```yaml
programmatic_panel_required_viewports:
  - width: 800
    height: 300
  - width: 300
    height: 500
  - width: 660
    height: 440
  - width: 520
    height: 200
```

必须检查：

- 圆角不随尺寸比例变形。
- 边框宽度保持固定像素。
- 发光和阴影不作为整图被拉伸。
- 中心区域纯净。
- 直线段只延长，不拉伸角和装饰。
- 固定装饰保持像素尺寸。
- 没有使用整张 AI 面板图冒充通用底板。

### 禁止的图片 fallback 边界

```yaml
forbidden_image_fallbacks:
  panel_base_*.png: forbidden
  divider_*.png: forbidden
  state_*.png: forbidden
  generated_button_icon_badge_png: forbidden
  dual_background_png:
    bg_landscape_*.png_plus_bg_main_*.png: forbidden
```

- 程序化面板的尺寸适配由 `panel_render_recipe` 和 Maker 参数负责，不通过生成 `panel_base_*.png` 解决。
- 状态视觉由 `state_visual_rule` 和现有 UI primitive / overlay 负责，不通过生成 `state_*.png` 解决。
- 分割线、按钮、图标和徽章必须使用 UI primitive、已有图标库或现有组件资源，不新增 PNG。
- 背景只能使用允许的单张 `background_tool.png`，不拆成双背景图片。

## 图层约束

推荐图层顺序：

```yaml
1: background_image_or_page_color
2: programmatic_panel_base
3: optional_panel_accent_png
4: optional_tint_or_surface_overlay
5: content_image_or_icon
6: text
7: top_strip_or_category_layer
8: selected_overlay_or_border
9: state_marker_or_check
10: disabled_overlay_if_needed
```

状态层必须位于基础面板之上。禁用态 overlay 可以覆盖内容，但不能让关键文字不可读。

## QA 失败归因

QA 失败先归因，再回修。不要在没有归因时直接重生成物料。

视觉层级类问题必须先检查 `04` 的 `visual_hierarchy_brief`。如果 brief 允许了过强背景、错误主次或错误组件处理，反馈 `04-visual-hierarchy-brief.md`；如果 brief 已经清楚约束，但 `05` 的 prompt、recipe、metadata 或 QA 没执行，反馈 `05-material-generation.md`。资产预算、fallback、白名单、尺寸和多尺寸 QA 硬约束仍反馈本文件。

图片资产边界类问题的规则归属在 03；如果 03 已明确禁止 `panel_base_*.png` 等资产，但 05 仍生成或请求该资产，执行回修在 `05-material-generation.md`。

| 失败现象 | 优先归因层 | 回修文件 |
|---|---|---|
| 看起来不像目标游戏或关键词 | 风格理解错误 | `02-style-knowledge.md` |
| 背景质量合格，但像通用风格原型，不像目标游戏 | Style Specificity Contract 缺失 | `02-style-knowledge.md` |
| `style_brief` 有差异锚点，但生成 prompt 没有使用 | prompt 消费失败 | `05-material-generation.md` |
| 差异锚点出现了，但破坏 UI 安全区 | identity anchor placement 错误 | `05-material-generation.md` |
| 颜色、材质、形状语言和情绪冲突 | 风格理解错误 | `02-style-knowledge.md` |
| `visual_hierarchy_brief` 允许背景抢内容、中心过花、对比过强或把背景设为错误主焦点 | 视觉层级 brief 错误 | `04-visual-hierarchy-brief.md` |
| `visual_hierarchy_brief` 已限制背景强度、中心细节和中心对比，但背景仍抢内容或中心太花 | prompt / recipe / QA 执行失败 | `05-material-generation.md` |
| `visual_hierarchy_brief` 允许错误主次、组件层级混乱，或把所有矩形都当成完整面板候选 | 视觉层级 brief 错误 | `04-visual-hierarchy-brief.md` |
| `visual_hierarchy_brief` 已清楚约束组件处理、颜色角色和主次，但组件层级仍混乱或所有矩形都套面板 | visual hierarchy 执行失败 | `05-material-generation.md` |
| 面板有白底或截图底 | 图片导出错误 | `03-production-constraints.md` |
| 默认面板主体使用整张 AI 面板图冒充通用底板 | 面板实现方案错误（blocker） | `03-production-constraints.md` |
| 生成或请求 `panel_base_*.png`、`divider_*.png`、`state_*.png`、按钮 / 图标 / 徽章 PNG 或双背景 PNG | forbidden image asset 错误（blocker） | `03-production-constraints.md` |
| 03 已明确禁止 `panel_base_*.png`，但 05 仍生成或请求该资产 | forbidden image asset 执行未落实（blocker） | `05-material-generation.md` |
| 程序化面板没有验证 800x300、300x500、660x440、520x200 | 多尺寸 QA 缺失（blocker） | `03-production-constraints.md` |
| 圆角随尺寸比例变形，或边框宽度被缩放 | 程序化面板参数错误（blocker） | `03-production-constraints.md` |
| 发光或阴影作为整图被拉伸 | 程序化面板图层错误（blocker） | `03-production-constraints.md` |
| 中心区域出现纹理、噪点或装饰干扰内容 | 程序化面板中心安全区失败（blocker） | `03-production-constraints.md` |
| 角、装饰或固定节点被拉伸，直线段没有只做延长 | 程序化面板缩放策略错误（blocker） | `03-production-constraints.md` |
| 固定角部装饰随面板比例缩放，或跨进可缩放区域后被拉伸 | 面板装饰贴片错误（blocker） | `03-production-constraints.md` |
| 面板装饰超过一个角，或默认位置没有优先使用右下角且其他角不干净 | 面板装饰数量 / 位置错误（blocker） | `03-production-constraints.md` |
| 面板装饰包含文字、logo、角色、徽章、状态图标或具体 IP 物件 | 面板装饰语义越界（blocker） | `03-production-constraints.md` |
| 边中复杂装饰依赖图片拉伸 | 面板装饰拉伸依赖错误（blocker） | `03-production-constraints.md` |
| 出现任何 `panel_base_*.png`、`panel_base_9slice.png` 或 9-slice 面板底图请求 | forbidden image asset 错误（blocker） | `03-production-constraints.md` |
| `programmatic_panel` 被导出为 PNG，而不是 `panel_render_recipe` | non-image output 边界错误（blocker） | `03-production-constraints.md` |
| `state_guidance` 被导出为 `state_*.png`，而不是 `state_visual_rule` | non-image output 边界错误（blocker） | `03-production-constraints.md` |
| 长条组件像被压扁的卡片 | 尺寸边界错误 | `03-production-constraints.md` |
| selected 和 warning 都像红色警告 | 状态语义错误 | `05-material-generation.md` |
| 装饰随机堆叠，干扰阅读 | 装饰语义错误 | `05-material-generation.md` |
| 资源数量暴涨、每个尺寸一张图 | 预算错误 | `03-production-constraints.md` |
| 内容、布局、交互被改了 | 替换范围错误 | `03-production-constraints.md` |

## 回修顺序

1. 如果风格判断错，先回到 `02-style-knowledge.md` 修 style_brief。
2. 如果风格判断正确但背景只剩通用 archetype，先补 `02-style-knowledge.md` 的 `specific_differentiators`、`signature_semantics` 和 `anti_generic_guardrails`。
3. 如果 `style_brief` 有差异锚点但 prompt 没消费，回到 `05-material-generation.md` 修背景 prompt 模板。
4. 如果差异锚点破坏 UI 安全区，回到 `05-material-generation.md` 调整 `background_identity_anchors` 的位置和强度。
5. 如果产物超出 Maker 或预算边界，回到本文件收紧约束。
6. 如果风格正确且传递完整，但生成物料不合格，回到 `05-material-generation.md` 修生成规则。
7. 如果实现参数错误，优先修 Maker 配置，不重新生成图片。
8. 只有非图片输出形态被确认不匹配时，才回修 recipe / rule / Maker 参数；不得扩大 image asset 白名单。
