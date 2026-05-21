# 03 Production Constraints

> 角色：03 Production Constraints / 生产硬约束。定义 Maker 生产链路的全局硬约束。本文件不绑定任何具体游戏风格，也不保存风格特征。

## 输入与输出

输入：

- 已确认的工具页面、组件清单和替换范围。
- `02-style-knowledge.md` 输出的 `style_brief` 引用。
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
- 使用代码程序化绘制面板主体。
- 使用透明 PNG 作为背景和不参与拉伸的固定角部面板装饰。
- 仅在极简、规则、低纹理面板 fallback 中使用 `UI.Panel` 的真实 9-slice / scale9 渲染。
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

默认面板方案为 `programmatic_draw`，不再要求 AI 生成整张 `panel_base_9slice.png`。

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

`panel_base_9slice.png` 不是默认必需资产，只能作为极简、规则、低纹理面板的 fallback。默认不再要求 AI 生成整张 9-slice 面板底图。

## 默认资源预算

默认预算用于“已有工具 UI 的风格替换”，不是完整游戏界面重做。

## 明确生成资产白名单

默认只允许生成以下图片资产：

```yaml
generated_asset_whitelist:
  - asset_name: background_tool.png
    count: 1
    purpose: 页面氛围与世界观语义背景
  - asset_name: panel_corner_accent.png
    count: 1 when global_style_generation_enabled == enabled
    purpose: 固定位置面板角部装饰，不参与拉伸
  - asset_name: panel_accent.png
    count: 1 when global_style_generation_enabled == enabled
    purpose: 固定位置面板角部抽象装饰，不参与拉伸
```

`panel_corner_accent.png` 和 `panel_accent.png` 二选一，默认最多生成 1 张。`global_page_style` 或 `global_style_generation_enabled: enabled` 时，风格化生成请求默认生成其中 1 张固定角部饰品；显式局部元素修改只有在 01 的 `allowed_outputs` 或 `target_image_assets` 明确包含角饰时才可生成。只有 QA 判定会遮挡内容、破坏层级或违反生产约束时，才可记录原因后关闭。除上述白名单外，不允许生成其他独立图片资产。

```yaml
generated_asset_blacklist:
  - deco_*
  - decoration_*
  - ornament_*
  - badge_*
  - icon_*
  - state_*
```

`panel_base_9slice.png` 和 `bar_panel_9slice.png` 都不是默认资产，只能在 fallback 条件成立后申请。

```yaml
asset_budget_default:
  background_image: 1
  panel_programmatic_draw: required
  panel_accent_png: 1 when global_style_generation_enabled == enabled
  panel_9slice_base: 0
  panel_9slice_fallback: 0
  generic_decoration_png: 0
  state_image: 0
  icon_new: 0
  particle_or_animation_asset: 0
  font_new: 0
```

`panel_accent_png` 仅指白名单中的 `panel_corner_accent.png` 或 `panel_accent.png`，二者最多生成 1 张。
`generic_decoration_png: 0` 表示不开放通用 `decoration_*.png`。
`icon_new: 0` 表示当前不允许新增任何独立图标资源；风格语义只能由背景图、程序化面板、固定角部面板装饰、UI primitive 或已有图标库承载。
`page_color_rule` 是非图片规则，不计入生成图片资产预算，但必须只落到 UI primitive、已有组件样式或状态 overlay，不能绕过本文件新增图标、字体或状态图片。

当前禁止新增以下独立装饰 / 图标资产命名：

```yaml
forbidden_standalone_asset_patterns:
  - deco_*
  - decoration_*
  - ornament_*
  - badge_*
  - icon_*
```

优先级：

1. 先使用 UI primitive。
2. 风格化生成请求中背景图必须保留 1 张，用于承载页面氛围和世界观语义。
3. 面板主体默认使用 `programmatic_draw`。
4. 面板装饰默认使用 1 张固定角部 PNG，且不参与拉伸；最多仍为 1 张。
5. 不生成新图标；装饰语义优先由背景图、程序化面板、固定角部面板装饰、UI primitive 或已有图标库承载。
6. 只有确认失败且说明原因后，才允许增加 fallback 资源。

## 资源 fallback 条件

允许新增 `panel_base_9slice.png` 等 fallback 资源必须同时满足：

1. 程序化绘制能力或运行环境已确认不适配当前面板需求。
2. 已在能力允许范围内尝试调整程序化绘制参数、padding、opacity、tint、shadow、primitive 组合。
3. 问题被 QA 归因为“程序化绘制能力 / 运行环境不适配”，不是风格理解错误、prompt 转译错误或实现参数错误。
4. 新增资源的用途、适用组件和禁用范围已写入物料元数据。

默认允许的第一个 fallback：

```yaml
asset: panel_base_9slice.png
for:
  - simple_regular_low_texture_panel
condition: programmatic_draw_capability_or_runtime_confirmed_incompatible
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
- 必须匹配当前目标背景界面的实际尺寸和宽高比，不使用固定默认尺寸。
- 如果当前制作对象是手机界面，背景按该手机界面或背景容器尺寸生成；如果是电脑界面，背景按该电脑界面或背景容器尺寸生成。
- 尺寸来源优先级：用户明确给出的宽高 > 设计稿 / artboard / 当前背景节点尺寸 > 工程配置中的目标 viewport > 当前截图可确认的背景区域。
- 只有设备类型但没有具体宽高时，不得用通用手机或电脑预设尺寸代替；必须在请求中记录缺失尺寸并阻断背景生成，直到获得目标宽高。
- 默认只生成当前目标界面所需的 1 张背景图；多断点、多设备或横竖屏同时适配必须作为明确需求单独声明，不得无理由批量生成多尺寸背景。
- 不允许先生成一个任意比例背景，再依赖后续拉伸、裁切或居中缩放来冒充尺寸匹配。
- 中央 45%-60% 保持低纹理、低对比、低信息密度。
- 装饰优先放在边缘和角落。
- 不得影响正文、表单、列表、按钮和可点击区域识别。

9-slice 图：

- 不是默认面板主体方案，只能在程序化绘制能力或运行环境确认不适配时，作为极简、规则、低纹理面板 fallback。
- 必须是单张透明 PNG 源图。
- 必须有显式 `backgroundSlice`。
- 必须使用真实 sliced 渲染，不允许普通 stretch。
- 中心区域必须可拉伸，角和边必须留在非拉伸区。
- `panel_base_9slice.png` 最多允许一个角落有小型抽象点缀。
- 其他三个角必须干净，只保留基础边框，不允许角花、折角、色片或重复符号。
- 四边必须连续、均匀、可切片拉伸，不允许边中缺口、凸起、阶梯、卡扣、重复结构或明显断裂。
- 外圈 alpha 与切片边缘必须干净，不允许柔光、阴影、光晕、半透明脏边或白边污染。
- 点缀只能体现抽象材质、轮廓或线条感，不能承担图标、徽章、状态或 IP 物件语义。
- 任一违反项都阻断该资源作为基础 `panel_base_9slice.png` 使用。

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

### 9-slice fallback 安全边界

```yaml
panel_base_9slice_fallback_safe_usage:
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
panel_base_9slice_fallback_caution_usage:
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
panel_base_9slice_fallback_forbidden_usage:
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

- 只适用于已批准的 9-slice fallback，不适用于默认程序化面板主体。
- 推荐切片边距为源图尺寸的 8%-14%。
- 如有角部点缀，必须完全落在非拉伸区，且不得破坏相邻边的连续性。
- 对低高度组件，`top + bottom` 不应超过组件高度的 30%-40%。
- 必须满足 `component_height - top - bottom > 0`。
- 必须满足 `component_width - left - right > 0`。
- 坐标和尺寸优先整数对齐，避免细边模糊。

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

| 失败现象 | 优先归因层 | 回修文件 |
|---|---|---|
| 看起来不像目标游戏或关键词 | 风格理解错误 | `02-style-knowledge.md` |
| 背景质量合格，但像通用风格原型，不像目标游戏 | Style Specificity Contract 缺失 | `02-style-knowledge.md` |
| `style_brief` 有差异锚点，但生成 prompt 没有使用 | prompt 消费失败 | `05-material-generation.md` |
| 差异锚点出现了，但破坏 UI 安全区 | identity anchor placement 错误 | `05-material-generation.md` |
| 颜色、材质、形状语言和情绪冲突 | 风格理解错误 | `02-style-knowledge.md` |
| 背景生成前没有采集当前界面或背景容器宽高 | 输入采集失败 | `01-workflow-dispatch.md` |
| 背景输出尺寸或宽高比不匹配当前目标背景界面 | background_rule 执行失败 | `05-material-generation.md` |
| `visual_hierarchy_brief` 允许背景抢内容、中心过花、对比过强或把背景设为错误主焦点 | 视觉层级 brief 错误 | `04-visual-hierarchy-brief.md` |
| `visual_hierarchy_brief` 已限制背景强度、中心细节和中心对比，但背景仍抢内容或中心太花 | prompt / recipe / QA 执行失败 | `05-material-generation.md` |
| `visual_hierarchy_brief` 允许错误主次、组件层级混乱，或把所有矩形都当成完整面板候选 | 视觉层级 brief 错误 | `04-visual-hierarchy-brief.md` |
| `visual_hierarchy_brief` 已清楚约束组件处理、颜色角色和主次，但组件层级仍混乱或所有矩形都套面板 | visual hierarchy 执行失败 | `05-material-generation.md` |
| 面板有白底或截图底 | 图片导出错误 | `03-production-constraints.md` |
| 默认面板主体使用整张 AI 面板图冒充通用底板 | 面板实现方案错误（blocker） | `03-production-constraints.md` |
| 未确认程序化绘制能力 / 运行环境不适配就启用 `panel_base_9slice.png` | fallback 触发条件错误（blocker） | `03-production-constraints.md` |
| 程序化面板没有验证 800x300、300x500、660x440、520x200 | 多尺寸 QA 缺失（blocker） | `03-production-constraints.md` |
| 圆角随尺寸比例变形，或边框宽度被缩放 | 程序化面板参数错误（blocker） | `03-production-constraints.md` |
| 发光或阴影作为整图被拉伸 | 程序化面板图层错误（blocker） | `03-production-constraints.md` |
| 中心区域出现纹理、噪点或装饰干扰内容 | 程序化面板中心安全区失败（blocker） | `03-production-constraints.md` |
| 角、装饰或固定节点被拉伸，直线段没有只做延长 | 程序化面板缩放策略错误（blocker） | `03-production-constraints.md` |
| 固定角部装饰随面板比例缩放，或跨进可缩放区域后被拉伸 | 面板装饰贴片错误（blocker） | `03-production-constraints.md` |
| 面板装饰超过一个角，或默认位置没有优先使用右下角且其他角不干净 | 面板装饰数量 / 位置错误（blocker） | `03-production-constraints.md` |
| 面板装饰包含文字、logo、角色、徽章、状态图标或具体 IP 物件 | 面板装饰语义越界（blocker） | `03-production-constraints.md` |
| 边中复杂装饰依赖图片拉伸 | 面板装饰拉伸依赖错误（blocker） | `03-production-constraints.md` |
| 9-slice 被普通拉伸，边框模糊 | 工程实现错误 | `03-production-constraints.md` |
| `panel_base_9slice.png` 多个角同时出现角花、折角、色片或重复符号 | 9-slice 角部硬约束失败（blocker） | `03-production-constraints.md` |
| `panel_base_9slice.png` 左右或上下边中部出现缺口、凸起、阶梯、卡扣、重复结构或断裂 | 9-slice 边连续性失败（blocker） | `03-production-constraints.md` |
| `panel_base_9slice.png` 外圈或切片边缘有柔光、阴影、光晕、半透明脏边或白边污染 | 9-slice alpha 边缘失败（blocker） | `03-production-constraints.md` |
| `panel_base_9slice.png` 点缀像图标、徽章、状态标记或 IP 物件 | 9-slice 点缀语义越界（blocker） | `03-production-constraints.md` |
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
8. 只有资源形态被确认不匹配时，才进入 fallback 资源申请。
