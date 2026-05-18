# 02 Style Knowledge

> 角色：02 Style Knowledge / 风格理解。根据关键词、标签或游戏名检索游戏视觉特征，并输出 `style_brief`。本文件只负责风格理解，不负责物料生成、资源预算、尺寸边界或 Maker 实现。

## 00. Style Discovery Fallback Rule

当输入游戏不在现有风格知识库中，或 Excel 表中没有明确风格字段时，不要直接使用“通用游戏风格”。
必须先执行 Style Discovery，再决定是匹配已有风格原型、创建新风格原型，还是使用 Generic Fallback。
不要因为同系列、续作、IP 相近或名称相似，就直接继承已有游戏条目的风格。

其中 `maker_safe_assets` 只作为风格可迁移素材线索，不定义资源预算、尺寸边界或 Maker 实现参数。

### 1. Lookup Priority

按照以下顺序查找：

1. game_name 精确匹配现有知识库条目
2. alias 精确匹配现有知识库条目中明确登记的别名
3. 如果 game_name / alias 都没有精确命中，必须先查找 TapTap / 官网 / 商店页文字资料，不允许直接按同系列、续作或 IP 衍生关系套用已有条目
4. 从文字资料中提取：
   - game_type
   - core_gameplay
   - world_setting
   - player_role
   - emotional_experience
   - key_visual_keywords
5. 将提取结果映射到已有 style_archetype
6. 如果没有合适 archetype，则创建新 archetype
7. 如果资料不足，才使用 Generic Adaptive Game UI Fallback

### 2. Style Discovery Method

Style Discovery 必须发生在资料查找之后。

当 game_name / alias 未命中时，必须先查找 TapTap / 官网 / 商店页资料；如果这些资料里没有明确风格字段，再基于已检索到的文字资料按以下问题判断。

以下规则是“搜索后的分析维度”，不是跳过搜索直接猜风格的捷径。

#### A. 玩法类型先定结构

玩法类型只用于确定优先分析方向，不能单独决定最终风格。

- RPG / MMORPG：优先判断幻想、国风、二次元、暗黑、开放世界、回合制等视觉表现
- SLG / 策略：优先判断军事、历史、文明、战争沙盘、地图指挥感
- 射击 / 战术：优先判断现代军事、科幻 HUD、末日废土、竞技电竞
- 生存 / 建造：优先判断荒野、废土、手作、资源管理、安全屋
- 休闲 / 派对 / 消除：优先判断明亮、圆润、糖果、玩具、轻量卡通
- 模拟经营：优先判断生活、店铺、农场、城市、市井、手作
- 卡牌 / 放置：优先判断幻想阵营、角色收集、舞台化展示、符文/徽章系统

#### B. 世界观决定语义

从文字资料中提取最稳定的世界元素：

- 国风：祥云、卷轴、玉佩、灯笼、窗棂、宣纸、古铜
- 现代军事：战术编号、金属板、军用箱、警示条、雷达、坐标线
- 科幻：玻璃 HUD、发光线框、芯片、能量环、舱体、网格
- 废土/末日：旧金属、胶带、警示贴、裂痕、污染标识、临时营地
- 魔幻：符文、宝石、法阵、羊皮纸、金边、石板、魔法光
- 生活模拟：木牌、布料、贴纸、植物、手账、便签、柔和纸感
- 怪物狩猎：皮革、骨片、地图纸、木箱、猎具、爪痕、营地装备
- 像素/复古：像素边、低分辨率图标、街机灯、扫描线、网格块

#### C. 情绪决定强度

- 轻松 / 治愈：低对比、圆角、暖色、柔软阴影
- 紧张 / 生存：低明度、磨损、硬边、警示色克制使用
- 专业 / 战术：高秩序、网格、切角、编号、冷灰/军绿
- 华丽 / 史诗：层级更厚、金属边、宝石点缀、内高光
- 神秘 / 暗黑：深底、低饱和、冷光、符文但不满屏

#### D. UI 材质由世界观转译，不从截图硬抄

不要直接复制原游戏 UI。
只提取可迁移的材质和组件语言：

- panel material
- border shape
- state color
- accent icon
- background atmosphere
- maker_safe_assets

#### E. 原型 vs 差异锚点

Style Discovery 不能停在大风格原型，必须继续提取差异化锚点：

1. 这个游戏属于哪个大风格原型。
2. 它和同原型游戏相比，最容易被混淆成谁。
3. 它不能只靠哪些通用元素表达。
4. 哪些元素是低密度工具背景中仍可安全保留的识别点。
5. 哪些元素会破坏 UI 安全区，必须只放边缘、角落或远景。
6. 如果背景图只保留大风格原型，会变成什么错误结果。

只有完成以上判断，才允许写入 `style_brief` 并进入物料生成。

### 3. Decision Output

每个未命中游戏必须输出：

```yaml
style_match_status: exact_match | alias_match | archetype_match | new_archetype_needed | generic_fallback
matched_style_entry:
reason:
source_basis:
confidence:
next_action:
```

## 使用边界

本文件可以回答：

- 某个游戏或关键词对应什么视觉身份、情绪和世界观语义。
- 颜色、材质、形状语言和 UI 翻译方向应该如何理解。
- 哪些误判方向必须避免。
- 应该输出哪些可检索标签，供后续生产流程引用。
- 程序化绘制面板主体和固定局部装饰可以参考哪些材质、形状、色彩和 panel 转译方向。

本文件不能回答：

- 要生成几张图片、图片尺寸是多少、9-slice 参数是多少。
- 背景图、面板图、装饰图、状态图的具体产物规格。
- Maker 支持什么能力、资源预算是多少、QA 失败归因到哪一层。
- 页面视觉层级、背景强度、组件主次或色彩角色分配。

这些问题按职责分别处理：

- `03-production-constraints.md` 负责生产约束。
- `04-visual-hierarchy-brief.md` 负责页面视觉层级 brief。
- `05-material-generation.md` 负责 prompt / recipe / metadata / QA 执行。

## Programmatic Panel 风格输入边界

程序化面板绘制和固定局部装饰语义转译可以把 `style_brief` 当作视觉方向来源，但只能消费可转译为“主体材质、底色倾向、边框轮廓、线条气质、发光/阴影气质、抽象局部装饰语义候选”的字段。

程序化面板的主要风格输入字段：

```yaml
programmatic_panel_style_fields:
  - materials
  - shape_language
  - ui_translation.panel_base_style_hint
  - colors
  - forbidden_mistakes
```

风格拆分方向：

```yaml
panel_style_decomposition:
  scalable_body_style:
    - base_fill_material
    - center_surface_color
    - border_shape
    - stroke_profile
    - glow_or_shadow_mood
    - edge_light_color
    - inner_highlight_style
  fixed_accent_style:
    - abstract_corner_fold
    - small_material_chip
    - ribbon_or_lace_fragment
    - linework_or_outline_motif
```

只作为语义理解和误读过滤上下文的字段：

```yaml
programmatic_panel_context_only_fields:
  - ui_translation.panel
  - ui_translation.accent
  - world_elements
  - signature_semantics.must_include
  - signature_semantics.must_avoid
  - specific_differentiators
  - anti_generic_guardrails
```

`programmatic_panel_context_only_fields` 只能辅助理解风格语义、避免通用化和过滤误读，不能把其中的角色、世界观物件、徽章、状态、标签、布局组合、文字、图标、logo 或 IP 符号直接转成面板主体或局部装饰。

`ui_translation.accent` 只能作为局部装饰语义筛选或误读过滤上下文，不能直接把其中的角色、图标、徽章、IP 物件、状态标记画进面板主体或固定局部装饰。

`ui_translation.panel` 描述完整面板系统，可以包含大卡、小卡、头像、标签、状态、徽章、角标和布局组合建议；它不是程序化面板主体规格，也不是固定局部装饰的直接绘制清单。

`ui_translation.panel_base_style_hint` 不再表示“让 AI 生成整张基础面板图”。它只描述面板主体和局部装饰可借用的风格线索，例如主体材质、底色倾向、边框轮廓、线条气质、发光/阴影气质、抽象局部装饰候选，以及不适合作为基础主体或局部装饰来源的具体物件。

`scalable_body_style` 只能描述适合被程序化还原的连续面板主体语言，例如填充材质、中心面颜色、边框形状、描边气质、边缘光、内高光和阴影气质。

`fixed_accent_style` 只能描述不参与拉伸的抽象局部装饰语义候选，例如折角、材质嵌片、缎带 / 蕾丝碎片、线描或轮廓母题。02 只提供可借用的风格联想，不决定最终是否使用装饰，也不规定装饰数量、装饰位置、像素尺寸、渲染 API、资源数量、文件名或 QA 多尺寸列表。

示例：无限暖暖这类风格可以表达为“暖白 / 极淡粉白纸卡或低饱和柔光底，柔圆纸卡边，细珍珠或淡金边；可选局部装饰可联想缎带折角、小纽扣、蕾丝角片或单片花瓣边角；但大蝴蝶结、心形吊坠、花束、衣服、相机、角色和复杂蕾丝不作为基础主体或局部装饰来源”。

## 检索方式

1. 先执行 `00. Style Discovery Fallback Rule` 中的 Lookup Priority。
2. 只接受 game_name 精确匹配或已登记 alias 精确匹配作为已有条目命中。
3. 未命中 game_name / alias 时，必须先查找 TapTap / 官网 / 商店页资料。
4. 搜索并提取文字资料后，才允许用 `tags`、玩法品类、情绪词和世界观元素映射已有 style_archetype。
5. 禁止在未搜索资料的情况下，按同系列、续作、IP 相近、名称相似或主观印象直接套用已有条目。
6. 搜索后仍没有合适 archetype 时，输出 Decision Output，决定创建新 archetype 还是使用 Generic Adaptive Game UI Fallback。
7. 输出给生产链路时，只输出 `style_brief` 或 Style Discovery 决策结果，不要把生产参数混入其中。

## Style Specificity Contract

`style_brief` 不是“风格大类标签”，而是具名游戏的可识别风格描述。传递给 `05-material-generation.md` 时，不允许把完整风格理解压缩成通用 archetype。

### 需要显式拆分

每个 `style_brief` 必须区分：

- `style_archetype`：这个游戏属于哪类大风格。
- `archetype_shared_traits`：同原型游戏都可能拥有的共性。
- `specific_differentiators`：它和同原型游戏相比，最关键的差异。
- `signature_semantics`：低密度背景里仍应保留、减少或避免的识别语义。
- `background_identity_anchors`：哪些语义可放边缘、远景或中央安全区。
- `anti_generic_guardrails`：防止生成“漂亮但谁都能用”的通用图。
- `visual_identity_test`：判断结果是否仍能区别于同原型游戏。

### 禁止压缩成

- 通用题材：例如“二次元科幻”“国风仙侠”“现代军事”“温馨治愈”。
- 通用氛围：例如“低对比、柔和、边缘装饰、中心留白”。
- 通用颜色：例如“蓝紫科幻”“奶油白木纹”“军绿灰黑”。
- 通用 UI 词：例如“圆角卡片、半透明面板、柔和阴影”。

这些信息可以作为 `archetype_shared_traits` 使用，但不能替代差异锚点。

### 旧条目补全规则

如果现有条目尚未显式填写 Style Specificity Contract，使用该条目前必须先从 `visual_identity`、`genre_presentation`、`game_features.core_gameplay`、`world_elements`、`materials`、`shape_language`、`background_visual_language` 和 `forbidden_mistakes` 中补全这些字段，再进入 05 生成物料。

如果旧条目没有 `ui_translation.panel_base_style_hint`，不能直接把 `ui_translation.panel` 当成程序化面板规格。必须先从 `materials`、`shape_language`、`colors` 和 `ui_translation.panel` 中抽取主体材质、中心底色倾向、边框轮廓、线条气质、发光/阴影气质和可选局部装饰候选，并过滤掉头像、状态、徽章、标签、文字、图标、logo、角色、具体世界观物件和布局组合。

### 传递失败判定

如果 05 生成的 prompt 只剩下大类题材、通用颜色、通用材质和中心留白规则，即使产物满足 UI 可读性，也判定为 `style_specificity_loss`。

## style_brief 输出契约

```yaml
style_brief:
  name: string
  aliases: []
  tags: []
  source:
    checked: string
    confidence: high | medium | low
  game_features:
    type: string
    core_gameplay: string
  visual_identity: string
  genre_presentation: string
  mood: string
  world_elements: string
  colors: string
  materials: string
  shape_language: string
  background_visual_language: string
  ui_translation:
    panel: string
    panel_base_style_hint: string
    state: string
    text: string
    accent: string
  forbidden_mistakes: string
  style_archetype: string
  archetype_shared_traits:
    - string
  specific_differentiators:
    - string
  signature_semantics:
    must_include:
      - string
    should_include:
      - string
    should_reduce:
      - string
    must_avoid:
      - string
  background_identity_anchors:
    foreground_forbidden:
      - string
    edge_or_corner_allowed:
      - string
    far_background_allowed:
      - string
    center_safe_area_allowed:
      - string
  anti_generic_guardrails:
    - string
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: string
```

## 条目

## 01. 心动小镇

```yaml
style_brief:
  name: "心动小镇"
  aliases: []
  tags: ["cozy", "life-sim", "town", "handmade"]
  source:
    checked: "https://www.taptap.cn/app/45213"
    confidence: "high"
  game_features:
    type: "生活模拟 / 社交 / 建造装饰"
    core_gameplay: "开放小镇生活、家园布置、钓鱼采集、社交与轻压力日常循环"
  visual_identity: "温暖治愈的手作小镇生活模拟"
  genre_presentation: "把生活模拟视觉化为柔软、低压力、可亲近的社区日常，而不是写实城镇或硬核经营。"
  mood: "温暖、轻松、松弛、陪伴感"
  world_elements: "小镇街角、木屋、手作家具、花草、便签、邮票、路牌、咖啡、海边微风"
  colors: "奶油白作面板底；浅木色/暖棕作边框；草绿/天蓝作辅助；珊瑚橙/暖黄作选中和奖励，不用高饱和红做普通选中"
  materials: "浅色纸卡、浅木纹、布料贴纸、半透明磨砂卡片"
  shape_language: "大圆角、软阴影、手绘不规则边、贴纸角标、轻量卡片层叠"
  background_visual_language: "浅暖天空或小镇远景，中心留出干净低纹理区域，边缘放树影、屋檐、路牌和花草"
  ui_translation:
    panel: "面板用奶油白半透明卡片配浅木边，普通卡简化贴纸纹理，大卡可加手绘角花和轻微纸张颗粒"
    panel_base_style_hint: "奶油白半透明纸卡或浅木边手作卡底，软圆角、浅木描边和轻纸张颗粒；可联想邮票边、手写标签、布料贴片边角，但小房子、猫爪、花草堆、路牌和功能贴纸不作为基础主体或局部装饰来源"
    state: "selected 用暖黄描边+小贴纸；disabled 降低饱和和透明度；locked 用小锁牌；warning 才用柔和橙红"
    text: "标题可用圆润手写感，正文使用清晰黑棕色；小字禁止强描边"
    accent: "树叶、邮票、手写标签、木牌、花瓣、小房子、猫爪。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "被误判成真实田园、儿童绘本或高饱和农场经营"
```

## 02. 潜水员戴夫

```yaml
style_brief:
  name: "潜水员戴夫"
  aliases: []
  tags: ["pixel", "underwater", "restaurant", "adventure"]
  source:
    checked: "https://www.taptap.cn/app/720041"
    confidence: "high"
  game_features:
    type: "冒险 / 经营 / 像素"
    core_gameplay: "白天潜水探索、捕鱼收集，夜晚寿司店经营的混合玩法"
  visual_identity: "复古像素海底冒险 + 寿司店经营"
  genre_presentation: "把冒险和经营视觉化为蓝绿色海底深度、像素物件和夜间小店烟火气的双场景切换。"
  mood: "幽默、探索、轻松、海底神秘"
  world_elements: "潜水装备、海藻、鱼群、氧气瓶、寿司柜台、木质菜单、像素道具"
  colors: "深海蓝/青绿作背景；米白作菜单面板；木棕作经营区；橙黄作新鲜度和奖励；红色只作氧气/危险"
  materials: "像素化玻璃面板、深海蓝透明板、木牌菜单、餐厅纸卡"
  shape_language: "像素边、矩形块、轻量描边、小图标网格"
  background_visual_language: "海底渐层或寿司店模糊远景，中央安全区不要有密集鱼群和强水纹"
  ui_translation:
    panel: "深蓝半透明卡或木质菜单卡，像素描边控制在 1-2 层"
    panel_base_style_hint: "深海蓝透明像素玻璃或米白木质菜单卡底，像素块描边、木牌边和轻气泡质感；可联想气泡边、菜单纸角、像素木片，但鱼群、寿司图形、氧气表和海底场景不作为基础主体或局部装饰来源"
    state: "selected 用气泡高光或橙黄菜单贴；locked 用小锁/深海阴影；warning 用氧气红"
    text: "标题可用复古像素字，正文保持常规清晰字体"
    accent: "气泡、鱼影、寿司、氧气表、像素箭头、木牌价格签。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成纯海洋治愈或纯餐厅经营，忽略像素幽默与双场景"
```

## 03. 鹅鸭杀

```yaml
style_brief:
  name: "鹅鸭杀"
  aliases: []
  tags: ["party", "deduction", "cartoon", "social"]
  source:
    checked: "https://www.taptap.cn/app/258720"
    confidence: "high"
  game_features:
    type: "社交推理 / 派对"
    core_gameplay: "多人身份推理、任务、会议投票与阵营欺骗"
  visual_identity: "卡通鸟类社交推理派对"
  genre_presentation: "把狼人杀式推理视觉化为圆润鸟类角色、轻松搞笑与任务地图中的悬疑反差。"
  mood: "搞笑、紧张、友尽、轻悬疑"
  world_elements: "鹅、鸭、会议桌、投票牌、任务清单、脚印、警报、可疑标记"
  colors: "浅灰/米白作面板；蓝绿作安全/任务；黄色作讨论和提醒；红色仅用于警报、淘汰或危险"
  materials: "扁平塑料卡、白板纸、任务便签、圆形徽章"
  shape_language: "圆润、厚描边、气泡式对话框、投票卡片"
  background_visual_language: "简单地图舱室或农场/空间站远景，边缘放脚印和任务纸，中央干净"
  ui_translation:
    panel: "圆角厚卡+投票便签风，大卡可加头像圆章，小卡只保留身份/状态"
    panel_base_style_hint: "浅灰/米白厚纸卡或扁平塑料卡底，圆润厚边和投票便签材质感；可联想胶带角、纸片折角、贴边投票便签，但头像圆章、身份文字、状态标记和角色图标不作为基础主体或局部装饰来源"
    state: "selected 用黄色投票框；warning 用红色警报灯；disabled 灰化但保留轮廓"
    text: "标题可圆润粗体，正文黑灰；状态词用高对比标签"
    accent: "羽毛、脚印、问号、投票贴纸、警报灯、身份徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成儿童低龄卡通；需要保留“可疑/投票/推理”语义"
```

## 04. 三角洲行动

```yaml
style_brief:
  name: "三角洲行动"
  aliases:
    - "Delta Force"
    - "三角洲行动手游"
  tags:
    - "战术射击"
    - "搜打撤"
    - "生存"
    - "FPS"
    - "多人联机"
    - "军事写实"
  source:
    checked: "TapTap 页面：游戏名、搜打撤/生存/射击/动作标签，以及新一代战术射击、交战搜索撤离、24v24 海陆空载具协同、重火力破坏、随机事件、电影级剧情战役描述。"
    confidence: high
  game_features:
    type: "写实战术射击游戏"
    core_gameplay: "搜打撤、多人战场、载具协同、场景破坏、剧情战役、随机事件"
  visual_identity: "现代军事写实与高规格战术 UI 结合，强调危险、专业、压迫、装备可信度和战场信息控制。"
  genre_presentation: "以真实战术语境包装多模式射击，不是夸张英雄射击，也不是玩具化军事。"
  mood: "紧张、冷峻、专业、危险、硬核、电影化；情绪来自低饱和战场色、战术装备、烟尘、爆炸、夜视和信息化 HUD。"
  world_elements: "特战队员、战术装备、撤离点、物资箱、海陆空载具、破坏场景、重火力、随机战场事件、战役任务。"
  colors: "主色倾向军绿、沙褐、灰黑、铁灰、低饱和蓝；强调色多为战术橙、警示红、HUD 青绿；对比克制但关键状态强烈。"
  materials: "写实军工材质；尼龙织物、凯夫拉、防弹板、磨砂金属、橡胶、泥沙、混凝土、烟雾和火光；边缘有磨损和战场脏污。"
  shape_language: "硬边矩形、模块化装备块、斜切战术卡、网格、坐标线、扫描框、军工标签和危险警示条。"
  background_visual_language: "背景适合战区废墟、港口、荒漠、工业设施、烟尘、夜战和远处载具轮廓；安全倾向是低饱和、强空间纵深、中心留出战术信息区。"
  ui_translation:
    panel: "面板系统偏军用终端和战术 HUD，大卡像任务档案、装备仓或地图面板，小卡像物资栏、战备槽、队伍状态；标签偏窄、硬、斜切，状态层强调威胁、撤离、品质、耐久、队友和资源风险。"
    panel_base_style_hint: "深色军工塑料、磨砂金属、战术布料或旧终端底色，硬边矩形、斜切角、细网格线、低亮 HUD 光和厚重阴影；可选局部装饰可联想磨损边角、警示条纹、扫描线、坐标网格、战术贴片纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别武器、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏高风险战术信息，常见警示色、扫描框、品质色、血量护甲、倒计时、撤离提示和地图标记。"
    text: "字体偏窄体、工业、无衬线、强可读；标题可大写化、压缩字宽，正文像战术终端信息。"
    accent: "语义候选包括 HUD 青绿、战术橙、坐标网格、警示条、扫描框、装备仓、撤离标记、军工磨损。"
  forbidden_mistakes: "不要误判成卡通吃鸡、科幻机甲射击、赛博朋克、英雄技能射击或普通丧尸生存。"
  style_archetype: "现代军事写实战术射击"
  archetype_shared_traits:
    - "低饱和战场色"
    - "硬边军工 UI"
    - "装备和物资可信度"
    - "战术信息优先"
  specific_differentiators:
    - "搜打撤风险感与大战场协同并存"
    - "电影化战役和真实战术装备结合"
    - "HUD 要专业克制，不应娱乐化"
    - "重火力破坏提升战场压迫"
  signature_semantics:
    must_include:
      - "现代战术军工质感"
      - "搜打撤风险信息"
      - "硬边 HUD 和坐标网格"
      - "低饱和烟尘战场"
    should_include:
      - "装备仓和物资格语义"
      - "警示条与扫描线"
      - "载具协同远景"
      - "磨损金属和战术布料"
    should_reduce:
      - "鲜艳卡通色"
      - "夸张英雄技能光效"
      - "圆润玩具按钮"
      - "纯科幻蓝光界面"
    must_avoid:
      - "香肠派对式搞怪卡通"
      - "赛博霓虹城市"
      - "二次元角色中心"
      - "直接照搬真实枪械型号、官方 logo 或部队徽章"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别干员、武器、军徽、官方 logo 或具体载具作为前景主体"
    edge_or_corner_allowed:
      - "警示条纹"
      - "坐标网格"
      - "磨损金属片"
      - "战术织物纹理"
    far_background_allowed:
      - "战区废墟"
      - "工业设施"
      - "烟尘火光"
      - "远处载具剪影"
    center_safe_area_allowed:
      - "低饱和烟雾"
      - "暗色终端渐变"
      - "弱网格和地图纹理"
  anti_generic_guardrails:
    - "必须有战术信息感，不能只做战争背景。"
    - "军事写实要克制，不要赛博化。"
    - "搜打撤需要风险和物资语义，不只是 FPS 枪战。"
    - "UI 形状要硬朗，不要圆润卡通。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出现代战术、搜打撤、军工 HUD、低饱和战场和高风险装备信息。"
```

## 05. 明日方舟：终末地

```yaml
style_brief:
  name: "明日方舟：终末地"
  aliases: []
  tags: ["sci-fi", "industrial", "arknights", "exploration"]
  source:
    checked: "https://www.taptap.cn/app/232326"
    confidence: "medium"
  game_features:
    type: "科幻 RPG / 策略建造 / 开放探索"
    core_gameplay: "行星开拓、工业设施、角色协作与资源链条构建"
  visual_identity: "荒星工业科幻开拓 RPG"
  genre_presentation: "把探索和建造视觉化为荒星基地、工业终端、轨道物流、矿物能源和低调冷硬的企业设施感。"
  mood: "克制、专业、荒凉、探索"
  world_elements: "荒星地表、基地终端、轨道运输、矿石、能源管线、干员装备、工业标识"
  colors: "灰黑/岩土色作底；白灰作文字；青蓝作系统能源；橙黄作警示/交互；红色只作事故"
  materials: "工业终端玻璃、金属板、磨砂黑面、警示胶带、矿石能量条"
  shape_language: "硬边切角、模块化网格、编号块、细线 HUD"
  background_visual_language: "荒星地貌或基地墙体远景，中央保持低对比网格"
  ui_translation:
    panel: "模块化硬边卡+工业细线，大卡可加入管线/能源槽，小卡简化"
    panel_base_style_hint: "灰黑工业终端玻璃或磨砂金属板底，模块化硬边、切角细线和冷青能源感；可联想管线嵌片、工业编号、警示胶带边，但干员装备、矿石、轨道物流图和基地设施不作为基础主体或局部装饰来源"
    state: "selected 用青蓝光条或橙黄任务边；warning 与 selected 颜色区分"
    text: "标题冷静工业字，正文高可读无衬线"
    accent: "管线、矿石、工业编号、警示条、物流箭头、基地图标。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成传统明日方舟医疗黑白 UI 或通用未来蓝"
```

## 06. 我的世界：移动版

```yaml
style_brief:
  name: "我的世界：移动版"
  aliases: []
  tags: ["voxel", "sandbox", "crafting", "survival"]
  source:
    checked: "https://www.taptap.cn/app/43639"
    confidence: "high"
  game_features:
    type: "沙盒 / 生存 / 建造"
    core_gameplay: "方块采集、合成、建造、生存探索"
  visual_identity: "体素方块沙盒生存建造"
  genre_presentation: "把沙盒视觉化为像素方块、合成格、物品栏和可拼装的世界材料。"
  mood: "自由、创造、探索、轻冒险"
  world_elements: "草方块、木板、矿石、工作台、背包格、火把、红石、洞穴"
  colors: "草绿/泥棕作自然底；石灰作结构；木棕作面板；青钻/红石作稀有和状态；红色只作受伤"
  materials: "像素方块、木板、石砖、物品栏槽、半透明黑底"
  shape_language: "直角、像素边、8-bit 网格、块面拼接"
  background_visual_language: "低细节体素地形远景，中央干净块状渐变"
  ui_translation:
    panel: "木板/石砖九宫格面板，按钮直角或小圆角像物品槽"
    panel_base_style_hint: "木板/石砖像素卡底，直角块面、物品栏槽边和低分辨率拼接感；可联想合成格、矿石色块、木板接缝，但草方块、工具、火把和洞穴场景不作为基础主体或局部装饰来源"
    state: "selected 用白色像素描边或钻石青；locked 用基岩灰"
    text: "标题可像素体，正文保持清晰无衬线/像素大字"
    accent: "方块、镐子、矿石、火把、合成格、像素箭头。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通像素复古，必须保留体素方块和合成语义"
```

## 07. 王者荣耀

```yaml
style_brief:
  name: "王者荣耀"
  aliases: []
  tags: ["moba", "heroic", "fantasy", "competitive"]
  source:
    checked: "https://www.taptap.cn/app/2301"
    confidence: "high"
  game_features:
    type: "MOBA / 竞技 / 英雄"
    core_gameplay: "5v5 英雄对战、技能成长、皮肤和赛事化竞技"
  visual_identity: "华丽东方幻想竞技 MOBA"
  genre_presentation: "把 MOBA 视觉化为英雄殿堂、战场徽章、金色段位、技能符文和高 polish 竞技面板。"
  mood: "热血、荣耀、华丽、竞技"
  world_elements: "英雄徽章、王者冠冕、战场地图、技能符文、段位牌、峡谷能量"
  colors: "深蓝/靛紫作底；金色作荣耀与选中；白色作文字；红蓝区分敌我；绿色仅作生命/安全"
  materials: "深蓝玻璃、金属金边、玉石/水晶、旗帜布纹"
  shape_language: "对称构图、金边浮雕、圆角+尖角混合、徽章化"
  background_visual_language: "深蓝峡谷/宫殿远景，中央避免复杂英雄和强光"
  ui_translation:
    panel: "大卡可用金边浮雕，小卡使用简化金线+深蓝底"
    panel_base_style_hint: "深蓝玻璃或旗帜布纹卡底，细金属边、圆角与尖角混合的徽章化轮廓；可联想段位牌边、符文细线、冠冕轮廓片，但英雄徽章、技能符文主体、战场地图和阵营标识不作为基础主体或局部装饰来源"
    state: "selected 用金色光圈/段位星；warning 用红色敌情，不作普通装饰"
    text: "标题可高对比华丽，正文白色或浅金但需足够对比"
    accent: "冠冕、段位星、符文、旗帜、剑盾、战场路线。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成泛蓝紫魔幻手游，需保留竞技和荣耀段位语义"
```

## 08. 原神

```yaml
style_brief:
  name: "原神"
  aliases: []
  tags: ["elemental-fantasy", "open-world", "anime", "adventure"]
  source:
    checked: "https://www.taptap.cn/app/168332"
    confidence: "high"
  game_features:
    type: "开放世界 RPG / 二次元幻想"
    core_gameplay: "七国探索、元素反应、角色养成与冒险解谜"
  visual_identity: "明亮诗意的元素幻想开放世界"
  genre_presentation: "把幻想 RPG 视觉化为自然地貌、元素符号、冒险者图鉴、古典纹样和轻透明界面。"
  mood: "明亮、诗意、冒险、神秘"
  world_elements: "元素符号、风之翼、地图、七国建筑、蒲公英、晶蝶、冒险手册"
  colors: "米白/浅灰作面板；金色作高级和选中；青绿/天蓝作探索；元素色作状态；红色只作危险/火元素"
  materials: "浅色羊皮纸、半透明玻璃、石纹、金属细边、元素晶石"
  shape_language: "圆角、细金边、卷草纹、轻浮雕、图鉴卡"
  background_visual_language: "柔和天空/自然远景，边缘放建筑和元素纹样，中心干净"
  ui_translation:
    panel: "浅纸卡+细金边+元素小徽章，小卡避免复杂花纹"
    panel_base_style_hint: "米白羊皮纸或浅色磨砂玻璃卡底，细金边、柔圆角和轻古典浮雕感；可联想冒险手册页角、元素晶石色边、卷草细线，但元素徽记、晶蝶、羽毛和七国建筑不作为基础主体或局部装饰来源"
    state: "selected 用金色细边+元素光；warning 用红色火焰图标，不作通用选中"
    text: "标题可古典细腻，正文深灰或白色高对比"
    accent: "元素徽记、晶蝶、羽毛、地图针、卷草纹、星星。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用二次元魔幻或高饱和手游"
```

## 09. 香肠派对

```yaml
style_brief:
  name: "香肠派对"
  aliases:
    - "Sausage Man"
    - "香肠派对手游"
  tags:
    - "卡通射击"
    - "吃鸡"
    - "TPS"
    - "多人联机"
    - "搞怪派对"
    - "竞技"
  source:
    checked: "TapTap 页面：游戏名、截图入口、射击/吃鸡/卡通/TPS/多人联机标签，以及“搞怪魔性、内容丰富且手感出众的卡通射击派对游戏”描述。"
    confidence: high
  game_features:
    type: "卡通派对射击竞技游戏"
    core_gameplay: "吃鸡、搜打撤、多人竞技、团队合作、活动玩法"
  visual_identity: "以香肠人身体比例为核心识别的荒诞卡通射击风，军事竞技被软化成玩具化、派对化和整蛊化体验。"
  genre_presentation: "把传统战术射击转译成低门槛、强娱乐、强角色记忆点的卡通战场。"
  mood: "搞怪、轻松、热闹、竞技、魔性；情绪来自圆柱角色、夸张表情、玩具枪械、亮色地图和派对式道具。"
  world_elements: "肠岛、香肠人、玩具化武器、载具、机甲、赛季主题、摸金撤离、怪物、补给、夸张战备物资。"
  colors: "整体高饱和、高明度，常见蓝天草地、糖果色建筑、鲜亮服装和高对比赛季色；强调色跳脱，服务搞怪识别和竞技反馈。"
  materials: "半写实卡通材质；塑料、橡胶、软糖、亮面金属、玩具武器感明显；高光干净，阴影柔和，边缘圆滑。"
  shape_language: "圆柱、胶囊、泡泡、软边矩形、厚描边、夸张比例；装饰母题偏贴纸、徽章、补给箱、玩具零件和派对符号。"
  background_visual_language: "背景可采用明亮岛屿、竞技场、赛季主题空间和玩具化战场；安全倾向是色块清晰、冲突轻松、危险元素被卡通化。"
  ui_translation:
    panel: "面板偏厚实圆角、亮色卡片、贴纸化标签和玩具按钮；状态层常用大图标、徽章、赛季标识、红点和夸张动效语义；内容组合层强调热闹、活动感和易读性。"
    panel_base_style_hint: "亮面塑料、软胶或玩具包装感底色，圆润厚边、轻快描边、软阴影与弹性质感；可选局部装饰可联想贴纸折角、胶囊轮廓、泡泡高光、软塑料纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别道具、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏夸张和即时，常见弹跳、闪光、贴纸章、彩色描边、任务奖励爆点。"
    text: "字体偏粗圆、活泼、游戏综艺感；标题可有厚描边、渐变和阴影，正文保持清晰友好。"
    accent: "语义候选包括胶囊轮廓、派对贴纸、玩具补给、亮色赛季光效、软塑料高光、夸张警示符号。"
  forbidden_mistakes: "不要误判成严肃军武吃鸡、硬核战术射击、低龄纯儿童卡通、普通糖果休闲或二次元射击。"
  style_archetype: "搞怪卡通竞技射击"
  archetype_shared_traits:
    - "军事题材被玩具化"
    - "高饱和色彩与圆润造型"
    - "强贴纸化 UI 和派对反馈"
    - "危险感被幽默感抵消"
  specific_differentiators:
    - "香肠人胶囊身体是核心识别"
    - "射击竞技与整蛊派对并存"
    - "赛季主题可以强变体但底层仍保持软胶卡通"
    - "武器和载具更像玩具而非真实装备"
  signature_semantics:
    must_include:
      - "圆润胶囊化角色语义"
      - "搞怪派对射击气质"
      - "亮色玩具化战场"
      - "贴纸化 UI 反馈"
    should_include:
      - "软塑料高光"
      - "夸张补给与赛季感"
      - "玩具武器轮廓"
      - "轻松竞技氛围"
    should_reduce:
      - "真实枪械压迫感"
      - "暗色军事写实"
      - "过细科技线框"
      - "严肃血腥表现"
    must_avoid:
      - "写实战争废墟"
      - "冷硬特种兵 UI"
      - "成人向血腥暴力"
      - "直接照搬香肠角色或 logo"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置香肠人主角、官方 logo、赛季徽章或可识别武器作为前景主体"
    edge_or_corner_allowed:
      - "抽象胶囊边角"
      - "贴纸折角"
      - "玩具包装纹理"
      - "泡泡高光"
    far_background_allowed:
      - "卡通岛屿"
      - "明亮竞技场"
      - "玩具化建筑剪影"
      - "轻松赛季主题空间"
    center_safe_area_allowed:
      - "清亮渐变"
      - "低对比云朵或草地色块"
      - "软塑料浅纹理"
  anti_generic_guardrails:
    - "必须同时保留射击竞技和搞怪派对感。"
    - "卡通不等于幼儿化，要保留竞技节奏。"
    - "武器元素应玩具化，不能转向真实军武。"
    - "亮色背景不能变成普通休闲糖果风。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出胶囊香肠式搞怪射击、玩具化战场、贴纸 UI 和派对竞技气质。"
```

## 10. 解压泡泡

```yaml
style_brief:
  name: "解压泡泡"
  aliases:
    - "解压泡泡测试"
  tags:
    - "休闲"
    - "解压"
    - "泡泡"
    - "小游戏"
    - "轻量"
  source:
    checked: "TapTap 页面：游戏名、测试状态、休闲标签、上线日期 2026-05-16，以及“一款很有趣的休闲解压小游戏”描述。"
    confidence: medium
  game_features:
    type: "休闲解压小游戏"
    core_gameplay: "点击、消除、触发泡泡反馈、轻量关卡体验"
  visual_identity: "以泡泡、柔软、弹性和即时反馈为核心的轻休闲视觉，强调无压力、可爱和连续释放感。"
  genre_presentation: "不靠复杂世界观，而靠触感联想、明快色彩和简单反馈建立可玩性。"
  mood: "轻松、舒缓、清爽、愉快、低负担；情绪来自圆形泡泡、柔和渐变、弹性形变和清脆反馈联想。"
  world_elements: "泡泡、气泡群、软糖质感、清透液体、轻量关卡、简单障碍、奖励反馈。"
  colors: "倾向浅亮、低压、糖果色和清透色；常见粉、蓝、紫、薄荷绿、奶白；对比柔和，强调干净和治愈。"
  materials: "偏卡通扁平加轻 3D 质感；透明泡泡、软胶、果冻、玻璃高光、水润渐变；阴影轻，边缘圆滑。"
  shape_language: "圆形、椭圆、泡泡簇、软边矩形、漂浮小粒子；重复母题是圆点、气泡、高光弧线和轻弹性曲线。"
  background_visual_language: "背景适合清爽浅色、柔和渐变、稀疏气泡和低对比纹理；安全倾向是不抢主体、保持干净、避免复杂叙事。"
  ui_translation:
    panel: "面板偏轻薄圆角、半透明或果冻卡片，小卡像软糖贴片，标签像圆润气泡，状态层以弹出、闪光、连击和奖励粒子表达。"
    panel_base_style_hint: "半透明泡泡、果冻软胶或浅色磨砂底色，圆润无锐角边框、柔和内高光、轻浮阴影；可选局部装饰可联想气泡弧线、果冻折光、软糖贴片、圆点纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别道具、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈强调释放和满足感，适合泡泡破裂、轻闪、粒子散开、连击数字和柔和震动语义。"
    text: "字体偏圆润、亲切、轻量；标题可有软描边或果冻高光，正文简单清晰。"
    accent: "语义候选包括泡泡高光、果冻折射、软糖圆角、清透水感、舒缓粒子。"
  forbidden_mistakes: "不要误判成复杂三消、儿童教育、海洋冒险、医疗清洁或重度卡通经营。"
  style_archetype: "清爽泡泡解压休闲"
  archetype_shared_traits:
    - "圆形重复带来触感暗示"
    - "浅色渐变降低压力"
    - "反馈比叙事更重要"
    - "材质柔软透明"
  specific_differentiators:
    - "核心是解压反馈而非策略消除"
    - "泡泡语义应保持轻盈清透"
    - "画面不宜过度堆叠系统感"
    - "触感想象比角色识别更重要"
  signature_semantics:
    must_include:
      - "泡泡圆形母题"
      - "清透柔软质感"
      - "轻松低压氛围"
      - "即时解压反馈"
    should_include:
      - "浅色糖果渐变"
      - "圆润软边 UI"
      - "轻微粒子和高光"
      - "果冻或水润联想"
    should_reduce:
      - "复杂纹理"
      - "强对抗色"
      - "厚重金属边"
      - "暗黑背景"
    must_avoid:
      - "写实污渍清洁风"
      - "硬核消除棋盘风"
      - "恐怖黏液感"
      - "直接照搬 TapTap 截图里的具体图标或 logo"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别 logo、关卡图标或特定截图 UI 作为前景主体"
    edge_or_corner_allowed:
      - "气泡群"
      - "果冻高光弧"
      - "浅色圆点"
      - "软糖贴片"
    far_background_allowed:
      - "淡色渐变"
      - "漂浮泡泡"
      - "轻量抽象空间"
      - "低对比液体纹理"
    center_safe_area_allowed:
      - "浅蓝、浅粉或奶白清爽留白"
      - "稀疏气泡"
      - "柔和光斑"
  anti_generic_guardrails:
    - "必须突出泡泡触感和解压反馈，不能只做普通休闲按钮。"
    - "画面要轻，不要堆满活动运营感。"
    - "透明和柔软是关键，不要做成硬塑料玩具。"
    - "色彩应舒缓，避免高压霓虹。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出泡泡、清透、软弹、舒缓解压，而不是普通三消或泛休闲糖果背景。"
```

## 11. 伊洛纳

```yaml
style_brief:
  name: "伊洛纳"
  aliases:
    - "Elona"
    - "伊洛纳手游"
  tags:
    - "像素"
    - "开放世界"
    - "高自由度"
    - "角色扮演"
    - "回合制"
    - "冒险"
  source:
    checked: "TapTap 页面：游戏名、角色扮演/开放世界/像素/高自由度/冒险/回合制标签，以及高自由度、高难度、以太风异变世界描述。"
    confidence: high
  game_features:
    type: "像素开放世界 Roguelike RPG"
    core_gameplay: "自由探索、角色养成、地下城冒险、职业种族搭配、生活行为和随机事件"
  visual_identity: "复古像素 RPG 与荒诞高自由度世界结合，视觉不追求华丽，而强调密集信息、奇妙事件和老派冒险质感。"
  genre_presentation: "以像素格子承载复杂开放世界，用粗粝、低分辨率和高信息密度表达自由人生。"
  mood: "奇异、自由、复古、混乱、冒险、略带荒诞；情绪来自像素地图、地下城、怪物、职业符号和随机生活细节。"
  world_elements: "以太风、异变世界、地面城镇、地下城、怪物、职业、种族、钢琴家、种田钓鱼、自由人生。"
  colors: "像素色板倾向中高饱和但受限，常见草地绿、土黄、石灰、暗红、深蓝、木色；对比直接，明暗分块清楚。"
  materials: "低分辨率像素材质；石块、泥土、木板、布料、金属和魔法效果被压缩成色块与少量高光；边缘锯齿是风格的一部分。"
  shape_language: "方格、瓦片、像素边、粗轮廓、小图标、模块化地图块；面板与图标强调老派 RPG 的清晰分类和功能密度。"
  background_visual_language: "背景适合像素城镇、地牢、荒野、异变气候和瓦片地图语言；安全倾向是网格感明确、符号密集但主体区域可控。"
  ui_translation:
    panel: "面板系统偏复古 RPG 菜单、像素边框、深浅色块分区、物品格、状态条和小图标矩阵；大卡像背包/角色面板，小卡像道具格或任务条。"
    panel_base_style_hint: "像素石板、旧纸、暗色木框或低分辨率色块底色，方正边框、阶梯状像素边、硬阴影和粗线分隔；可选局部装饰可联想像素折角、瓦片纹理、低清图案块、粗颗粒纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别道具、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏老派数值化，常见条形血量、像素图标、闪烁边框、颜色替换和小型提示。"
    text: "字体应接近像素或低分辨率 RPG 字体；字重清晰，描边少，强调可读和复古。"
    accent: "语义候选包括像素瓦片、以太异变色、地牢石纹、背包格、复古 RPG 状态条、魔法小光点。"
  forbidden_mistakes: "不要误判成高清二次元 RPG、现代扁平像素、可爱农场像素、赛博像素或纯复古街机。"
  style_archetype: "高自由度复古像素 RPG"
  archetype_shared_traits:
    - "低分辨率像素构成世界"
    - "UI 信息密度高"
    - "地图和道具以格子组织"
    - "冒险感来自复杂系统而非华丽演出"
  specific_differentiators:
    - "以太风和异变世界带有奇诡感"
    - "自由人生玩法让视觉包含战斗和生活杂糅"
    - "像素粗粝感不能被过度精修"
    - "荒诞职业与事件比标准勇者史诗更重要"
  signature_semantics:
    must_include:
      - "像素瓦片质感"
      - "复古 RPG 菜单气质"
      - "自由冒险与混乱生活感"
      - "以太异变世界语义"
    should_include:
      - "地牢和城镇格子"
      - "背包物品格"
      - "职业与种族符号感"
      - "粗颗粒色块"
    should_reduce:
      - "高清渐变"
      - "光滑 3D 材质"
      - "大面积现代玻璃拟态"
      - "过度可爱圆角"
    must_avoid:
      - "精致二次元立绘主导"
      - "纯田园治愈像素"
      - "现代手游卡片化过强"
      - "直接照搬具体怪物、角色或 logo"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别角色、怪物、logo、具体道具或官方图标作为前景主体"
    edge_or_corner_allowed:
      - "像素瓦片边"
      - "粗颗粒石纹"
      - "物品格暗纹"
      - "低分辨率折角"
    far_background_allowed:
      - "像素城镇"
      - "地牢墙面"
      - "荒野格子地图"
      - "异变天空色块"
    center_safe_area_allowed:
      - "低对比像素地面"
      - "旧纸或暗色格纹"
      - "简化瓦片留白"
  anti_generic_guardrails:
    - "像素必须粗粝、有老派 RPG 信息感，不能变成高清伪像素。"
    - "自由和混乱是识别点，不能只做普通地牢。"
    - "UI 应像系统复杂的 RPG，而不是极简休闲。"
    - "异变世界要保留奇诡，不要完全田园化。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出复古像素、高自由度 RPG、以太异变、地牢与生活系统混杂的伊洛纳式自由冒险。"
```

## 12. 王牌竞速

```yaml
style_brief:
  name: "王牌竞速"
  aliases:
    - "Ace Racer"
  tags:
    - "赛车"
    - "竞速"
    - "漂移"
    - "高画质"
    - "多人联机"
    - "写实赛车"
  source:
    checked: "TapTap 页面：游戏名、赛车/高画质/竞速/多人联机/漂移/竞技标签，以及写实风格、授权真车、超现实概念车、王牌大招和实景赛道描述。"
    confidence: high
  game_features:
    type: "写实高画质技能赛车手游"
    core_gameplay: "竞速、漂移、多人对战、赛车收集、车辆专属大招、实景赛道"
  visual_identity: "真实汽车工业质感与超现实技能演出结合，强调豪车、速度节、城市与风景赛道的高光竞速体验。"
  genre_presentation: "不是纯模拟赛车，而是写实外壳下带技能释放和幻想加速的爽感竞速。"
  mood: "高速、炫酷、豪华、热血、都市、年轻化；情绪来自车漆反光、赛道光带、速度线、霓虹和大招特效。"
  world_elements: "授权真车、超现实概念车、速度节、西湖、天门山、洪崖洞、全球风光、漂移、大招、豪华超跑。"
  colors: "常见深色车体与高亮霓虹对比，蓝紫、橙红、金属银、赛道白线、城市夜景光；高对比、高亮点，强调速度和高级感。"
  materials: "写实 PBR 质感明显；车漆、碳纤维、玻璃、镀铬、湿地反光、柏油路、霓虹光带；高光锐利，反射丰富。"
  shape_language: "流线型、低趴、锐角、空气动力学曲线、仪表盘弧线、速度箭头、赛道切线和科技卡片。"
  background_visual_language: "背景适合城市夜景、山路、景区赛道、湿地反光和速度模糊；安全倾向是强透视和光带引导，但避免遮挡信息。"
  ui_translation:
    panel: "面板偏现代车机和赛事转播语言，大卡像赛车展台或赛事卡，小卡像车辆参数卡，标签偏锐角、斜切和发光条，状态层强调速度、稀有度、技能充能和赛事奖励。"
    panel_base_style_hint: "深色磨砂金属、碳纤维、玻璃或车漆反光底色，斜切边框、锐利细线、冷色辉光和高速投影；可选局部装饰可联想赛道切线、碳纤维纹理、光带折角、仪表弧线片段，但具体角色、logo、徽章、状态标记、文字、可识别车辆、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏赛事化和能量化，常见充能条、速度光带、氮气感、排名闪光、技能释放特效。"
    text: "字体偏现代无衬线、斜体、粗窄、速度感；标题可金属渐变和发光，数字要像仪表读数。"
    accent: "语义候选包括速度光带、碳纤维、车漆高光、赛道箭头、仪表盘、霓虹城市、赛事编号。"
  forbidden_mistakes: "不要误判成纯写实模拟赛车、卡通漂移、复古街机赛车、赛博朋克都市或普通豪车广告。"
  style_archetype: "写实幻想技能赛车"
  archetype_shared_traits:
    - "真实车体材质"
    - "速度光效强化反馈"
    - "现代赛事 UI"
    - "赛道透视和动感模糊"
  specific_differentiators:
    - "真车与超现实概念车并置"
    - "车辆大招是核心视觉差异"
    - "国内外实景赛道带旅行感"
    - "豪车质感与手游爽感并存"
  signature_semantics:
    must_include:
      - "写实车漆和金属质感"
      - "高速光带和漂移动势"
      - "现代赛事科技 UI"
      - "超现实技能释放语义"
    should_include:
      - "霓虹城市或景区赛道"
      - "碳纤维纹理"
      - "仪表盘弧线"
      - "湿地或车身反射"
    should_reduce:
      - "纯卡通轮廓"
      - "低清贴图"
      - "厚重军武 UI"
      - "过度可爱装饰"
    must_avoid:
      - "儿童玩具赛车"
      - "硬核模拟器冷界面"
      - "全赛博无真实车感"
      - "直接照搬授权车辆、logo 或赛事标识"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别授权车辆、车标、官方 logo 或赛事徽章作为前景主体"
    edge_or_corner_allowed:
      - "碳纤维纹理"
      - "斜切光带"
      - "赛道线片段"
      - "仪表弧线"
    far_background_allowed:
      - "城市夜景"
      - "山路赛道"
      - "景区道路"
      - "霓虹灯带和速度模糊"
    center_safe_area_allowed:
      - "暗色磨砂渐变"
      - "低对比路面纹理"
      - "柔化光带透视"
  anti_generic_guardrails:
    - "必须保留写实车体高级感，不能变成卡通赛车。"
    - "必须有技能竞速的能量感，不能只做模拟器。"
    - "速度节和赛事感要强于普通汽车广告。"
    - "实景赛道语义可出现，但不要喧宾夺主。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出写实豪车、技能大招、速度节赛事、霓虹光带和高画质漂移竞速。"
```

## 13. 绝尘漂移

```yaml
style_brief:
  name: "绝尘漂移"
  aliases:
    - "绝尘漂移"
  tags:
    - "赛车"
    - "漂移"
    - "竖屏"
    - "快节奏"
    - "多人联机"
    - "休闲竞技"
  source:
    checked: "TapTap 页面：游戏名、赛车/竞速/漂移/DIY/多人联机/竖屏/休闲标签，以及真实玩家对战、超过 60 辆车型、多天气、近道、加速带、首款竖屏快节奏赛车对战描述。"
    confidence: high
  game_features:
    type: "竖屏快节奏漂移赛车对战游戏"
    core_gameplay: "短局漂移、多人对战、车辆收集升级、单人和组队挑战、天气与近道赛道"
  visual_identity: "轻量化赛车竞技风，强调竖屏、快节奏、漂移烟尘和车辆收集，介于写实赛车和休闲对战之间。"
  genre_presentation: "把赛车对战压缩成移动端短局体验，用清晰赛道、漂移反馈和升级收集支撑循环。"
  mood: "爽快、紧凑、竞技、轻休闲、尘土飞扬；情绪来自漂移烟雾、加速带、赛道弯道和快速排名变化。"
  world_elements: "肌肉车、改装车、越野车、超跑、多天气赛道、近道、加速带、联赛、滚滚浓烟。"
  colors: "倾向鲜明运动色和赛道环境色；车体色可高饱和，背景更自然；强调色常用于加速、排名、警示和奖励。"
  materials: "半写实移动端赛车材质；车漆、轮胎、柏油、尘烟、雨雪天气、泥地和赛道道具；反光适中，整体比高端写实赛车更轻。"
  shape_language: "竖向构图、赛道 S 弯、箭头、速度线、轮胎痕、烟尘云、斜切按钮和车辆卡片。"
  background_visual_language: "背景适合竖屏赛道纵深、天气层次、弯道、近道和加速带；安全倾向是路径清晰、速度感强、信息不复杂。"
  ui_translation:
    panel: "面板偏移动端竞技卡片，车辆卡、升级卡、联赛卡和挑战卡清晰分层；标签多用斜切、箭头、速度条，状态层强调排名、升级、车辆稀有度和漂移反馈。"
    panel_base_style_hint: "运动塑料、轻金属、柏油纹理或车漆感底色，斜切轮廓、速度线边框、轻量阴影和清晰高光；可选局部装饰可联想轮胎痕、烟尘弧线、赛道箭头、加速光带片段，但具体角色、logo、徽章、状态标记、文字、可识别车辆、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈强调短局爽感，常见漂移连击、加速亮光、排名跳变、车辆升级闪光和烟尘爆发。"
    text: "字体偏粗、运动、易读，适合竖屏小空间；数字和排名可更强烈，标题可斜体或带速度感。"
    accent: "语义候选包括轮胎痕、漂移烟尘、加速带、赛道箭头、车漆高光、天气粒子。"
  forbidden_mistakes: "不要误判成王牌竞速式高端写实豪车、QQ 飞车式强卡通潮流、极限竞速模拟器或儿童玩具赛车。"
  style_archetype: "竖屏轻量漂移竞技"
  archetype_shared_traits:
    - "短局快节奏"
    - "漂移动作是主要视觉反馈"
    - "车辆收集升级清晰"
    - "UI 更轻量移动端化"
  specific_differentiators:
    - "竖屏构图是明显差异"
    - "烟尘和轮胎痕比豪华光效更关键"
    - "休闲竞技和真实玩家对战并重"
    - "车型多样但不以授权豪车奢华为核心"
  signature_semantics:
    must_include:
      - "竖屏赛道纵深"
      - "漂移烟尘和轮胎痕"
      - "轻量竞技 UI"
      - "车辆收集升级感"
    should_include:
      - "加速带箭头"
      - "多天气赛道"
      - "短局排名反馈"
      - "运动色块"
    should_reduce:
      - "过度豪车广告感"
      - "复杂霓虹科技"
      - "严肃模拟器仪表"
      - "低龄玩具感"
    must_avoid:
      - "纯写实高端超跑大片"
      - "强二次元赛车娘风"
      - "横屏主机赛车构图"
      - "直接照搬具体车辆、logo 或车牌"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别车辆、车牌、官方 logo 或联赛徽章作为前景主体"
    edge_or_corner_allowed:
      - "轮胎痕"
      - "烟尘弧线"
      - "赛道箭头"
      - "轻金属折角"
    far_background_allowed:
      - "竖向赛道"
      - "弯道远景"
      - "天气粒子"
      - "近道和加速带轮廓"
    center_safe_area_allowed:
      - "低对比柏油纹理"
      - "柔化赛道透视"
      - "淡烟尘留白"
  anti_generic_guardrails:
    - "必须强调竖屏短局，不要做成横屏大片。"
    - "漂移烟尘是识别重点，不能只放速度光。"
    - "休闲竞技要轻快，不能过度硬核模拟。"
    - "车辆视觉可多样，但不要依赖具体车标。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出竖屏、短局、漂移烟尘、车辆收集和轻量多人竞速，而非泛赛车背景。"
```

## 14. 无限大

```yaml
style_brief:
  name: "无限大"
  aliases:
    - "Project Mugen"
  tags:
    - "开放世界"
    - "都市"
    - "二次元"
    - "角色扮演"
    - "动作"
    - "高自由度"
  source:
    checked: "TapTap 页面：游戏名、开放世界/角色扮演/高自由度/都市/二次元/动作/高画质/3D 标签，以及都市开放世界、商业中心、街巷角落、异常威胁、霓虹灯下烟火气、社交媒体和对策局描述。"
    confidence: high
  game_features:
    type: "都市题材二次元开放世界 RPG"
    core_gameplay: "城市探索、角色切换、动作冒险、伙伴团队、势力任务、都市日常与异常事件"
  visual_identity: "高画质二次元都市开放世界，日常烟火气与异常危机并存，强调年轻、潮流、社交媒体和城市立体穿梭。"
  genre_presentation: "把开放世界从自然幻想转向现代都市，用商业街、巷道、霓虹和奇人异事承载冒险。"
  mood: "鲜活、潮流、自由、奇异、都市日常、轻冒险；情绪来自霓虹、街头招牌、社交界面、角色时尚和异常事件反差。"
  world_elements: "新启、商业中心、隐秘街巷、对策局、巡卫署、犬魔社、蓝猫物流、社交媒体、网红、异常威胁、伙伴团队。"
  colors: "倾向现代都市高彩度，白天清亮、夜晚霓虹；常见青蓝、品红、橙黄、黑白灰城市基底；冷暖对比活跃，强调潮流和生活感。"
  materials: "二次元 3D 渲染与现代城市材质；玻璃幕墙、柏油、霓虹灯箱、金属栏杆、塑料招牌、电子屏和服装布料；质感干净但不纯写实。"
  shape_language: "现代都市方块、广告牌、手机界面、弹窗、斜切贴纸、潮流图形、漫画式速度线和轻科技卡片。"
  background_visual_language: "背景适合繁华商圈、天台、街巷、地铁口、霓虹夜景和生活化小店；安全倾向是都市密度高但留出清晰信息区。"
  ui_translation:
    panel: "面板系统偏手机社交、都市服务 App 和轻科技终端混合；大卡像城市任务板或社交动态，小卡像角色身份卡、地点卡、任务贴片，状态层强调声望、势力、异常、社交热度和伙伴关系。"
    panel_base_style_hint: "清爽玻璃、浅色电子屏、磨砂塑料或霓虹反光底色，现代圆角矩形、轻斜切、细线框、干净投影和局部发光；可选局部装饰可联想广告牌折角、社交弹窗边、霓虹线条、街头贴纸纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别道具、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏都市 App 化和潮流化，常见弹窗、点赞热度、任务标记、异常警报、地图定位和霓虹高亮。"
    text: "字体偏现代无衬线、年轻、干净；标题可有潮流字形和局部霓虹，正文像手机 UI 信息。"
    accent: "语义候选包括霓虹灯、社交媒体弹窗、城市定位、街头贴纸、电子屏、异常警报、势力任务。"
  forbidden_mistakes: "不要误判成传统幻想开放世界、赛博朋克重工业、纯校园二次元、硬科幻都市或写实 GTA 风。"
  style_archetype: "二次元现代都市开放世界"
  archetype_shared_traits:
    - "现代城市空间"
    - "角色时尚和伙伴团队"
    - "霓虹与日常烟火并存"
    - "手机化 UI 信息"
  specific_differentiators:
    - "社交媒体和网红拯救世界是强识别"
    - "日常生活与异常威胁交替"
    - "城市势力带有轻怪诞幽默"
    - "不是末世都市，而是鲜活繁华都市"
  signature_semantics:
    must_include:
      - "现代都市开放世界"
      - "二次元高画质角色语义"
      - "霓虹与烟火气并存"
      - "社交媒体和城市任务感"
    should_include:
      - "商业中心和街巷"
      - "手机弹窗式 UI"
      - "潮流贴纸"
      - "异常警报反差"
    should_reduce:
      - "古典幻想纹样"
      - "过重赛博脏污"
      - "军事硬核 HUD"
      - "纯校园粉色调"
    must_avoid:
      - "传统魔法大陆"
      - "废土末世城市"
      - "写实犯罪都市"
      - "直接照搬角色、势力 logo 或具体招牌"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别角色、势力标志、官方 logo、具体招牌或 IP 道具作为前景主体"
    edge_or_corner_allowed:
      - "霓虹线条"
      - "街头贴纸"
      - "电子屏折角"
      - "社交弹窗框"
    far_background_allowed:
      - "商业街"
      - "天台城市线"
      - "隐秘巷道"
      - "霓虹夜景和生活小店"
    center_safe_area_allowed:
      - "浅色玻璃渐变"
      - "虚化城市光斑"
      - "低对比电子屏纹理"
  anti_generic_guardrails:
    - "必须有现代都市生活感，不要只做二次元角色背景。"
    - "霓虹不能过度赛博废土，要保留烟火气。"
    - "UI 应像城市 App 与任务终端混合，不是军工 HUD。"
    - "异常威胁是反差点，不应吞没日常气质。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出二次元都市开放世界、霓虹烟火、社交媒体、异常危机和潮流城市任务感。"
```

## 15. 代号：新生

```yaml
style_brief:
  name: "代号：新生"
  aliases:
    - "代号新生"
  tags:
    - "俯视角射击"
    - "搜打撤"
    - "多人联机"
    - "末世"
    - "小动物特工"
    - "3D"
  source:
    checked: "TapTap 页面：游戏名、射击/俯视角/搜打撤/多人联机/3D/动作标签，以及小动物特工、风雪末世、价值物资撤离、末日庇护所、末班车和冰原描述。"
    confidence: high
  game_features:
    type: "俯视角动物特工搜打撤射击游戏"
    core_gameplay: "组队探索、俯视角战斗、搜集物资、撤离、庇护所打造、末班车基地推进"
  visual_identity: "冰雪末世与可爱小动物特工形成反差，既有搜打撤的谨慎和危险，也有毛茸茸角色与移动基地的温度。"
  genre_presentation: "用俯视角和动物拟人降低硬核门槛，把撤离射击转译成寒冷世界里的小队生存冒险。"
  mood: "寒冷、谨慎、可爱、坚韧、孤独中带温暖；情绪来自风雪、冰原、钢铁列车、火种、物资和小动物特工。"
  world_elements: "末班车、无尽冰原、企业、军团、火种、移动钢铁基地、小动物特工、风雪末世、物资撤离、末日庇护所。"
  colors: "整体偏冷，常见冰蓝、雪白、灰钢、暗青和低饱和棕；火种、警示灯和物资奖励可用暖橙形成生存希望感。"
  materials: "半写实卡通 3D；雪、冰、旧钢铁、车厢、布料、背包、毛绒角色、木箱和煤火；材质带寒冷磨损，但不应过度写实残酷。"
  shape_language: "俯视角地图块、圆润动物轮廓、战术装备小件、车厢模块、雪地脚印、箱体、轨道线和庇护所构件。"
  background_visual_language: "背景适合冰原、风雪、列车基地、废弃设施、补给点和低能见度空间；安全倾向是冷色大气、边缘有生存设施线索，中心保持清晰。"
  ui_translation:
    panel: "面板系统可结合移动基地终端、补给箱、车票、电台和寒地战术卡；大卡像车厢任务板或物资清单，小卡像补给格、动物特工状态卡，状态层强调撤离、温度、物资价值、风险和庇护所成长。"
    panel_base_style_hint: "冷色旧钢、磨砂玻璃、雪覆金属或厚布料底色，圆角箱体边框、轻战术线、寒冷阴影和微弱暖光；可选局部装饰可联想雪痕折角、车厢铆钉纹理、冰晶线条、补给箱贴片片段，但具体角色、logo、徽章、状态标记、文字、可识别道具、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈兼具搜打撤风险和温暖庇护所感，常见撤离提示、物资价值、寒冷警示、火种暖光、队友状态和补给完成反馈。"
    text: "字体偏清晰无衬线，可带电台/车票/末世档案感；标题可略粗，避免过分可爱或过分军工。"
    accent: "语义候选包括火种暖光、雪痕、车厢金属、补给箱、冰晶、撤离箭头、电台波形、小动物特工身份感。"
  forbidden_mistakes: "不要误判成三角洲式硬核军事、明日之后式写实废土、纯动物治愈经营、冰雪童话或卡通塔防。"
  style_archetype: "冰雪末世动物特工搜打撤"
  archetype_shared_traits:
    - "冷色末世环境"
    - "物资撤离风险"
    - "小队生存和基地成长"
    - "可爱角色缓和残酷题材"
  specific_differentiators:
    - "小动物特工是核心反差"
    - "末班车移动基地提供温暖识别"
    - "冰原而非沙漠或城市废土"
    - "俯视角让战术更像小队棋盘"
  signature_semantics:
    must_include:
      - "冰雪末世冷色氛围"
      - "小动物特工反差"
      - "末班车移动基地语义"
      - "搜打撤物资和撤离风险"
    should_include:
      - "火种暖光"
      - "补给箱和物资清单"
      - "风雪粒子"
      - "车厢或轨道线索"
    should_reduce:
      - "纯硬核军武"
      - "过度血腥废土"
      - "糖果动物乐园"
      - "高饱和热带色"
    must_avoid:
      - "三角洲式现代特战写实"
      - "纯冰雪童话"
      - "丧尸恐怖血污"
      - "直接照搬动物角色、末班车 logo 或具体物资图标"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别小动物特工、官方 logo、具体物资、车厢标识或阵营标志作为前景主体"
    edge_or_corner_allowed:
      - "雪痕"
      - "冰晶线条"
      - "旧钢铆钉"
      - "补给箱贴片纹理"
    far_background_allowed:
      - "冰原"
      - "风雪废弃设施"
      - "远处列车基地"
      - "暖光车厢剪影"
    center_safe_area_allowed:
      - "冷色雪雾"
      - "低对比冰面纹理"
      - "微弱暖光渐变"
  anti_generic_guardrails:
    - "必须同时有寒冷末世和可爱动物特工，缺一会变成泛废土或泛萌宠。"
    - "搜打撤风险要存在，但不要变成硬核军事压迫。"
    - "末班车和火种提供温度，不能全画成绝望冰原。"
    - "俯视角信息感应比 FPS 枪械特写更重要。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出冰雪末世、动物特工、末班车火种、俯视角搜打撤和温暖庇护所反差。"
```

## 16. 怪物猎人：旅人

```yaml
style_brief:
  name: "怪物猎人：旅人"
  aliases:
    - "怪猎旅人"
    - "Monster Hunter Outlanders"
    - "怪物猎人旅人"
  tags:
    - "ARPG"
    - "动作"
    - "冒险"
    - "多人联机"
    - "共斗"
    - "开放世界狩猎"
    - "怪物猎人 IP"
  source:
    checked: "TapTap 页面：游戏名、ARPG/冒险/多人联机/社交/共斗/动作标签，以及“卡普空正版授权、腾讯天美研发、保留经典武器和狩猎体验、全新世界观、冒险家、随从和独占新怪物”描述；补充参考官网关于埃索岛、随从、融光种火龙等公开设定。"
    confidence: high
  game_features:
    type: "开放世界共斗狩猎动作手游"
    core_gameplay: "探索生态区域、追踪大型怪物、使用经典武器狩猎、多人共斗、采集素材、随从协作、营地与移动端探索"
  visual_identity: "以怪物猎人系列的原始生态、巨兽狩猎、工艺装备和冒险营地为基础，加入移动端开放世界的明亮探索感与新大陆旅行感。"
  genre_presentation: "不是纯幻想 RPG，也不是写实生存游戏，而是以生态可信度、怪物压迫感、猎人工艺和团队狩猎仪式构成的动作冒险。"
  mood: "壮阔、野性、探索、紧张、工匠感、共斗感；情绪来自巨大怪物、自然地貌、骨木皮革装备、营地火光、追踪痕迹和战斗后的素材收获。"
  world_elements: "埃索岛、冒险家、经典武器、随从艾露猫、原生兽人、特有鸟类随从、独占新怪物、融光化怪物、火龙、森林、火山、荒野、营地、素材、狩猎工具。"
  colors: "整体偏自然生态色，常见森林绿、泥土棕、岩石灰、兽皮黄、火焰橙、天空蓝和营地暖光；饱和度中等，强调自然环境的真实层次与怪物战斗时的高亮危险色。"
  materials: "半写实偏写实的狩猎工艺材质；皮革、骨甲、兽鳞、木材、粗布、岩石、泥土、金属刃、火焰、熔岩与生态植被；纹理丰富，边缘有磨损、绑带、缝线和手作痕迹。"
  shape_language: "轮廓偏原始、厚重、手工、不规则；常见骨刺、兽牙、鳞片、爪痕、绳结、木框、皮革包边、营地牌、地图卷边和狩猎工具轮廓。"
  background_visual_language: "背景以自然生态区域和狩猎旅途为核心，适合森林、火山、岩壁、营地、草原、怪物栖息地、远景山体与生态痕迹；安全倾向是保留自然纵深和冒险氛围，避免把背景做成纯 UI 纹样或泛魔幻场景。"
  ui_translation:
    panel: "面板系统应带有猎人营地、任务板、生态笔记和装备工坊语言；大卡像任务委托板、地图档案或装备展示板，小卡像素材格、武器状态卡、随从信息卡，标签可联想皮革签条、木牌、卷边纸片和金属铆钉，状态层强调怪物状态、素材品质、武器锐度、队伍协作、探索进度和狩猎奖励。"
    panel_base_style_hint: "旧皮革、粗木、兽骨、磨损金属、旧地图纸或营地布料底色，不规则厚边、手作包边、铆钉缝线、爪痕磨损和暖火阴影；可选局部装饰可联想兽骨折角、皮革贴片、绳结线条、鳞片纹理、木牌纹理片段，但具体角色、logo、徽章、状态标记、文字、可识别武器、具体怪物、具体世界观物件不作为基础主体或局部装饰来源。"
    state: "状态反馈偏狩猎信息化和素材奖励感，常见怪物警戒、部位破坏、武器锋利度、任务完成、采集成功、队伍支援、火焰危险和生态事件提示。"
    text: "字体气质偏厚重、冒险、工艺感；标题可带木牌、皮革、金属或旧地图质感，正文保持清晰，避免过分现代科技或可爱圆体。"
    accent: "语义候选包括爪痕、兽骨、鳞片、皮革缝线、营地火光、任务板、旧地图、素材品质、生态痕迹、融光火焰和狩猎警示。"
  forbidden_mistakes: "不要误判成传统魔法二次元 RPG、写实末日生存、硬核军事狩猎、纯恐龙史前题材、Q版宠物冒险或泛开放世界奇幻。最容易误判为原神式高饱和幻想、方舟式生存建造、暗黑魔幻巨兽或普通 MMORPG 副本 UI。"
  style_archetype: "生态巨兽共斗狩猎冒险"
  archetype_shared_traits:
    - "自然生态与巨型怪物构成核心压迫"
    - "装备和 UI 有手工制作、素材加工和营地任务感"
    - "战斗反馈围绕怪物状态、部位、武器和素材收益"
    - "环境不是纯背景，而是狩猎路径、生态线索和危险来源"
  specific_differentiators:
    - "怪物猎人 IP 的武器狩猎仪式感强于普通 ARPG 技能循环"
    - "工艺材质以兽骨、皮革、木材、鳞片和旧金属为主，而非华丽魔法宝石"
    - "开放世界探索服务于生态追踪和狩猎准备，不是纯风景漫游"
    - "随从和营地带来旅途温度，但整体不应变成萌宠休闲"
    - "融光化与独占新怪物可提供新的火焰、熔融和生态异变识别"
  signature_semantics:
    must_include:
      - "生态巨兽狩猎感"
      - "猎人工艺材质"
      - "营地任务与冒险旅途语义"
      - "多人共斗和经典武器狩猎氛围"
      - "自然地貌中的危险与探索"
    should_include:
      - "兽骨、鳞片、皮革、木材、绳结等素材加工感"
      - "旧地图、任务板、营地火光"
      - "怪物痕迹、爪痕、部位破坏和素材奖励语义"
      - "随从协作和旅人冒险气质"
      - "火山、森林、岩壁、荒野等生态区域线索"
    should_reduce:
      - "过度二次元角色立绘中心化"
      - "纯科技 HUD"
      - "高饱和糖果色"
      - "过分平滑的现代玻璃拟态"
      - "普通 MMORPG 宝石金边堆叠"
    must_avoid:
      - "直接照搬火龙、艾露猫、官方 logo、经典武器外形或怪物造型"
      - "做成原神式清新幻想 UI"
      - "做成方舟式生存科技界面"
      - "做成暗黑魔幻血腥巨兽"
      - "做成卡通萌宠探险"
  background_identity_anchors:
    foreground_forbidden:
      - "不可直接放置可识别怪物、猎人角色、艾露猫、官方 logo、经典武器、任务图标或具体随从作为前景主体"
      - "不可用大型可识别 IP 怪物剪影抢占中心"
    edge_or_corner_allowed:
      - "抽象爪痕"
      - "皮革缝线"
      - "兽骨折角"
      - "木牌纹理"
      - "鳞片纹理片段"
      - "旧地图卷边"
      - "营地布料边"
    far_background_allowed:
      - "森林栖息地"
      - "火山岩壁"
      - "荒野山体"
      - "远处营地火光"
      - "生态痕迹和大型巢穴轮廓"
      - "融光或火焰异变的远景色带"
    center_safe_area_allowed:
      - "旧地图纸面"
      - "低对比皮革或木纹"
      - "柔和自然雾气"
      - "暗化营地布料"
      - "不具可识别 IP 指向的生态纹理"
  anti_generic_guardrails:
    - "必须同时具备自然生态、巨兽压迫、猎人工艺和共斗狩猎，不能只做普通开放世界。"
    - "材质要有素材加工和手作痕迹，不能变成现代科技金属卡片。"
    - "怪物语义应通过爪痕、鳞片、骨质和生态痕迹表达，避免直接复制具体怪物。"
    - "冒险感应偏狩猎营地和生态调查，不是华丽魔法学院或二次元都市。"
    - "随从语义可以增加温度，但不能把整体拉成可爱宠物养成。"
    - "火焰和融光可以作为危险强调，但不能把整套风格误做暗黑熔岩魔幻。"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "仍能看出生态巨兽狩猎、猎人工艺材质、营地任务板、旧地图、兽骨皮革鳞片、共斗冒险与怪物痕迹，而不是泛开放世界、泛魔幻 RPG 或普通生存游戏。"
```
