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

旧条目补全是 02 的职责。补全后必须把完整 `style_brief` 作为 04 / 05 的输入；05 只能消费补全结果，不能在生成执行阶段临时补写 Style Specificity Contract、`panel_base_style_hint`、`ui_translation.state` 或 `ui_translation.text`。

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
  style_archetype: "低多边形治愈系慢生活小镇模拟"
  archetype_shared_traits:
    - "明亮自然光、柔和色彩、圆润建筑、轻松生活感"
    - "钓鱼、烹饪、园艺、养猫、社交等低压力活动语义"
    - "画面节奏慢，强调空气感、留白和日常幸福感"
  specific_differentiators:
    - "不是纯田园农场，而是带社交广场、海边、家园装扮的小镇生活"
    - "低多边形 3D 感明显，建筑和自然物体有玩具模型般的简化体块"
    - "氛围更像夏日小镇度假，而不是牧场物语式农耕村落"
  signature_semantics:
    must_include:
      - "海边小镇、木质小屋、草坡、阳光、生活活动道具"
      - "低多边形圆润体块和清新柔色"
    should_include:
      - "钓竿、猫、野餐、邮箱、小摊、花园、木栅栏"
    should_reduce:
      - "过度梦幻光效、过多花海、过饱和童话糖果色"
    must_avoid:
      - "写实欧美农场、像素牧场、赛博都市、强竞技 HUD"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大的角色立绘"
      - "复杂战斗武器或怪物"
    edge_or_corner_allowed:
      - "钓竿、猫窝、花盆、邮箱、木牌、小镇路标"
    far_background_allowed:
      - "海岸线、灯塔、坡地小屋、云朵、远处小镇建筑"
    center_safe_area_allowed:
      - "柔和草地、浅色木地板、低对比云影"
  anti_generic_guardrails:
    - "不能只做成普通治愈小镇，必须有低多边形体块感和海边慢生活语义"
    - "生活道具要偏轻社交和休闲，不要变成农场经营工具堆叠"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到低多边形海边小镇、慢生活道具和柔和度假感，能判断是心动小镇，而不是普通农场或动物森友会仿风。"
```

## 02. 乱涂彩世界

```yaml
style_brief:
  name: "乱涂彩世界"
  aliases: []
  tags: ["graffiti", "paint", "anime", "casual-shooter"]
  source:
    checked: "https://www.taptap.cn/app/748842"
    confidence: "high"
  game_features:
    type: "休闲射击 / 二次元 / 涂鸦染色"
    core_gameplay: "以染色、弹幕射击、神明娘伙伴与无厘头漫画剧情驱动的轻爽闯关"
  visual_identity: "潮酷涂鸦系二次元爆涂休闲射击"
  genre_presentation: "把射击视觉化为颜料喷溅、颜色反应、漫画梗和黑白世界被染色的反差。"
  mood: "高能、搞怪、解压、彩色爆发"
  world_elements: "颜料枪、喷漆罐、涂鸦、黑白世界、神明娘、漫画分格、外星入侵、爆色进度条"
  colors: "黑白灰作底层世界；青/粉/黄/紫作染色强调；白色面板保证可读；红色仅作危险或火系反应"
  materials: "高饱和油漆、贴纸胶带、漫画纸、塑料喷罐、亮面颜料层"
  shape_language: "不规则喷溅边、漫画爆炸框、粗描边、倾斜标签、圆角卡片"
  background_visual_language: "灰白城市或漫画纸底，边缘被彩色颜料侵蚀，中央保持浅色干净区域"
  ui_translation:
    panel: "白底漫画卡+粗黑边+少量颜料飞溅，主按钮可用彩色胶囊但字区必须纯净"
    panel_base_style_hint: "白底漫画纸卡或亮面颜料薄层卡底，粗黑描边、不规则喷溅边和倾斜贴纸感；可联想胶带角、颜料滴边、漫画爆炸框边缘，但神明娘、喷漆罐、文字感叹号和大面积颜料泼洒不作为基础主体或局部装饰来源"
    state: "selected 用彩色描边+喷溅角标；warning 用红黑警示，不可与普通彩色装饰混淆"
    text: "标题可用漫画粗体，正文用黑色无衬线；副文字放在纯色底上"
    accent: "喷漆滴落、胶带、手绘箭头、漫画感叹号、颜料块、神格小徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通赛博霓虹或儿童绘画；不要把所有区域都喷满颜料"
  style_archetype: "高饱和涂鸦射击 + 神明娘恶搞幻想"
  archetype_shared_traits:
    - "大面积彩色颜料、喷涂轨迹、夸张 Q 弹特效"
    - "轻度射击/涂色玩法，强调解压、混乱、爽感"
    - "角色和世界观偏二次元搞怪"
  specific_differentiators:
    - "核心不是普通涂鸦街头潮流，而是神明娘/恶魔打工队式荒诞幻想"
    - "色彩应像世界从失色到被爆涂恢复，而不是单纯彩虹背景"
    - "画面可以很吵，但必须有涂色武器和失色世界被染色的对比"
  signature_semantics:
    must_include:
      - "颜料飞溅、喷涂武器、彩色恢复区域、神魔/打工恶魔的搞怪语义"
    should_include:
      - "涂鸦箭头、颜料桶、彩色弹痕、失色建筑局部被染色"
    should_reduce:
      - "全屏无序彩虹、过多荧光贴纸、纯街舞涂鸦感"
    must_avoid:
      - "Splatoon 式鱿鱼海洋语义"
      - "赛博霓虹街头帮派"
      - "儿童填色书低龄感"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大颜料爆炸遮住主体"
      - "高度相似的章鱼/鱿鱼元素"
    edge_or_corner_allowed:
      - "颜料桶、喷枪、神魔小徽章、彩色脚印"
    far_background_allowed:
      - "半失色城市、被染色的街区、异世界传送门"
    center_safe_area_allowed:
      - "低透明颜料渐变、轻微喷溅纹理"
  anti_generic_guardrails:
    - "必须有失色世界被神明娘乱涂拯救的语义，不要只做成彩色涂鸦背景"
    - "色彩可以多，但要有冷灰失色底与彩色覆盖的关系"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "画面同时出现涂色武器、神魔恶搞符号和失色/彩色对比，而不是普通彩虹涂鸦。"
```

## 03. 潜水员戴夫

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
  style_archetype: "像素海洋冒险 + 夜间寿司店经营"
  archetype_shared_traits:
    - "像素艺术、海底探索、鱼类资源收集、轻经营循环"
    - "白天冒险、夜晚经营的双场景结构"
    - "幽默角色和轻松但有神秘感的剧情氛围"
  specific_differentiators:
    - "蓝洞海域与寿司店必须同时构成识别，不只是潜水游戏"
    - "像素不是复古低清，而是高细节像素光影与丰富海洋生态"
    - "荒诞喜剧感强，海洋探索中带一点危险和贪心"
  signature_semantics:
    must_include:
      - "深蓝蓝洞、潜水装备、鱼群、寿司店灯笼/吧台语义"
    should_include:
      - "氧气瓶、鱼叉、金枪鱼、海草、珊瑚、霓虹寿司招牌"
    should_reduce:
      - "过度写实海底、纯治愈水族馆感、日式料理元素堆满全屏"
    must_avoid:
      - "恐怖深海"
      - "儿童海洋乐园"
      - "现代写实潜水模拟器"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大写实鲨鱼压迫主视觉"
      - "纯寿司摆盘大特写"
    edge_or_corner_allowed:
      - "氧气表、鱼叉、寿司菜单牌、小灯笼"
    far_background_allowed:
      - "蓝洞层级、鱼群剪影、珊瑚、沉船遗迹"
    center_safe_area_allowed:
      - "深蓝水体渐变、少量气泡、低对比鱼影"
  anti_generic_guardrails:
    - "不能只画海底，必须暗示白天潜水、晚上寿司店的双循环"
    - "像素颗粒与海底光束要同时存在，避免变成普通海洋插画"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "蓝洞像素海底与寿司店经营符号同时成立，用户能联想到戴夫式海洋冒险经营。"
```

## 04. 鹅鸭杀

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
  style_archetype: "卡通鸟类社交推理派对"
  archetype_shared_traits:
    - "多人社交推理、阵营对抗、任务房间、会议投票"
    - "卡通化动物角色，表情夸张，死亡与伪装以轻喜剧方式呈现"
    - "地图有空间站、沙漠、节庆主城等主题变化"
  specific_differentiators:
    - "主角必须是鹅/鸭/鸟类阵营，而不是无名太空小人"
    - "喜剧、吵闹、互坑感比悬疑恐怖更重要"
    - "角色职业和阵营身份是识别核心"
  signature_semantics:
    must_include:
      - "鹅鸭鸟类剪影、会议按钮、任务终端、阵营对抗暗示"
    should_include:
      - "脚印、羽毛、投票牌、紧急会议桌、管道/机关"
    should_reduce:
      - "太空站金属占比过高"
      - "阴暗狼人杀悬疑感"
    must_avoid:
      - "Among Us 小人造型误读"
      - "真实农场家禽"
      - "恐怖血腥凶案现场"
  background_identity_anchors:
    foreground_forbidden:
      - "无脸太空人主视觉"
      - "血腥尸体"
    edge_or_corner_allowed:
      - "羽毛、鸭脚印、投票卡、职业小图标"
    far_background_allowed:
      - "飞船走廊、任务房、沙漠地图、节庆主城"
    center_safe_area_allowed:
      - "圆桌阴影、地板网格、低对比任务面板"
  anti_generic_guardrails:
    - "不能只做成太空狼人杀，必须保留鹅鸭鸟类身份和荒诞社交感"
    - "鸟类语义要可爱但带互坑，不要做成农场萌宠"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "一眼能看到鹅鸭阵营、会议投票和任务机关，而不是泛太空狼人杀。"
```

## 05. 以闪亮之名

```yaml
style_brief:
  name: "以闪亮之名"
  aliases: []
  tags: ["fashion", "dress-up", "luxury", "modern"]
  source:
    checked: "https://www.taptap.cn/app/218210"
    confidence: "high"
  game_features:
    type: "女性向 / 换装 / 生活"
    core_gameplay: "高自由度捏脸、服装搭配、家园与社交内容"
  visual_identity: "现代高定时装生活美学"
  genre_presentation: "把换装视觉化为时尚杂志、轻奢摄影棚、珠光玻璃和精致妆造系统。"
  mood: "精致、闪耀、优雅、自由表达"
  world_elements: "高定礼服、化妆镜、衣架、珠宝、镜面、摄影灯、香水瓶"
  colors: "珍珠白作底；浅粉/香槟金作高级感；银灰作信息层；玫瑰金/星光蓝作选中；黑色用于时尚对比"
  materials: "珠光玻璃、半透明亚克力、丝绸纸、金属细边、柔光卡片"
  shape_language: "纤细圆角、轻薄层叠、细线框、留白丰富"
  background_visual_language: "柔焦摄影棚或衣帽间边缘元素，中央保持纯净高亮但不过曝"
  ui_translation:
    panel: "大卡用磨砂玻璃+细金属边，小卡去掉复杂花纹只保留光泽"
    panel_base_style_hint: "珍珠白磨砂玻璃或丝绸纸卡底，纤细圆角、玫瑰金/银灰细边和柔光层次；可联想杂志页角、镜框细线、星光切片感，但礼服、珠宝主体、化妆镜和摄影灯不作为基础主体或局部装饰来源"
    state: "selected 用玫瑰金细描边+星点；warning 用珊瑚红小标签，不污染整体"
    text: "标题可高对比优雅字重，正文现代无衬线，字间距略松"
    accent: "钻石、缎带、衣架、镜框、星光、香水瓶、杂志页角。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通粉色少女或婚礼风，忽略现代时装杂志感"
  style_archetype: "高精度时尚生活换装 + 虚拟秀场"
  archetype_shared_traits:
    - "高精度角色、服饰材质、妆容、拍照、家园建造"
    - "强调女性向审美、精致生活方式和高光时刻"
    - "场景常用秀场、衣帽间、豪宅、花园、镜面材质"
  specific_differentiators:
    - "不是普通换装卡牌，而是超自由捏脸、染色、DIY 服饰和生活家园"
    - "视觉重点在材质高级感：丝绸、珠光、玻璃、金属、皮革"
    - "氛围更像个人高光大片，不是甜美纸娃娃"
  signature_semantics:
    must_include:
      - "高级秀场灯光、服装材质、镜面/玻璃、妆容与衣帽间语义"
    should_include:
      - "化妆镜、香水、服装架、珠宝、相机闪光、豪宅窗景"
    should_reduce:
      - "廉价粉色、过度少女贴纸、低龄公主房"
    must_avoid:
      - "二次元战斗少女"
      - "古风宫廷换装"
      - "纯家装模拟"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大人物半身像抢占背景"
      - "婚纱大面积铺满导致单一主题化"
    edge_or_corner_allowed:
      - "珠宝盒、化妆刷、衣架、香水瓶、聚光灯"
    far_background_allowed:
      - "秀场 T 台、落地窗豪宅、衣帽间、摄影棚"
    center_safe_area_allowed:
      - "柔和灯光渐变、浅色高级地面、低对比布料纹理"
  anti_generic_guardrails:
    - "必须体现可定制时尚生活，不要只做成普通奢华粉色背景"
    - "材质表现比花纹堆叠更重要"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "画面能同时传达高精度时尚、自由定制和生活秀场，而不是泛换装或公主风。"
```

## 06. 植物大战僵尸2

```yaml
style_brief:
  name: "植物大战僵尸2"
  aliases: []
  tags: ["cartoon", "tower-defense", "plants", "zombies"]
  source:
    checked: "https://www.taptap.cn/app/54031"
    confidence: "high"
  game_features:
    type: "塔防 / 休闲 / 卡通"
    core_gameplay: "植物单位放置、僵尸波次、跨时空主题关卡"
  visual_identity: "诙谐卡通植物塔防"
  genre_presentation: "把塔防视觉化为后院棋盘、夸张植物表情、僵尸喜剧和清晰的格子攻防。"
  mood: "幽默、轻松、策略、闹腾"
  world_elements: "草坪格子、向日葵、坚果、豌豆、墓碑、路障、时空旅行"
  colors: "草绿作环境；泥土棕作边框；阳光黄作资源/选中；灰紫作僵尸和危险；红色仅作警告"
  materials: "草地板、木牌、泥土卡、漫画纸、石碑按钮"
  shape_language: "圆润夸张、厚描边、棋盘网格、木牌标签"
  background_visual_language: "浅草坪或时空主题远景，边缘植物/墓碑，中心安全区不放密集格子"
  ui_translation:
    panel: "木牌/泥土面板，按钮像种子包但文字区平整"
    panel_base_style_hint: "泥土棕木牌或浅草地纸卡底，圆润厚描边、木牌边和轻漫画颗粒；可联想种子包纸角、叶片边、泥土小裂纹，但植物角色、僵尸、墓碑主体和草坪格阵不作为基础主体或局部装饰来源"
    state: "selected 用阳光黄光圈；locked 用灰墓碑；warning 用僵尸紫/红警示"
    text: "标题可漫画粗体，正文深棕/黑色"
    accent: "阳光、叶子、种子包、墓碑、路障桶、泥土裂纹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通农场或儿童园艺，必须保留塔防棋盘和僵尸喜剧"
  style_archetype: "夸张卡通植物塔防 + 多时空冒险"
  archetype_shared_traits:
    - "植物与僵尸对抗、横向草坪格子、卡通夸张表情"
    - "塔防阵列、阳光资源、植物技能和僵尸进攻路线"
    - "轻幽默、怪诞但不恐怖"
  specific_differentiators:
    - "2 代核心是多时空主题，不只是后院草坪"
    - "埃及、海盗、西部、未来等时代场景可以成为强识别锚点"
    - "植物角色比僵尸更适合做正向识别"
  signature_semantics:
    must_include:
      - "豌豆射手/向日葵语义、草坪格、僵尸剪影、多时空道具"
    should_include:
      - "戴夫、潘妮、阳光、墓碑、时代传送元素"
    should_reduce:
      - "写实恐怖僵尸"
      - "普通农场蔬菜"
      - "过密植物角色堆叠"
    must_avoid:
      - "末日丧尸写实风"
      - "植物精灵奇幻 RPG"
      - "低龄水果乐园"
  background_identity_anchors:
    foreground_forbidden:
      - "僵尸大脸恐怖特写"
      - "整屏植物角色遮挡 UI"
    edge_or_corner_allowed:
      - "阳光币、豌豆弹、墓碑、时空门碎片、植物卡牌"
    far_background_allowed:
      - "古埃及金字塔、海盗船、西部小镇、未来城市"
    center_safe_area_allowed:
      - "草坪格、泥土地、低对比时代纹理"
  anti_generic_guardrails:
    - "必须有塔防格子和植物/僵尸对抗关系，不能只做成卡通花园"
    - "多时空元素要服务 PVZ2，不要变成大杂烩主题公园"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到草坪格、阳光、植物炮台和时空僵尸语义，能识别为 PVZ2，而不是普通卡通塔防。"
```

## 07. 崩坏：星穹铁道

```yaml
style_brief:
  name: "崩坏：星穹铁道"
  aliases: []
  tags: ["space-fantasy", "turn-based-rpg", "train", "anime"]
  source:
    checked: "https://www.taptap.cn/app/224267"
    confidence: "high"
  game_features:
    type: "回合制 RPG / 太空幻想"
    core_gameplay: "列车穿越星海、多星球冒险、角色回合制战斗"
  visual_identity: "星际列车式华丽太空幻想 RPG"
  genre_presentation: "把回合制 RPG 视觉化为星轨旅行、车票、档案、行星文明和精致科幻魔法混合界面。"
  mood: "史诗、浪漫、神秘、精致"
  world_elements: "星穹列车、车票、星轨、行星、命途符号、档案馆、空间站"
  colors: "深蓝/黑作宇宙底；银白作信息层；金色作高级/命途；青蓝/紫作能量；红色只作危险"
  materials: "深色玻璃 HUD、金属细边、星图纸、车票卡、能量纹理"
  shape_language: "圆角与锐角结合、细线框、轨道弧线、星图节点"
  background_visual_language: "深色星空或列车舷窗远景，中心避免高亮星云和密集粒子"
  ui_translation:
    panel: "深色磨砂玻璃卡+细金边/银边，大卡可加星图线，小卡简洁"
    panel_base_style_hint: "深色磨砂玻璃或车票卡质感，细金/银金属边，圆角与轻切角结合；可联想轨道弧线、星图节点和档案卡边，但星图线、命途徽记、车票图形和发光粒子不作为基础主体或局部装饰来源"
    state: "selected 用金色轨道光；warning 用红橙小警示；disabled 降亮度并保留线框"
    text: "标题可细致高对比，正文用浅灰白，避免小字发光"
    accent: "车票、星轨、星星节点、命途徽记、列车窗、档案夹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用蓝紫科幻手游；要保留列车旅行与档案质感"
  style_archetype: "银河列车科幻奇幻 JRPG"
  archetype_shared_traits:
    - "高精度二次元角色、星际旅行、命运/神性/文明主题"
    - "科幻空间站、异星城市、华丽技能光效"
    - "章节世界差异强，视觉奇观化"
  specific_differentiators:
    - "核心识别是星穹列车与开拓旅途，不是普通太空歌剧"
    - "每个世界有强主题文明，但都被银河旅行框架统一"
    - "科幻与神话命途并存，既有列车、星轨，也有宗教/戏剧/城市文明符号"
  signature_semantics:
    must_include:
      - "星穹列车、星轨、宇宙窗景、命途/星神式抽象符号"
    should_include:
      - "车厢金属、跃迁光带、星图、票据、空间站几何结构"
    should_reduce:
      - "泛银河星空"
      - "过度机甲战斗"
      - "单一赛博霓虹"
    must_avoid:
      - "硬核宇航写实"
      - "星际战争 RTS"
      - "普通校园二游"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大角色立绘"
      - "写实宇航员"
    edge_or_corner_allowed:
      - "车票、星轨线、命途徽记、列车窗框、数据屏"
    far_background_allowed:
      - "星穹列车剪影、空间站、异星城市、星云"
    center_safe_area_allowed:
      - "深色星空渐变、车厢地面、低亮度星轨"
  anti_generic_guardrails:
    - "不能只做成二次元太空，必须有列车旅行和命途神话感"
    - "星空背景要有文明叙事，不要只是漂亮宇宙壁纸"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "画面通过星穹列车、星轨、命途符号和异星文明共同指向星铁式银河开拓。"
```

## 08. 我的休闲时光

```yaml
style_brief:
  name: "我的休闲时光"
  aliases: []
  tags: ["cozy", "home", "decor", "casual"]
  source:
    checked: "https://www.taptap.cn/app/242251"
    confidence: "high"
  game_features:
    type: "装修模拟 / 休闲 / 生活"
    core_gameplay: "DIY 小窝、服装造型、猫、种花、网购、料理钓鱼等佛系生活"
  visual_identity: "柔和手绘宅家装修生活模拟"
  genre_presentation: "把休闲模拟视觉化为温馨小屋、软萌家居、猫咪陪伴和轻手绘生活质感。"
  mood: "佛系、温馨、可爱、慢节奏"
  world_elements: "小窝、猫、花盆、家具、咖啡、网购纸箱、钓鱼、料理"
  colors: "奶油白/浅粉作底；浅木棕作结构；薄荷绿/天蓝作辅助；暖橙作奖励；灰色用于已完成/不可用"
  materials: "米白纸卡、布艺贴片、浅木、软塑料按钮"
  shape_language: "大圆角、软卡片、轻手绘描边、贴纸式小图标"
  background_visual_language: "温馨房间或窗边远景，边缘放家具和绿植，中心简洁"
  ui_translation:
    panel: "布艺/纸质圆角卡，卡片之间保持宽松呼吸感"
    panel_base_style_hint: "米白纸卡或布艺圆角卡底，浅木/布边、柔软手绘描边和低饱和家居感；可联想猫爪压痕、纸箱胶带、花朵贴边，但猫、家具、料理和钓鱼小物不作为基础主体或局部装饰来源"
    state: "selected 用暖橙描边+猫爪；locked 用小门牌；warning 用柔和红棕"
    text: "标题圆润可爱，正文深灰，副文浅棕"
    accent: "猫爪、花朵、纸箱、咖啡杯、沙发、窗帘、小鱼。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成高饱和儿童装修或真实家装软件"
  style_archetype: "手绘竖屏佛系家装生活模拟"
  archetype_shared_traits:
    - "温馨小屋、装修 DIY、养猫、种花、做饭、钓鱼"
    - "柔和手绘感、轻社交、低压力日常"
    - "以室内生活和宅家经济为核心"
  specific_differentiators:
    - "比心动小镇更室内蜗居/装修/网购，不是开放小镇度假"
    - "竖屏手游感强，构图更适合房间剖面和家具陈列"
    - "有真实宅家生活语义：快递、网购、猫、咖啡、房间布置"
  signature_semantics:
    must_include:
      - "温馨房间、家具 DIY、猫、快递/网购、手绘软萌质感"
    should_include:
      - "小沙发、地毯、绿植、咖啡杯、窗帘、料理台"
    should_reduce:
      - "户外大世界"
      - "过度童话城堡"
      - "复杂多人社交广场"
    must_avoid:
      - "高精度时尚豪宅"
      - "低多边形小镇"
      - "农场经营主视觉"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大人物换装展示"
      - "广阔海边小镇"
    edge_or_corner_allowed:
      - "猫爪、快递箱、盆栽、小台灯、抱枕"
    far_background_allowed:
      - "窗外街景、阳台、厨房、咖啡店角落"
    center_safe_area_allowed:
      - "浅色墙面、木地板、地毯纹理、柔和窗光"
  anti_generic_guardrails:
    - "必须体现宅家装修 + 佛系生活，不要变成泛治愈小屋"
    - "道具要有现代生活感，尤其快递、网购、猫和家具搭配"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能看出是竖屏手绘宅家装修生活，而不是开放小镇或普通家居插画。"
```

## 09. 三角洲行动

```yaml
style_brief:
  name: "三角洲行动"
  aliases: []
  tags: ["tactical", "military", "shooter", "hud"]
  source:
    checked: "https://www.taptap.cn/app/330259"
    confidence: "high"
  game_features:
    type: "战术射击 / 搜打撤 / 军事"
    core_gameplay: "现代战术小队、枪械、撤离、兵种协作和高信息密度战斗"
  visual_identity: "现代战术军事 HUD"
  genre_presentation: "把射击视觉化为专业军规终端、装备仓、战术地图、枪械参数和紧张但克制的信息系统。"
  mood: "专业、紧张、硬核、冷静"
  world_elements: "战术地图、军用装备、弹匣、编号、坐标、护甲、警戒条、补给箱"
  colors: "深橄榄/枪灰作底；浅灰绿作信息；琥珀黄作选中/任务；红色只用于受伤/危险；蓝色用于友军/系统"
  materials: "磨砂军用终端、碳纤维、旧金属、橡胶、半透明 HUD"
  shape_language: "硬边、切角、网格线、短横条、编号标签、窄边框"
  background_visual_language: "暗绿灰战术地图或仓库墙面，中央低纹理，边缘放装备箱与地图线"
  ui_translation:
    panel: "深色硬边面板+切角边框+网格底纹，小卡减少材质保留编号"
    panel_base_style_hint: "深橄榄/枪灰磨砂终端卡底，硬边切角、窄金属边和克制网格线；可联想装备箱嵌片、坐标短线、军用编号条，但枪械、弹匣、护甲、警戒三角和战场地图不作为基础主体或局部装饰来源"
    state: "selected 用琥珀黄边框+扫描线；locked 用金属锁/封条；warning 用红色高优先级"
    text: "标题窄体硬朗，正文浅灰；小字不要依赖发光"
    accent: "螺丝、军用编号、坐标线、弹孔、警戒三角、装备插槽。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成赛博霓虹或泛科幻蓝 UI；不要过多发光"
  style_archetype: "现代战术军事射击 HUD"
  archetype_shared_traits:
    - "现代枪械、战术装备、载具、军事基地、烟尘爆破"
    - "冷色工业环境、硬边 UI、数据化战术 HUD"
    - "强调小队协作、中远距离交火、战场压迫感"
  specific_differentiators:
    - "不是纯写实军事海报，而是移动/PC 双端战术射击产品化 HUD"
    - "同时包含搜打撤、载具大战场、剧情战役三类战斗语义"
    - "识别点在三角洲特战小队、现代装备和清晰战术信息层"
  signature_semantics:
    must_include:
      - "现代战术 HUD、枪械配件、战术地图、烟尘、军事载具"
    should_include:
      - "无人机、头盔夜视仪、货箱、撤离点标记、坐标网格"
    should_reduce:
      - "过度血腥"
      - "科幻机甲"
      - "英雄角色大特写"
    must_avoid:
      - "卡通吃鸡"
      - "二战复古军事"
      - "赛博朋克枪战"
  background_identity_anchors:
    foreground_forbidden:
      - "角色持枪正面大特写"
      - "血迹和尸体"
    edge_or_corner_allowed:
      - "坐标、雷达、小队编号、弹匣、战术箱"
    far_background_allowed:
      - "港口、沙漠基地、废墟楼体、直升机、装甲车"
    center_safe_area_allowed:
      - "低对比烟雾、战术地图纹理、暗色金属面"
  anti_generic_guardrails:
    - "必须有战术信息层和现代联合作战感，不能只是枪械军事背景"
    - "画面要克制、硬朗、功能化，避免娱乐化卡通吃鸡语义"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能通过现代战术 HUD、撤离/大战场/载具语义判断为三角洲行动，而不是普通 FPS。"
```

## 10. 明日方舟：终末地

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
  style_archetype: "冷峻工业科幻开拓 RPG"
  archetype_shared_traits:
    - "二次元角色、异星开拓、科技设施、基地建设"
    - "低饱和高级灰、工业模块、几何标识、冷光点缀"
    - "世界观强调组织、矿物/源石、危险生态和开拓任务"
  specific_differentiators:
    - "比普通二游更冷、更硬、更工业化，少甜美和高饱和"
    - "核心是塔卫二开拓、终末地工业、协议源石、基地/生产设施"
    - "自然山水与工业设施并置，而不是纯空间站或纯废土"
  signature_semantics:
    must_include:
      - "工业基地、模块化设备、冷灰色调、协议源石/几何能源核心"
    should_include:
      - "轨道设施、运输管线、警示标识、山地科考站、低饱和植被"
    should_reduce:
      - "艳丽魔法特效"
      - "软萌二游装饰"
      - "赛博霓虹商业街"
    must_avoid:
      - "原神式明快幻想大陆"
      - "末日废土脏乱"
      - "太空歌剧列车感"
  background_identity_anchors:
    foreground_forbidden:
      - "角色立绘主导"
      - "过大魔法阵"
    edge_or_corner_allowed:
      - "工业编号、警示条、能量接口、几何徽记、管线"
    far_background_allowed:
      - "塔卫二山地、基地设施、轨道飞行器、开拓据点"
    center_safe_area_allowed:
      - "高级灰金属地面、淡雾、低对比设施阴影"
  anti_generic_guardrails:
    - "必须是冷灰工业开拓，不是泛二次元科幻"
    - "自然景观要被工业建设组织起来，不能只画漂亮异星风景"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到低饱和工业设施、异星开拓基地和明日方舟式几何标识，能区别于星铁、原神或普通科幻二游。"
```

## 11. 香肠派对

```yaml
style_brief:
  name: "香肠派对"
  aliases: ["Sausage Man"]
  tags: ["battle_royale", "shooter", "cartoon", "party", "multiplayer"]
  source:
    checked: "https://www.taptap.cn/app/58881"
    confidence: "high"
  game_features:
    type: "卡通射击 / 吃鸡 / 多人竞技"
    core_gameplay: "第三人称射击、吃鸡生存、摸金撤离、多人模式与搞怪道具对战"
  visual_identity: "贱萌香肠角色的卡通射击派对"
  genre_presentation: "把传统战术射击和吃鸡玩法包装成轻松、搞怪、夸张的香肠角色竞技，强调欢乐对抗和高辨识度角色动作。"
  mood: "搞怪、热闹、竞技、轻松、贱萌"
  world_elements: "香肠人、肠岛、降落伞、枪械、能量饮料、载具、战斗服、赛博道具、宝箱、摸金物资"
  colors: "高饱和蓝紫/橙黄作主视觉；明亮绿和天空蓝营造派对感；红色用于受击、淘汰、警告；金色用于奖励和稀有物资"
  materials: "软塑料玩具、卡通金属、橡胶贴纸、糖果色装备、发光赛博面板"
  shape_language: "圆润、膨胀、粗描边、贴纸化、玩具化，按钮可有弹性和夸张比例"
  background_visual_language: "明亮战场岛屿或趣味训练场，远景可放空投、载具、彩色建筑和夸张武器剪影，中央保持信息清晰"
  ui_translation:
    panel: "圆角厚卡+玩具贴纸风，核心面板可像派对通行证或装备卡片，小组件可用弹性标签和软塑料徽章"
    panel_base_style_hint: "明亮高饱和塑料卡底，厚描边、圆润外轮廓和轻微膨胀感；可联想装备箱贴纸、派对通行证、玩具包装边框，但香肠角色、枪械图标、赛季文字和头像不作为基础主体或局部装饰来源"
    state: "selected 用黄色高亮贴纸或发光描边；warning 用红色受击/警报标签；disabled 降低饱和度并保留厚轮廓"
    text: "标题可用圆润粗体或游戏感粗黑体；正文保持黑灰高可读；奖励数字可用亮黄描边"
    accent: "香肠贴纸、空投箱、子弹、派对旗、降落伞、能量饮料、赛博小图标。装饰应偏轻松搞怪，不要压过射击竞技信息。"
  forbidden_mistakes: "误判成低龄儿童糖果风；需要保留射击、吃鸡、竞技、道具战的核心语义，不能只做成普通卡通派对"
  style_archetype: "魔性香肠人卡通射击派对"
  archetype_shared_traits:
    - "卡通吃鸡、派对竞技、搞怪武器、轻松社交"
    - "色彩明快，角色造型夸张，战斗语义不血腥"
    - "多模式、多道具、多娱乐玩法"
  specific_differentiators:
    - "核心识别必须是香肠人身体结构，不是普通卡通人"
    - "射击要有搞事和整活感，而不是严肃竞技"
    - "战场可以很丰富，但表情包式魔性是第一优先级"
  signature_semantics:
    must_include:
      - "香肠人、搞怪枪械、派对吃鸡、夸张表情/动作"
    should_include:
      - "飞行巴士、彩色补给箱、滑稽头盔、夸张弹道"
    should_reduce:
      - "真实军事装备"
      - "严肃枪战烟尘"
      - "普通卡通动物"
    must_avoid:
      - "三角洲式战术军事"
      - "PUBG 写实吃鸡"
      - "儿童食品广告感"
  background_identity_anchors:
    foreground_forbidden:
      - "写实士兵"
      - "真实枪械大特写"
    edge_or_corner_allowed:
      - "香肠脚印、搞怪头盔、补给箱、派对彩带"
    far_background_allowed:
      - "彩色战场、卡通建筑、跳伞航线、派对地图"
    center_safe_area_allowed:
      - "明快草地、卡通地面、淡色云影"
  anti_generic_guardrails:
    - "不能只做成卡通吃鸡，香肠人轮廓和魔性整活必须强"
    - "枪战信息要轻松搞笑，不要军事化"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "即使没有角色正脸，也能通过香肠轮廓、搞怪装备和派对吃鸡氛围识别。"
```

## 12. 解压泡泡（创意工坊）

```yaml
style_brief:
  name: "解压泡泡"
  aliases: ["解压泡泡测试"]
  tags: ["casual", "relaxing", "clicker", "creative_workshop", "minimal"]
  source:
    checked: "https://www.taptap.cn/app/776653?os=android"
    confidence: "medium"
  game_features:
    type: "休闲解压 / 单机小游戏 / 创意工坊"
    core_gameplay: "通过点击、戳破或清理泡泡获得即时反馈，强调轻松、简单、无压力的解压体验"
  visual_identity: "软萌泡泡与轻量点击反馈的休闲解压"
  genre_presentation: "把压力释放转译为可触碰、可破裂、可连锁反馈的泡泡视觉，用低门槛和清爽界面降低操作负担。"
  mood: "轻松、治愈、柔软、干净、即时满足"
  world_elements: "泡泡、气泡膜、软胶、圆点、涟漪、星星、小闪光、进度条、轻量奖励"
  colors: "浅蓝/薄荷绿作主底；粉紫和淡黄作温和点缀；白色高光表现透明泡泡；避免大面积强红黑"
  materials: "透明凝胶、气泡膜、软玻璃、果冻、轻塑料、磨砂半透明卡片"
  shape_language: "圆形、椭圆、胶囊按钮、柔和圆角、轻微浮起和回弹"
  background_visual_language: "干净浅色渐变或轻微泡泡纹理背景，边缘可漂浮半透明气泡，中央留白用于点击区域和状态信息"
  ui_translation:
    panel: "半透明圆角磨砂卡+果冻高光，小卡片像轻飘泡泡标签，按钮可带按压回弹感"
    panel_base_style_hint: "浅色半透明果冻卡底，柔软圆角、微弱内高光和泡泡膜纹理；可联想透明泡泡、软胶、气泡膜，但具体泡泡棋盘、关卡文字、创作者头像和 TapTap 标识不作为基础主体或局部装饰来源"
    state: "selected 用柔和蓝绿发光；warning 尽量用橙黄轻提示而非强警报；disabled 变成低透明度雾面"
    text: "标题用圆润轻粗体，正文以深灰为主；数字反馈可短暂放大或弹跳"
    accent: "小气泡、涟漪、闪光、软胶滴、轻量星星。装饰要少而轻，避免做成复杂养成或重度消除游戏。"
  forbidden_mistakes: "误判成三消、泡泡龙或儿童教育游戏；重点是“解压点击反馈”和轻量休闲，不是复杂关卡策略"
  style_archetype: "深色竖屏点击泡泡消除解压小游戏"
  archetype_shared_traits:
    - "低门槛休闲解压，核心反馈来自点击、爆破、连击和分数增长"
    - "竖屏单局界面，信息结构简单，按钮大而直接"
    - "彩色圆形泡泡、柔和发光、高对比数字反馈构成主要视觉语言"
  specific_differentiators:
    - "不是 Pop-it 硅胶按压玩具，也不是泡泡龙弹射，而是自由生成泡泡后的点击爆破"
    - "背景是深蓝/深紫夜色感，泡泡像发光半透明小球，而不是软糖色玩具面板"
    - "识别点在得分、连击、爆破计数和全部清除这种清屏爽感"
  signature_semantics:
    must_include:
      - "深蓝竖屏背景"
      - "彩色半透明发光泡泡"
      - "得分、连击、爆破计数"
      - "点击爆破后的加分数字或连击提示"
    should_include:
      - "泡泡高光"
      - "浅色散景大泡泡背景"
      - "清除/暂停/设置等简单功能按钮"
      - "黄色大号连击反馈"
    should_reduce:
      - "硅胶 Pop-it 玩具质感"
      - "粉彩糖果色占满全屏"
      - "复杂角色、剧情或场景装饰"
    must_avoid:
      - "泡泡龙弹珠发射器"
      - "水下气泡冒险"
      - "儿童玩具按压板"
      - "医学/皮肤疙瘩误读"
  background_identity_anchors:
    foreground_forbidden:
      - "人物角色"
      - "发射炮台或弹珠轨道"
      - "整块硅胶按压板"
      - "复杂三消棋盘"
    edge_or_corner_allowed:
      - "设置齿轮"
      - "得分面板"
      - "连击面板"
      - "爆破计数面板"
      - "暂停与全部清除按钮"
    far_background_allowed:
      - "深蓝渐变"
      - "低透明大泡泡散景"
      - "微弱光晕和粒子"
    center_safe_area_allowed:
      - "少量低透明泡泡"
      - "深色干净留白"
      - "点击爆破后的轻量加分数字"
  anti_generic_guardrails:
    - "不能做成 Pop-it 软胶按压玩具；这款的核心是点击泡泡爆破与连击得分"
    - "不能做成泡泡龙；不要出现发射器、轨道、瞄准线或顶部堆叠泡泡墙"
    - "背景必须保持深色、干净、低干扰，让彩色泡泡和数字反馈成为主视觉"
    - "按钮要像轻量小游戏 UI，不要做成复杂商业休闲游戏大厅"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到深蓝竖屏界面、彩色发光泡泡、得分/连击/爆破计数和全部清除按钮，能判断是点击爆破解压泡泡，而不是 Pop-it、泡泡龙或水下气泡背景。"
```

## 13. 伊洛纳手游（像素风格）

```yaml
style_brief:
  name: "伊洛纳"
  aliases: ["Elona Mobile"]
  tags: ["pixel", "roguelike", "rpg", "open_world", "high_freedom"]
  source:
    checked: "https://www.taptap.cn/app/79072"
    confidence: "high"
  game_features:
    type: "高自由度像素 RPG / Roguelike 冒险"
    core_gameplay: "自由探索、地下城冒险、职业与种族搭配、生活玩法、随机事件和高自由成长"
  visual_identity: "复古像素开放世界中的荒诞自由冒险"
  genre_presentation: "用像素 RPG 的轻量画面承载高复杂度系统，把探索、生活、战斗、奇遇和荒诞选择混合成开放式冒险。"
  mood: "自由、古怪、冒险、复古、略荒诞"
  world_elements: "冒险者、地下城、以太风、城镇、酒馆、背包、食物、装备、卷轴、怪物、农场、宠物"
  colors: "土黄/草绿/木棕构成复古 RPG 基础；暗紫和灰蓝表现地下城与以太异变；金色用于装备和奖励"
  materials: "像素石砖、木牌、羊皮纸、旧布、铁器、背包皮革、复古菜单框"
  shape_language: "像素格、硬边矩形、复古 RPG 菜单、低分辨率图标、方块化边框"
  background_visual_language: "像素城镇、地下城或荒野远景，使用网格地面和小型道具散点，背景信息丰富但需保持中心 UI 可读"
  ui_translation:
    panel: "像素 RPG 菜单框+羊皮纸/木牌底，大面板可像冒险日志，小卡可像背包道具格"
    panel_base_style_hint: "低分辨率像素边框、羊皮纸或木板卡底，硬边矩形和网格化道具栏结构；可联想旧 RPG 菜单、背包格、冒险日志，但角色头像、装备 icon、职业文字和怪物图标不作为基础主体或局部装饰来源"
    state: "selected 用金色像素框或小箭头；warning 用暗红/紫色异常提示；disabled 降低亮度并灰化像素边"
    text: "标题可用像素风粗体或复古 RPG 字体，正文需保持现代可读性；数值信息使用清楚的表格/格子"
    accent: "像素小剑、卷轴角标、背包格、木牌钉子、迷你怪物脚印。装饰可古怪但不要干扰复杂信息层级。"
  forbidden_mistakes: "误判成单纯可爱像素农场；必须保留高自由度、RPG、地下城、荒诞冒险和系统复杂度"
  style_archetype: "自由到鬼畜的日式像素 Roguelike RPG"
  archetype_shared_traits:
    - "像素 RPG、地下城、自由探索、奇怪职业和随机事件"
    - "复古界面、道具密度高、荒诞幽默"
    - "生活、冒险、跑商、建造、战斗混合"
  specific_differentiators:
    - "识别核心是自由到鬼畜的混乱人生，而不是普通日式像素冒险"
    - "以太风、荒诞 NPC、奇怪道具和随机事件是独特语义"
    - "画面可以杂乱，但要有老派 RPG 菜单/地图感"
  signature_semantics:
    must_include:
      - "像素地城、复古 RPG UI、奇怪道具、以太风/异变世界暗示"
    should_include:
      - "背包格子、酒馆、地下城入口、城镇小屋、跑商货物"
    should_reduce:
      - "精致二次元立绘"
      - "干净现代 UI"
      - "纯黑暗地牢"
    must_avoid:
      - "标准勇者斗恶龙童话"
      - "暗黑哥特 ARPG"
      - "单纯农场生活"
  background_identity_anchors:
    foreground_forbidden:
      - "大体量写实怪物"
      - "华丽二游角色"
    edge_or_corner_allowed:
      - "背包格、卷轴、奇怪食物、像素金币、任务牌"
    far_background_allowed:
      - "像素城镇、地下城、荒野、酒馆"
    center_safe_area_allowed:
      - "复古地砖、像素草地、低对比背包纹理"
  anti_generic_guardrails:
    - "必须有荒诞自由度和老派 Roguelike 感，不要做成普通像素 RPG"
    - "可以怪，但不要恐怖；可以杂，但要像可探索世界"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能从像素 RPG、奇怪道具、以太异变和高自由生活冒险混合感识别伊洛纳。"
```

## 14. 王牌竞速

```yaml
style_brief:
  name: "王牌竞速"
  aliases: ["Ace Racer"]
  tags: ["racing", "realistic", "high_quality", "drift", "super_skill"]
  source:
    checked: "https://www.taptap.cn/app/190867"
    confidence: "high"
  game_features:
    type: "写实赛车 / 技能竞速 / 多人竞技"
    core_gameplay: "驾驶授权真车和概念车竞速，漂移、氮气、专属大招、多人赛道对抗"
  visual_identity: "写实豪车与超现实技能结合的潮流竞速"
  genre_presentation: "以高品质赛车摄影感包装竞速，再用车辆大招、赛道奇观和潮流包装强化差异化爽感。"
  mood: "速度感、豪华、炫技、潮流、竞技"
  world_elements: "超跑、概念车、氮气尾焰、赛道灯带、城市夜景、国内外实景赛道、速度节、车库"
  colors: "深蓝/黑灰作科技底；霓虹蓝紫表现速度和夜景；橙红用于氮气与高能状态；银白和金属色表现豪车质感"
  materials: "碳纤维、镜面车漆、玻璃、金属铭牌、LED 灯带、赛道沥青"
  shape_language: "低矮锐利、流线型、斜切面、速度线、仪表盘式圆弧"
  background_visual_language: "城市夜景、赛道弯道、车库灯阵或风景赛道远景，使用运动模糊和灯带引导视线，中央留给车辆/信息"
  ui_translation:
    panel: "赛车仪表盘+碳纤维卡片风，大卡像车库展示铭牌，小卡像车辆参数/技能卡"
    panel_base_style_hint: "深色碳纤维或金属玻璃卡底，斜切边、霓虹描边和速度线结构；可联想车库铭牌、仪表盘、赛道灯带，但具体车辆剪影、品牌 Logo、车牌和技能图标不作为基础主体或局部装饰来源"
    state: "selected 用霓虹蓝/橙色灯带高亮；warning 用红色仪表警告；disabled 降低反光并灰化金属边"
    text: "标题可用窄体科技粗字，数据用仪表盘式数字；正文保持白灰高对比"
    accent: "氮气尾焰、速度线、仪表刻度、碳纤维纹理、车库灯条。装饰应强化速度，不要做成普通科幻面板。"
  forbidden_mistakes: "误判成纯写实车展或普通赛车模拟；需要保留“车辆大招”“潮流竞速”“高能技能”的差异化"
  style_archetype: "写实潮流赛车 + 超现实技能竞速"
  archetype_shared_traits:
    - "高光车漆、城市赛道、速度线、赛车 HUD"
    - "授权真车、概念车、改装、速度节竞技"
    - "画面偏写实但带潮流活动包装"
  specific_differentiators:
    - "不是纯模拟赛车，核心是真实车 + 王牌技能/超现实概念车"
    - "赛道和 UI 更像潮流赛车节，而不是严肃 F1 或街头非法飙车"
    - "车辆质感要高级，技能能量要克制点缀"
  signature_semantics:
    must_include:
      - "写实车身反光、速度节 HUD、能量技能轨迹、城市/赛道"
    should_include:
      - "霓虹速度线、车库灯光、弯道标识、赛车编号"
    should_reduce:
      - "过度卡通漂移"
      - "非法街头帮派涂鸦"
      - "纯 F1 赛事感"
    must_avoid:
      - "马里奥赛车道具喜剧"
      - "低模卡通赛车"
      - "重度赛博朋克飞车"
  background_identity_anchors:
    foreground_forbidden:
      - "巨大车头遮挡所有信息"
      - "车手人物主视觉"
    edge_or_corner_allowed:
      - "速度表、技能能量条、轮胎痕、车钥匙、赛事灯牌"
    far_background_allowed:
      - "城市赛道、海岸公路、车库、速度节舞台"
    center_safe_area_allowed:
      - "赛道沥青、速度光带、低对比碳纤维纹理"
  anti_generic_guardrails:
    - "必须同时有写实车质感和超现实技能竞速，不要只做成普通赛车海报"
    - "潮流包装要高级克制，不能变成杂乱夜店霓虹"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能看到写实授权车质感、速度节包装和技能竞速能量，而不是普通漂移赛车。"
```

## 15. 绝尘漂移

```yaml
style_brief:
  name: "绝尘漂移"
  aliases: ["Hot Slide"]
  tags: ["racing", "drift", "mobile", "competitive", "arcade"]
  source:
    checked: "https://www.taptap.cn/app/41192"
    confidence: "medium"
  game_features:
    type: "漂移赛车 / 排位竞技 / 轻操作竞速"
    core_gameplay: "通过漂移技巧、车辆培养、配件调校和赛道熟练度进行竞速对抗"
  visual_identity: "轻量街机漂移与车辆收藏竞技"
  genre_presentation: "把赛车竞速聚焦到漂移手感、弯道控制和车辆养成，以更直接的移动端操作表达速度爽感。"
  mood: "爽快、专注、竞技、街机、速度感"
  world_elements: "漂移车、轮胎烟、弯道、赛道护栏、计时牌、排位徽章、车辆零件、氮气、车库"
  colors: "黑灰/深蓝作速度底；亮橙表现漂移热度；青蓝用于氮气和科技提示；白色用于计时和赛道标线"
  materials: "沥青、橡胶轮胎、喷漆车身、金属配件、赛道反光牌、车库地坪"
  shape_language: "斜切、长条、弯道弧线、速度线、计时器数字块，整体比王牌竞速更轻、更街机"
  background_visual_language: "弯道赛道、轮胎烟雾、车库或简化赛道地图，背景可强调漂移轨迹和胎痕，中央信息区域保持清楚"
  ui_translation:
    panel: "街机赛车参数卡+赛道计时牌风，大卡像车辆调校面板，小卡像配件或排位标签"
    panel_base_style_hint: "深色赛道/车库卡底，斜切边、胎痕纹理、计时牌分区和轻金属边框；可联想漂移轨迹、配件盒、排位铭牌，但具体车型、车标、零件 icon 和赛道名不作为基础主体或局部装饰来源"
    state: "selected 用亮橙漂移火花或青蓝氮气线；warning 用红橙失控/碰撞提示；disabled 变为灰黑胎痕底"
    text: "标题可用赛车海报感粗体，数据用电子计时数字；按钮文字必须高对比"
    accent: "胎痕、漂移烟、弯道箭头、计时器、零件螺丝、排位徽章。装饰要服务于漂移和调校语义。"
  forbidden_mistakes: "误判成重写实豪车竞速；更应突出移动端街机漂移、弯道技巧、排位和车辆调校"
  style_archetype: "清新卡通漂移竞速小游戏"
  archetype_shared_traits:
    - "赛车、弯道、漂移烟、轻竞技"
    - "卡通化车辆和赛道，节奏轻快"
    - "强调漂移操作、关卡路线和冲线爽感"
  specific_differentiators:
    - "比王牌竞速更轻、更卡通、更小游戏化"
    - "识别重点是漂移轨迹和清新赛道，不是高写实车体"
    - "车辆可以简化，重点在弯道节奏与漂移烟雾"
  signature_semantics:
    must_include:
      - "卡通赛车、夸张漂移轨迹、弯道箭头、清新赛道"
    should_include:
      - "轮胎烟、路锥、赛道护栏、冲线旗、小型联赛牌"
    should_reduce:
      - "写实车漆反射"
      - "复杂赛车改装"
      - "黑暗地下飙车"
    must_avoid:
      - "王牌竞速式写实豪车"
      - "军事载具"
      - "科幻反重力赛车"
  background_identity_anchors:
    foreground_forbidden:
      - "真实豪车大特写"
      - "复杂车手人物"
    edge_or_corner_allowed:
      - "漂移箭头、路锥、轮胎痕、旗帜、计时牌"
    far_background_allowed:
      - "卡通赛道、山路、城市小路、联赛终点门"
    center_safe_area_allowed:
      - "赛道弯道、浅色沥青、低对比速度线"
  anti_generic_guardrails:
    - "必须突出漂移，不能只画一辆卡通车"
    - "整体要轻快清新，不要向写实竞速或重度改装靠"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "通过卡通车、清新赛道和夸张漂移轨迹识别为绝尘漂移，而不是写实赛车。"
```

## 16. 无限大

```yaml
style_brief:
  name: "无限大"
  aliases: ["ANANTA", "Project Mugen"]
  tags: ["open_world", "urban", "anime", "rpg", "high_freedom"]
  source:
    checked: "https://www.taptap.cn/app/383874"
    confidence: "high"
  game_features:
    type: "都市开放世界 RPG / 二次元动作"
    core_gameplay: "在高自由度都市中探索、切换身份角色、组建团队、处理城市异常威胁和日常事件"
  visual_identity: "霓虹都市、社交媒体和异常事件交织的二次元开放世界"
  genre_presentation: "把开放世界冒险从荒野转译到现代都市，用商业街、街巷、社交媒体、城市势力和异常危机制造日常与超常的反差。"
  mood: "都市、自由、潮流、轻奇幻、青春、悬疑"
  world_elements: "商业中心、街巷、霓虹招牌、社交媒体界面、对策局、黑客、物流、巡卫署、异常事件、伙伴团队"
  colors: "城市夜景黑蓝作底；霓虹粉紫/青蓝表现潮流与网络感；暖黄表现街边烟火气；红色用于异常危机"
  materials: "玻璃幕墙、霓虹灯牌、亚克力卡片、手机屏幕、城市广告牌、金属栏杆、潮流贴纸"
  shape_language: "现代城市 UI、手机卡片、霓虹描边、潮流贴纸、圆角矩形与错层信息流"
  background_visual_language: "繁华商业街、地铁口、天台、街角便利店或霓虹夜景，边缘可放广告牌和社交媒体弹窗，中央干净"
  ui_translation:
    panel: "手机信息流+都市霓虹卡片风，大卡像城市任务 App，小卡像社交媒体动态或街区标签"
    panel_base_style_hint: "深色玻璃/亚克力卡底，圆角错层、霓虹边线、手机信息流分区和城市广告贴纸感；可联想手机 App、地铁导视、霓虹招牌，但角色立绘、势力 Logo、社媒头像和具体任务 icon 不作为基础主体或局部装饰来源"
    state: "selected 用霓虹青蓝/粉紫发光；warning 用异常红色 glitch；disabled 降低透明度并做灰色失联状态"
    text: "标题可用现代无衬线粗体或潮流海报字；正文像手机 UI 信息流，注意留白"
    accent: "霓虹灯条、社媒弹窗、城市导视箭头、二维码感纹理、贴纸、glitch 小块。装饰应体现都市开放世界，不要变成纯赛博朋克。"
  forbidden_mistakes: "误判成传统奇幻开放世界或重科幻赛博；关键是现代都市、社交媒体、日常烟火气与异常危机并存"
  style_archetype: "现代都市开放世界二次元动作 RPG"
  archetype_shared_traits:
    - "现代城市、高楼、街道、交通、年轻角色、开放世界探索"
    - "二次元角色与都市生活结合，包含动作、跑酷、载具或城市互动"
    - "视觉偏明快都市，不是末日或纯赛博"
  specific_differentiators:
    - "重点是现实感都市里的无限行动可能，不是传统幻想大陆"
    - "城市要有年轻潮流、日常街区和高机动探索感"
    - "不能只做霓虹赛博，要更像当代亚洲大城市开放世界"
  signature_semantics:
    must_include:
      - "现代都市天际线、街区交通、高机动动作路径、年轻潮流符号"
    should_include:
      - "天桥、地铁口、便利店招牌、滑行/钩索路径、城市电子屏"
    should_reduce:
      - "过度赛博朋克"
      - "魔法幻想符号"
      - "废土破败"
    must_avoid:
      - "GTA 写实犯罪都市"
      - "赛博朋克夜城"
      - "校园恋爱小镇"
  background_identity_anchors:
    foreground_forbidden:
      - "真实暴力犯罪元素"
      - "大面积枪械军火"
    edge_or_corner_allowed:
      - "城市路牌、地铁标识、便利店灯箱、手机 UI、滑行轨迹"
    far_background_allowed:
      - "现代高楼、天桥、商业街、城市天际线"
    center_safe_area_allowed:
      - "浅色街道路面、天空、玻璃幕墙反光"
  anti_generic_guardrails:
    - "必须是现代都市开放世界，不要做成普通二游城市背景"
    - "城市要可行动、可探索，不只是漂亮商业街"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能通过现代亚洲都市、高机动路径和二次元开放世界感区别于赛博城市或写实 GTA。"
```

## 17. 代号：新生

```yaml
style_brief:
  name: "代号：新生"
  aliases: ["代号新生"]
  tags: ["extraction_shooter", "top_down", "animal_agents", "post_apocalyptic", "snowfield"]
  source:
    checked: "https://www.taptap.cn/app/826592"
    confidence: "medium"
  game_features:
    type: "俯视角搜打撤射击 / 末世生存"
    core_gameplay: "小动物特工组队，在风雪末世中俯视角搜索物资、战斗、撤离，并建设移动庇护所"
  visual_identity: "毛茸茸小动物特工与风雪末班车末世"
  genre_presentation: "用可爱动物角色中和搜打撤的硬核压力，把末世、物资争夺和撤离风险包装成俯视角战术小队行动。"
  mood: "可爱、紧张、寒冷、末世、轻硬核"
  world_elements: "小动物特工、硬核枪械、风雪冰原、末班车、火种、移动基地、物资箱、庇护所、企业与军团"
  colors: "冰蓝/雪白/灰蓝作环境基调；暖橙表现火种、车灯和庇护所温度；军绿和深灰表现装备；红色用于危险"
  materials: "毛绒、军用布料、冰霜金属、旧列车钢板、补给箱、橡胶护具、暖光灯"
  shape_language: "俯视角小队图标、圆润角色感与硬质战术框结合，既可爱又有装备重量"
  background_visual_language: "雪原、末班车车厢、移动基地或冰封废墟，使用风雪颗粒和暖色车灯制造冷暖对比，中央留出战术信息区"
  ui_translation:
    panel: "末班车庇护所+战术物资卡风，大卡像列车车厢公告板，小卡像补给箱标签或特工档案"
    panel_base_style_hint: "冰霜灰蓝金属/旧列车钢板卡底，圆角但带铆钉、布贴和暖光边；可联想移动庇护所、补给箱、列车公告牌，但小动物头像、枪械 icon、势力徽章和具体物资图标不作为基础主体或局部装饰来源"
    state: "selected 用暖橙车灯或火种光；warning 用红色低温/敌袭警示；disabled 用冰霜覆盖和低亮度"
    text: "标题可用厚实但不凶硬的战术字体；正文保持深灰/白色高可读；资源数字可像补给标签"
    accent: "爪印、雪花、火种灯、列车票、补给封条、布贴、铆钉。装饰要平衡可爱和末世，不要变成纯萌宠养成。"
  forbidden_mistakes: "误判成动物森友会式温馨经营；必须保留搜打撤、风雪末世、俯视角战术和物资撤离压力"
  style_archetype: "俯视角动物特工雪地搜打撤"
  archetype_shared_traits:
    - "搜打撤、物资、撤离点、多人合作、庇护所建设"
    - "俯视角战术射击，资源风险与收益并存"
    - "末世环境、危险区域、装备和背包管理"
  specific_differentiators:
    - "最强差异点是可爱小动物特工 + 风雪末世 + 末班车基地"
    - "不是硬核真人军事搜打撤，而是萌系动物与寒冷末世的反差"
    - "移动钢铁基地/末班车是背景识别核心"
  signature_semantics:
    must_include:
      - "小动物特工、雪地末世、俯视角战术路线、物资撤离、末班车"
    should_include:
      - "背包物资、火种、列车车厢、雪地脚印、简化枪械"
    should_reduce:
      - "写实人类士兵"
      - "血腥尸潮"
      - "纯可爱动物乐园"
    must_avoid:
      - "三角洲式现代军事"
      - "动物森友会治愈小镇"
      - "冰雪童话"
  background_identity_anchors:
    foreground_forbidden:
      - "写实士兵"
      - "巨大怪物"
    edge_or_corner_allowed:
      - "动物爪印、物资箱、车票、火种灯、撤离箭头"
    far_background_allowed:
      - "风雪冰原、末班车、废弃设施、临时庇护所"
    center_safe_area_allowed:
      - "雪地、低对比冰面、轻雾、俯视角地形网格"
  anti_generic_guardrails:
    - "必须保留动物特工与末世搜打撤的反差，不要只做雪地射击"
    - "末班车/火种/移动基地是关键，不能缺"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到小动物特工、风雪末世、物资撤离和末班车，就能区别于硬核搜打撤或普通萌宠游戏。"
```

## 18. 铃兰之剑

```yaml
style_brief:
  name: "铃兰之剑"
  aliases: ["铃兰之剑：为这和平的世界", "Sword of Convallaria"]
  tags: ["srpg", "pixel", "fantasy", "tactical", "dramatic"]
  source:
    checked: "https://www.taptap.cn/app/209866"
    confidence: "high"
  game_features:
    type: "本格战棋 SRPG / 像素幻想剧情"
    core_gameplay: "格子战棋、地形利用、角色养成、佣兵团经营、势力抉择和多线剧情推进"
  visual_identity: "新世代像素战棋与严肃幻想群像剧"
  genre_presentation: "以精致 NeoPixel 像素和舞台剧式剧情表现经典战棋，把战争、信念、牺牲和选择转译为克制而厚重的幻想策略体验。"
  mood: "史诗、克制、悲壮、温暖、策略、命运感"
  world_elements: "铃兰小镇、佣兵团、圣晶石、港口都市、地牢、战旗、剑盾、火把、地图、悬崖、炸药桶、各方势力"
  colors: "羊皮纸米黄和木棕作界面底；深蓝/墨绿表现战争阴影；金色表现信念与史诗感；红色用于冲突和危险"
  materials: "羊皮纸、木板、皮革封面、铜制边框、像素石砖、战旗布料、烛光"
  shape_language: "像素格、战棋网格、古典卷轴、硬边像素框、徽章式角标、舞台布景层次"
  background_visual_language: "像素小镇、战场棋盘、佣兵团据点或泛黄地图远景，使用暖光与暗部对比营造叙事厚度，中央保持战术信息清晰"
  ui_translation:
    panel: "古典战棋菜单+羊皮纸卷轴风，大卡像佣兵团任务书，小卡像角色战术牌或势力情报"
    panel_base_style_hint: "羊皮纸/木板/皮革卡底，像素硬边、铜色细框、战棋网格暗纹和卷轴分区；可联想佣兵任务书、战棋地图、古典菜单，但角色立绘、武器 icon、势力徽章和剧情标题不作为基础主体或局部装饰来源"
    state: "selected 用金色战棋格或烛光描边；warning 用暗红战场标记；disabled 用褪色羊皮纸和低对比铜边"
    text: "标题可用古典衬线或像素化标题字，正文保持清晰端正；剧情文字需留足行距"
    accent: "战棋格、卷轴角、火漆印、战旗布、铜钉、烛火、地图线。装饰要克制，避免变成轻浮二次元或普通像素冒险。"
  forbidden_mistakes: "误判成可爱像素 RPG 或纯日式二次元卡牌；必须保留严肃战争叙事、本格战棋、佣兵团与命运抉择"
  style_archetype: "新世代本格像素 SRPG + 中世纪群像战争"
  archetype_shared_traits:
    - "像素角色、战棋格、佣兵团、王国战争、阵营选择"
    - "中世纪幻想、政治冲突、信念抉择、剧情演出"
    - "场景像舞台剧与战棋棋盘结合"
  specific_differentiators:
    - "不是普通复古像素，而是精致 NeoPixel 与严肃政治叙事"
    - "核心是伊利亚、小国夹缝、圣晶石、佣兵团和艰难选择"
    - "情绪有史诗感和悲悯感，不是轻松勇者冒险"
  signature_semantics:
    must_include:
      - "像素战棋格、佣兵旗帜、中世纪城镇、圣晶石、战火"
    should_include:
      - "酒馆、地图卷轴、石墙、悬崖/高低差、火把、破损旗帜"
    should_reduce:
      - "可爱 Q 版冒险"
      - "纯 JRPG 魔法光效"
      - "过度黑暗血腥"
    must_avoid:
      - "梦幻西游式 Q 版国风"
      - "暗黑地牢恐怖"
      - "现代军事战棋"
  background_identity_anchors:
    foreground_forbidden:
      - "大比例角色立绘"
      - "写实战争尸体"
    edge_or_corner_allowed:
      - "棋盘格、圣晶石、佣兵徽章、卷轴、长剑"
    far_background_allowed:
      - "伊利亚城镇、山地战场、港口、城堡、战火远景"
    center_safe_area_allowed:
      - "石砖地、木质舞台、低对比棋盘格"
  anti_generic_guardrails:
    - "必须有战棋高低差和像素舞台感，不能只是中世纪像素背景"
    - "情绪要有信念与牺牲，不要过度萌化"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "能通过精致像素战棋、佣兵团、圣晶石和伊利亚战争叙事识别铃兰之剑。"
```

## 19. 梦幻西游

```yaml
style_brief:
  name: "梦幻西游"
  aliases: ["梦幻西游手游", "Fantasy Westward Journey"]
  tags: ["turn_based", "mmorpg", "q_version", "xiyou", "social", "classic"]
  source:
    checked: "https://www.taptap.cn/app/2315"
    confidence: "high"
  game_features:
    type: "国民级回合制 MMORPG / 西游题材社交 RPG"
    core_gameplay: "经典回合制战斗、门派养成、宠物培养、组队日常、师门捉鬼、押镖、副本、帮派和社交玩法"
  visual_identity: "Q版西游幻想与热闹三界社交"
  genre_presentation: "把《西游记》神魔世界转译为明亮亲和的 Q 版回合制江湖，用门派、宠物、帮派、日常任务和社交关系构成长期养成体验。"
  mood: "热闹、经典、亲切、喜庆、仙侠、社交"
  world_elements: "长安城、师门、仙魔人三族、宠物宝宝、法宝、祥云、灯笼、卷轴、铜钱、帮派、任务牌、回合战斗阵法"
  colors: "暖金/朱红表现国风喜庆与奖励；天青/碧绿表现仙气和清爽；米白/浅黄作面板底；深蓝紫用于夜景、法术和神魔氛围"
  materials: "宣纸、卷轴、木牌、锦缎、玉石、铜钱、祥云纹、灯笼纸、古典窗棂"
  shape_language: "圆润 Q 版、古典卷轴、祥云曲线、木牌边框、软圆角、对称中式装饰"
  background_visual_language: "长安街市、仙山云海、门派庭院或节日灯会远景，边缘可放灯笼、祥云、木牌和宠物小元素，中央保持任务与数值信息清楚"
  ui_translation:
    panel: "国风卷轴+Q版 MMORPG 任务卡风，大卡像师门任务书或帮派公告，小卡像宠物/道具/活动标签"
    panel_base_style_hint: "米白宣纸或浅金卷轴卡底，圆润木质/锦缎边框、祥云暗纹和轻微玉石高光；可联想师门任务书、长安告示牌、古典窗棂和节庆灯笼，但角色头像、宠物图标、门派 Logo、技能 icon 和具体活动文字不作为基础主体或局部装饰来源"
    state: "selected 用金色祥云描边或玉石高光；warning 用朱红灯笼/印章式提醒；disabled 降低饱和度并做褪色宣纸感"
    text: "标题可用圆润国风粗体或古典牌匾字，正文保持现代清晰；奖励、战力、等级数字可用金色或红金标签突出"
    accent: "祥云、灯笼、铜钱、卷轴角、玉佩、木牌钉、阵法纹、宠物脚印。装饰要喜庆亲和，但不要堆满导致像春节活动页。"
  forbidden_mistakes: "误判成写实仙侠、硬核武侠或普通二次元卡牌；必须保留 Q版西游、经典回合、门派宠物、长安社交和国民级热闹感"
  style_archetype: "Q版国风西游回合制 MMORPG"
  archetype_shared_traits:
    - "Q 版角色、国风建筑、门派、回合制战斗、社交组队"
    - "明亮喜庆色彩、祥云纹样、古典装饰、仙妖人三界"
    - "强调长期养成、帮派、师门、抓鬼、副本等社交循环"
  specific_differentiators:
    - "不是普通国风仙侠，而是西游题材 + Q 版圆润 + 门派社交"
    - "视觉应有三界烟火气，不是冷峻修仙或写实古风"
    - "门派、法宝、祥云、长安街市和萌化妖怪是强识别"
  signature_semantics:
    must_include:
      - "Q版国风、祥云、长安/门派建筑、西游妖怪/法宝语义"
    should_include:
      - "灯笼、铜钱、葫芦、桃花、石桥、门派旗、回合制阵法"
    should_reduce:
      - "写实仙侠"
      - "水墨空灵"
      - "暗黑妖魔"
    must_avoid:
      - "武侠江湖写实"
      - "二次元幻想大陆"
      - "欧美魔幻"
  background_identity_anchors:
    foreground_forbidden:
      - "写实人物大侠"
      - "恐怖妖怪"
    edge_or_corner_allowed:
      - "祥云、灯笼、葫芦、铜钱、桃花、门派小旗"
    far_background_allowed:
      - "长安城、门派山门、仙山、石桥流水、庙会街景"
    center_safe_area_allowed:
      - "浅色云纹、青石地面、低对比国风纹样"
  anti_generic_guardrails:
    - "必须保留 Q 版西游和社交回合制气质，不能只做泛国风"
    - "国风要热闹、圆润、烟火气，不要变成冷淡水墨"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "看到 Q 版西游、祥云门派、长安烟火气和回合制阵法，就能判断是梦幻西游。"
```

## 20. 怪物猎人：旅人

```yaml
style_brief:
  name: "怪物猎人：旅人"
  aliases: ["Monster Hunter Outlanders", "魔物獵人：旅人"]
  tags: ["monster_hunter", "arpg", "open_world", "hunting", "multiplayer", "adventure"]
  source:
    checked: "https://www.taptap.cn/app/243984"
    confidence: "high"
  game_features:
    type: "开放世界狩猎动作手游 / 多人共斗 ARPG"
    core_gameplay: "使用经典武器狩猎大型怪物，采集、探索、制作装备、组队共斗，并在全新世界观中遭遇独占新怪物与冒险家角色"
  visual_identity: "原始生态、巨兽狩猎与冒险营地感结合的掌上怪猎"
  genre_presentation: "把怪物猎人系列的武器手感、生态观察、部位破坏和多人狩猎，转译为更适合移动端的开放世界冒险与小队共斗体验。"
  mood: "野性、冒险、紧张、厚重、探索、共斗"
  world_elements: "大型怪物、猎人、经典武器、随从、营地、地图、足迹、爪痕、生态群落、采集点、素材、烤肉、陷阱、狩猎委托"
  colors: "森林绿/岩土棕作自然生态底；皮革棕和骨白表现猎人装备；橙黄用于营火、任务和奖励；红色仅用于怪物怒气、危险和受击提示"
  materials: "皮革、兽骨、木材、粗布、铁器、石板、地图羊皮纸、营地绳结、怪物鳞片"
  shape_language: "粗犷、原始、手工感、厚边框、不规则切角、狩猎铭牌、地图标记和部落式纹样"
  background_visual_language: "森林、峡谷、荒原、营地或巨兽栖息地远景，边缘可放爪痕、足迹、地图标线和采集素材，中央保持任务与装备信息清晰"
  ui_translation:
    panel: "猎人委托板+生态调查笔记风，大卡像狩猎任务书或营地公告，小卡像素材标签、武器卡或怪物调查记录"
    panel_base_style_hint: "皮革/木板/羊皮纸混合卡底，粗糙不规则边、兽骨或金属铆钉加固、地图暗纹和爪痕刻印；可联想狩猎委托板、生态调查笔记、营地物资箱，但具体怪物头像、武器 icon、猎人角色、随从形象和任务文字不作为基础主体或局部装饰来源"
    state: "selected 用橙黄营火光或猎人标记框；warning 用红色怪物怒气/危险爪痕；disabled 用灰褐色旧皮革和低对比磨损边"
    text: "标题可用粗犷冒险字体或刻印感标题字，正文保持清晰；素材数量、任务等级、危险度可用铭牌式标签突出"
    accent: "爪痕、足迹、兽骨、绳结、营火、地图针、素材袋、调查印章。装饰要有狩猎/生态/营地语义，不要做成普通奇幻 RPG 或写实军事面板。"
  forbidden_mistakes: "误判成恐龙生存、普通开放世界或硬核写实狩猎模拟；必须保留怪猎 IP 的武器狩猎、怪物生态、素材制作、营地委托和多人共斗语义"
  style_archetype: "写实奇幻生态狩猎动作手游"
  archetype_shared_traits:
    - "大型怪物、狩猎武器、生态环境、营地、素材采集"
    - "强调怪物体型压迫、武器动作反馈和自然地貌"
    - "奇幻生态比魔法特效更重要"
  specific_differentiators:
    - "核心是怪猎正版狩猎体验在移动端的适配，而不是普通开放世界打怪"
    - "必须有怪物生态、猎人营地、武器职业和随从语义"
    - "旅人可加入新岛屿、新冒险家、独占怪物等移动端新作特征"
  signature_semantics:
    must_include:
      - "巨型生态怪物、猎人武器、营地、自然地貌、随从/冒险者语义"
    should_include:
      - "大剑、长枪、弓、篝火、补给箱、脚印、怪物鳞片、狩猎地图"
    should_reduce:
      - "魔法阵"
      - "二次元技能光污染"
      - "MMO 自动战斗 UI"
    must_avoid:
      - "恐龙公园"
      - "暗黑屠龙魔幻"
      - "原神式明亮幻想冒险"
  background_identity_anchors:
    foreground_forbidden:
      - "怪物头部大特写压满画面"
      - "角色抽卡立绘"
    edge_or_corner_allowed:
      - "武器架、脚印、鳞片、补给箱、篝火、猎人徽章"
    far_background_allowed:
      - "岛屿荒野、山林、沼泽、火山、怪物活动痕迹"
    center_safe_area_allowed:
      - "自然地面、雾气、低对比草地/岩壁、营地光影"
  anti_generic_guardrails:
    - "必须表现狩猎生态，不是单纯打 BOSS 或开放世界冒险"
    - "怪物要像生态链中的生物，而不是恶魔或恐龙展品"
  visual_identity_test:
    question: "如果去掉游戏名，这张背景是否仍能区别于同原型下的其他游戏？"
    pass_condition: "通过大型生态怪物痕迹、猎人武器、营地和自然狩猎场，能识别为怪猎旅人，而不是泛奇幻 ARPG。"
```
