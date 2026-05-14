# 01 Style Knowledge

> 角色：根据关键词、标签或游戏名检索游戏视觉特征，并输出 `style_brief`。本文件只负责风格理解，不负责物料生成、资源预算、尺寸边界或 Maker 实现。

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
- 9-slice 面板可以参考哪些材质、形状、色彩和 panel 转译方向。

本文件不能回答：

- 要生成几张图片、图片尺寸是多少、9-slice 参数是多少。
- 背景图、面板图、装饰图、状态图的具体产物规格。
- Maker 支持什么能力、资源预算是多少、QA 失败归因到哪一层。

这些问题分别由 `02-production-constraints.md` 和 `03-material-generation.md` 处理。

## 9-slice 风格输入边界

9-slice 面板生成可以把 `style_brief` 当作视觉方向来源，但只能消费可转译为“基础面板材质、中心底色倾向、边框轮廓、线条气质”的字段。

基础 9-slice 的主要风格输入字段：

```yaml
9slice_primary_style_fields:
  - materials
  - shape_language
  - ui_translation.panel_base_style_hint
  - colors
  - forbidden_mistakes
```

只作为语义理解和误读过滤上下文的字段：

```yaml
9slice_context_only_fields:
  - ui_translation.panel
  - world_elements
  - signature_semantics.must_include
  - signature_semantics.must_avoid
  - specific_differentiators
  - anti_generic_guardrails
```

`9slice_context_only_fields` 只能辅助理解风格语义、避免通用化和过滤误读，不能把其中的角色、世界观物件、徽章、状态、标签、布局组合、文字、图标、logo 或 IP 符号直接画进基础 9-slice。

`ui_translation.panel` 描述完整面板系统，可以包含大卡、小卡、头像、标签、状态、徽章、角标和布局组合建议；它不是基础 9-slice 源图规格。

`ui_translation.panel_base_style_hint` 只描述基础底板可借用的风格联想，例如基础底板材质、中心底色倾向、边框轮廓、线条气质、抽象点缀候选和不适合作为基础底板来源的元素。

## 检索方式

1. 先执行 `00. Style Discovery Fallback Rule` 中的 Lookup Priority。
2. 只接受 game_name 精确匹配或已登记 alias 精确匹配作为已有条目命中。
3. 未命中 game_name / alias 时，必须先查找 TapTap / 官网 / 商店页资料。
4. 搜索并提取文字资料后，才允许用 `tags`、玩法品类、情绪词和世界观元素映射已有 style_archetype。
5. 禁止在未搜索资料的情况下，按同系列、续作、IP 相近、名称相似或主观印象直接套用已有条目。
6. 搜索后仍没有合适 archetype 时，输出 Decision Output，决定创建新 archetype 还是使用 Generic Adaptive Game UI Fallback。
7. 输出给生产链路时，只输出 `style_brief` 或 Style Discovery 决策结果，不要把生产参数混入其中。

## Style Specificity Contract

`style_brief` 不是“风格大类标签”，而是具名游戏的可识别风格描述。传递给 `03-material-generation.md` 时，不允许把完整风格理解压缩成通用 archetype。

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

如果现有条目尚未显式填写 Style Specificity Contract，使用该条目前必须先从 `visual_identity`、`genre_presentation`、`game_features.core_gameplay`、`world_elements`、`materials`、`shape_language`、`background_visual_language` 和 `forbidden_mistakes` 中补全这些字段，再进入 03 生成物料。

如果旧条目没有 `ui_translation.panel_base_style_hint`，不能直接把 `ui_translation.panel` 当成基础 9-slice 规格。必须先从 `materials`、`shape_language`、`colors` 和 `ui_translation.panel` 中抽取基础底板材质、中心底色倾向、边框轮廓和线条气质，并过滤掉头像、状态、徽章、标签、文字、图标和布局组合。

### 传递失败判定

如果 03 生成的 prompt 只剩下大类题材、通用颜色、通用材质和中心留白规则，即使产物满足 UI 可读性，也判定为 `style_specificity_loss`。

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
    panel_base_style_hint: "奶油白半透明纸卡或浅木边手作卡底，软圆角、浅木描边和轻纸张颗粒；可联想邮票边、手写标签、布料贴片边角，但小房子、猫爪、花草堆、路牌和功能贴纸不作为基础底板来源"
    state: "selected 用暖黄描边+小贴纸；disabled 降低饱和和透明度；locked 用小锁牌；warning 才用柔和橙红"
    text: "标题可用圆润手写感，正文使用清晰黑棕色；小字禁止强描边"
    accent: "树叶、邮票、手写标签、木牌、花瓣、小房子、猫爪。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "被误判成真实田园、儿童绘本或高饱和农场经营"
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
    panel_base_style_hint: "白底漫画纸卡或亮面颜料薄层卡底，粗黑描边、不规则喷溅边和倾斜贴纸感；可联想胶带角、颜料滴边、漫画爆炸框边缘，但神明娘、喷漆罐、文字感叹号和大面积颜料泼洒不作为基础底板来源"
    state: "selected 用彩色描边+喷溅角标；warning 用红黑警示，不可与普通彩色装饰混淆"
    text: "标题可用漫画粗体，正文用黑色无衬线；副文字放在纯色底上"
    accent: "喷漆滴落、胶带、手绘箭头、漫画感叹号、颜料块、神格小徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通赛博霓虹或儿童绘画；不要把所有区域都喷满颜料"
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
    panel_base_style_hint: "深海蓝透明像素玻璃或米白木质菜单卡底，像素块描边、木牌边和轻气泡质感；可联想气泡边、菜单纸角、像素木片，但鱼群、寿司图形、氧气表和海底场景不作为基础底板来源"
    state: "selected 用气泡高光或橙黄菜单贴；locked 用小锁/深海阴影；warning 用氧气红"
    text: "标题可用复古像素字，正文保持常规清晰字体"
    accent: "气泡、鱼影、寿司、氧气表、像素箭头、木牌价格签。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成纯海洋治愈或纯餐厅经营，忽略像素幽默与双场景"
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
    panel_base_style_hint: "浅灰/米白厚纸卡或扁平塑料卡底，圆润厚边和投票便签材质感；可联想胶带角、纸片折角、贴边投票便签，但头像圆章、身份文字、状态标记和角色图标不作为基础底板来源"
    state: "selected 用黄色投票框；warning 用红色警报灯；disabled 灰化但保留轮廓"
    text: "标题可圆润粗体，正文黑灰；状态词用高对比标签"
    accent: "羽毛、脚印、问号、投票贴纸、警报灯、身份徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成儿童低龄卡通；需要保留“可疑/投票/推理”语义"
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
    panel_base_style_hint: "珍珠白磨砂玻璃或丝绸纸卡底，纤细圆角、玫瑰金/银灰细边和柔光层次；可联想杂志页角、镜框细线、星光切片感，但礼服、珠宝主体、化妆镜和摄影灯不作为基础底板来源"
    state: "selected 用玫瑰金细描边+星点；warning 用珊瑚红小标签，不污染整体"
    text: "标题可高对比优雅字重，正文现代无衬线，字间距略松"
    accent: "钻石、缎带、衣架、镜框、星光、香水瓶、杂志页角。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通粉色少女或婚礼风，忽略现代时装杂志感"
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
    panel_base_style_hint: "泥土棕木牌或浅草地纸卡底，圆润厚描边、木牌边和轻漫画颗粒；可联想种子包纸角、叶片边、泥土小裂纹，但植物角色、僵尸、墓碑主体和草坪格阵不作为基础底板来源"
    state: "selected 用阳光黄光圈；locked 用灰墓碑；warning 用僵尸紫/红警示"
    text: "标题可漫画粗体，正文深棕/黑色"
    accent: "阳光、叶子、种子包、墓碑、路障桶、泥土裂纹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通农场或儿童园艺，必须保留塔防棋盘和僵尸喜剧"
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
    panel_base_style_hint: "深色磨砂玻璃或车票卡质感，细金/银金属边，圆角与轻切角结合；可联想轨道弧线、星图节点和档案卡边，但星图线、命途徽记、车票图形和发光粒子不作为基础底板来源"
    state: "selected 用金色轨道光；warning 用红橙小警示；disabled 降亮度并保留线框"
    text: "标题可细致高对比，正文用浅灰白，避免小字发光"
    accent: "车票、星轨、星星节点、命途徽记、列车窗、档案夹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用蓝紫科幻手游；要保留列车旅行与档案质感"
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
    panel_base_style_hint: "米白纸卡或布艺圆角卡底，浅木/布边、柔软手绘描边和低饱和家居感；可联想猫爪压痕、纸箱胶带、花朵贴边，但猫、家具、料理和钓鱼小物不作为基础底板来源"
    state: "selected 用暖橙描边+猫爪；locked 用小门牌；warning 用柔和红棕"
    text: "标题圆润可爱，正文深灰，副文浅棕"
    accent: "猫爪、花朵、纸箱、咖啡杯、沙发、窗帘、小鱼。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成高饱和儿童装修或真实家装软件"
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
    panel_base_style_hint: "深橄榄/枪灰磨砂终端卡底，硬边切角、窄金属边和克制网格线；可联想装备箱嵌片、坐标短线、军用编号条，但枪械、弹匣、护甲、警戒三角和战场地图不作为基础底板来源"
    state: "selected 用琥珀黄边框+扫描线；locked 用金属锁/封条；warning 用红色高优先级"
    text: "标题窄体硬朗，正文浅灰；小字不要依赖发光"
    accent: "螺丝、军用编号、坐标线、弹孔、警戒三角、装备插槽。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成赛博霓虹或泛科幻蓝 UI；不要过多发光"
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
    panel_base_style_hint: "灰黑工业终端玻璃或磨砂金属板底，模块化硬边、切角细线和冷青能源感；可联想管线嵌片、工业编号、警示胶带边，但干员装备、矿石、轨道物流图和基地设施不作为基础底板来源"
    state: "selected 用青蓝光条或橙黄任务边；warning 与 selected 颜色区分"
    text: "标题冷静工业字，正文高可读无衬线"
    accent: "管线、矿石、工业编号、警示条、物流箭头、基地图标。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成传统明日方舟医疗黑白 UI 或通用未来蓝"
```

## 11. 我的世界：移动版

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
    panel_base_style_hint: "木板/石砖像素卡底，直角块面、物品栏槽边和低分辨率拼接感；可联想合成格、矿石色块、木板接缝，但草方块、工具、火把和洞穴场景不作为基础底板来源"
    state: "selected 用白色像素描边或钻石青；locked 用基岩灰"
    text: "标题可像素体，正文保持清晰无衬线/像素大字"
    accent: "方块、镐子、矿石、火把、合成格、像素箭头。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通像素复古，必须保留体素方块和合成语义"
```

## 12. 王者荣耀

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
    panel_base_style_hint: "深蓝玻璃或旗帜布纹卡底，细金属边、圆角与尖角混合的徽章化轮廓；可联想段位牌边、符文细线、冠冕轮廓片，但英雄徽章、技能符文主体、战场地图和阵营标识不作为基础底板来源"
    state: "selected 用金色光圈/段位星；warning 用红色敌情，不作普通装饰"
    text: "标题可高对比华丽，正文白色或浅金但需足够对比"
    accent: "冠冕、段位星、符文、旗帜、剑盾、战场路线。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成泛蓝紫魔幻手游，需保留竞技和荣耀段位语义"
```

## 13. 鸣潮

```yaml
style_brief:
  name: "鸣潮"
  aliases: []
  tags: ["post-apocalyptic", "resonance", "anime", "open-world"]
  source:
    checked: "https://www.taptap.cn/app/234280"
    confidence: "high"
  game_features:
    type: "开放世界 ARPG / 二次元"
    core_gameplay: "末世后探索、共鸣能力、动作战斗与声骸收集"
  visual_identity: "后启示录声波科幻二次元开放世界"
  genre_presentation: "把动作开放世界视觉化为冷白黑灰、声波共振、断裂城市和高机动战斗界面。"
  mood: "孤寂、清冷、锋利、探索"
  world_elements: "共鸣、声波、残响、废墟、黑石、白色装甲、频谱线"
  colors: "黑/白/灰作大基调；青蓝作共鸣和系统；少量红橙作危险；避免大面积紫粉"
  materials: "冷白玻璃、黑灰金属、频谱 HUD、半透明能量层"
  shape_language: "锐利切角、细线、波形曲线、留白与断裂边"
  background_visual_language: "低饱和废墟/天空远景，中央浅灰或深灰安全区"
  ui_translation:
    panel: "冷白/黑灰半透明卡+青蓝细线，小卡极简"
    panel_base_style_hint: "冷白/黑灰半透明玻璃卡底，锐利切角、青蓝细线和轻频谱断裂感；可联想波形边、碎片嵌片、坐标点，但声骸符号、废墟物件、黑石主体和大面积能量波不作为基础底板来源"
    state: "selected 用青蓝频谱边；warning 用红橙故障条；disabled 去饱和"
    text: "标题冷峻纤细，正文高对比黑白，避免大面积发光"
    accent: "波形、频谱、碎片、黑石纹、坐标点、声骸符号。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用赛博二次元或末世废土脏乱风"
```

## 14. Phigros

```yaml
style_brief:
  name: "Phigros"
  aliases: []
  tags: ["rhythm", "minimal", "geometry", "music"]
  source:
    checked: "https://www.taptap.cn/app/165287"
    confidence: "high"
  game_features:
    type: "音乐节奏 / 抽象"
    core_gameplay: "动态判定线、谱面节奏和音乐章节体验"
  visual_identity: "极简抽象几何音乐节奏"
  genre_presentation: "把节奏游戏视觉化为线条、几何、速度、空白和音乐动势，而不是具象世界观。"
  mood: "纯净、专注、律动、实验"
  world_elements: "判定线、音符、几何块、波形、章节封面、节拍轨迹"
  colors: "黑/白/深灰作底；高纯度单色作章节强调；青紫粉黄可按曲风变化；警示色极少用"
  materials: "纯色面板、半透明玻璃、细线网格、轻粒子"
  shape_language: "极简矩形、锐利线段、非对称动态构图"
  background_visual_language: "大面积纯色/渐变，边缘少量线条和音符，中心绝对干净"
  ui_translation:
    panel: "面板极简，低边框或无边框，靠线条和间距建立层级"
    panel_base_style_hint: "黑白/深灰纯净几何卡底，极细线框、锐利矩形和低密度透明层；可联想判定线片段、几何切片、轻节拍光点，但音符图形、章节封面、波形轨迹和粒子群不作为基础底板来源"
    state: "selected 用细亮线/节拍脉冲；warning 用小红点，不大面积"
    text: "标题可现代几何，正文极简高对比"
    accent: "音符点、判定线、波形、几何碎片、节拍光点。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用霓虹 EDM；需要保留克制极简和动态线条"
```

## 15. 杖剑传说

```yaml
style_brief:
  name: "杖剑传说"
  aliases: []
  tags: ["idle-rpg", "isekai", "light-fantasy", "adventure"]
  source:
    checked: "https://www.taptap.cn/app/717836"
    confidence: "high"
  game_features:
    type: "放置 RPG / 异世界冒险"
    core_gameplay: "轻松放置成长、异世界探索、伙伴、圣兽与转职技能搭配"
  visual_identity: "轻爽异世界放置冒险"
  genre_presentation: "把异世界 RPG 视觉化为浮空岛、小木屋、地图迷雾、幻兽伙伴和不压迫的成长感。"
  mood: "轻松、明亮、冒险、陪伴"
  world_elements: "浮空岛、软床、小木屋、地图迷雾、圣兽、宝箱、剑与法杖"
  colors: "暖白/浅黄作面板；木棕作结构；天空蓝/草绿作探索；金色作宝物；红色仅作战斗警示"
  materials: "羊皮纸、浅木、软布、宝石小徽章、地图纸"
  shape_language: "圆润厚卡、卷轴边、木牌、地图折角"
  background_visual_language: "天空浮岛和小木屋远景，中央保持云雾留白"
  ui_translation:
    panel: "纸质/木质圆角面板，大卡可加地图纹，小卡保留宝箱角标"
    panel_base_style_hint: "暖白羊皮纸或浅木冒险卡底，圆润厚卡、卷轴边和地图纸折痕感；可联想宝箱金属扣、地图针边、云朵纸角，但圣兽、剑杖、宝箱主体和浮空岛场景不作为基础底板来源"
    state: "selected 用金色宝箱光或蓝色探索边；locked 用迷雾遮罩"
    text: "标题冒险手写感，正文深棕清晰"
    accent: "宝箱、云朵、法杖、短剑、地图针、羊、圣兽脚印。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成硬核西幻暗黑或泛 RPG 蓝紫"
```

## 16. 原神

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
    panel_base_style_hint: "米白羊皮纸或浅色磨砂玻璃卡底，细金边、柔圆角和轻古典浮雕感；可联想冒险手册页角、元素晶石色边、卷草细线，但元素徽记、晶蝶、羽毛和七国建筑不作为基础底板来源"
    state: "selected 用金色细边+元素光；warning 用红色火焰图标，不作通用选中"
    text: "标题可古典细腻，正文深灰或白色高对比"
    accent: "元素徽记、晶蝶、羽毛、地图针、卷草纹、星星。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用二次元魔幻或高饱和手游"
```

## 17. 洛克王国：世界

```yaml
style_brief:
  name: "洛克王国：世界"
  aliases: []
  tags: ["pet-collection", "magic", "open-world", "nostalgia"]
  source:
    checked: "https://www.taptap.cn/app/188212"
    confidence: "high"
  game_features:
    type: "精灵收集 / 开放世界 / 社交"
    core_gameplay: "精灵收集对战、开放世界探索、魔法学院与自由社交"
  visual_identity: "明亮童年魔法精灵大世界"
  genre_presentation: "把精灵收集视觉化为魔法学院、精灵伙伴、孵蛋、奇景和轻快社交。"
  mood: "明亮、童趣、怀旧、轻快"
  world_elements: "精灵球/捕捉器、魔法书、学院、新生、孵蛋、奇景、菜园、伙伴羁绊"
  colors: "天空蓝/奶油白作底；草绿作探索；金色作奖励；紫蓝作魔法；红色仅作危险"
  materials: "魔法书纸、圆润玻璃、糖果色徽章、浅木/石板"
  shape_language: "圆润厚卡、学院徽章、书页折角、星星魔法线"
  background_visual_language: "魔法学院或草地远景，中心清爽，边缘放精灵脚印和书页"
  ui_translation:
    panel: "书页卡+圆润玻璃按钮，大卡可加学院徽章，小卡只放精灵脚印"
    panel_base_style_hint: "天空蓝/奶油白书页或圆润玻璃卡底，厚圆角、学院徽章式边框和轻魔法星线；可联想书签边、蛋壳小片、星尘线，但精灵、捕捉器、学院徽章主体和蝴蝶结不作为基础底板来源"
    state: "selected 用金色星星描边；locked 用书锁；warning 用红色感叹号"
    text: "标题圆润魔法感，正文深蓝灰清晰"
    accent: "魔法星、精灵脚印、书签、蛋壳、学院徽章、蝴蝶结。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成冷蓝仙侠或低龄宠物乐园；要保留学院与精灵收集"
```

## 18. 蛋仔派对

```yaml
style_brief:
  name: "蛋仔派对"
  aliases: []
  tags: ["party", "toy", "cute", "ugc"]
  source:
    checked: "https://www.taptap.cn/app/206776"
    confidence: "high"
  game_features:
    type: "派对 / 休闲竞技 / UGC"
    core_gameplay: "多人闯关、圆滚角色、派对地图与 UGC 创作"
  visual_identity: "糖果玩具体圆滚派对竞技"
  genre_presentation: "把派对竞技视觉化为软塑料玩具、糖果色关卡、圆形角色和强社交表情。"
  mood: "欢乐、闹腾、可爱、竞技"
  world_elements: "蛋仔、弹簧、障碍、糖果、盲盒、游乐场、表情贴纸"
  colors: "高明度蓝粉黄作主视觉；白色作面板；紫/橙作活动；红色只作失败/危险"
  materials: "亮面软塑料、糖果胶、贴纸、泡泡玻璃"
  shape_language: "超圆润、厚块面、充气感、胶囊按钮"
  background_visual_language: "游乐场或玩具盒边缘，中央大面积浅色安全区"
  ui_translation:
    panel: "圆鼓鼓面板+软塑料高光，小卡用胶囊形态"
    panel_base_style_hint: "白底亮面软塑料或糖果胶卡底，超圆润厚块面、泡泡高光和胶囊边；可联想盲盒贴边、彩带色块、软弹小凸起，但蛋仔表情、障碍、弹簧和盲盒主体不作为基础底板来源"
    state: "selected 用彩虹描边或弹跳光；locked 用盲盒锁；warning 红色障碍标记"
    text: "标题可胖圆体，正文深色高对比"
    accent: "糖豆、星星、贴纸、弹簧、盲盒、彩带、表情。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通儿童乐园；要保留竞技闯关和盲盒玩具质感"
```

## 19. 鬼谷八荒

```yaml
style_brief:
  name: "鬼谷八荒"
  aliases: []
  tags: ["xianxia", "ink", "cultivation", "sandbox"]
  source:
    checked: "https://www.taptap.cn/app/700558"
    confidence: "high"
  game_features:
    type: "修仙 / 沙盒 / 角色扮演"
    core_gameplay: "开放修仙、宗门、机缘、功法、境界成长与随机事件"
  visual_identity: "东方水墨修仙沙盒"
  genre_presentation: "把修仙沙盒视觉化为山海卷轴、法阵、丹炉、宗门令牌和命运事件。"
  mood: "玄妙、苍茫、修行、古朴"
  world_elements: "山海、云雾、八卦、丹炉、符箓、宗门、功法卷轴、灵石"
  colors: "米黄宣纸作面板；墨黑作文字；青绿/玉色作灵力；朱砂作印章和重点；暗红只作危机"
  materials: "宣纸、墨迹、古铜、玉石、卷轴、石碑"
  shape_language: "水墨边、卷轴角、篆刻印章、圆形法阵"
  background_visual_language: "水墨山峦和云雾远景，中央淡宣纸留白"
  ui_translation:
    panel: "宣纸面板+墨线边+朱砂小印，小卡极简"
    panel_base_style_hint: "米黄宣纸或淡墨卷轴卡底，水墨边、卷轴角和朱砂/玉色细节；可联想篆刻印边、符纸纤维、山形淡纹，但八卦法阵、丹炉、灵石和功法卷轴主体不作为基础底板来源"
    state: "selected 用朱砂印/玉色光；warning 用暗红劫雷符，不作普通选中"
    text: "标题可书法化，正文必须宋/黑体清晰"
    accent: "符箓、八卦、印章、灵石、丹药、山形纹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成冷蓝仙侠或宫廷古风；要保留沙盒修行和道法语义"
```

## 20. 异环

```yaml
style_brief:
  name: "异环"
  aliases: []
  tags: ["urban-fantasy", "supernatural", "open-world", "anime"]
  source:
    checked: "https://www.taptap.cn/app/714119"
    confidence: "high"
  game_features:
    type: "超自然都市 / 开放世界 RPG"
    core_gameplay: "玩家作为异象猎人，在海特洛市接取委托、解决都市异象并体验生活经营"
  visual_identity: "超自然都市怪谈开放世界 RPG"
  genre_presentation: "把开放世界视觉化为现代都市、古董店委托、异象怪谈、轻喜剧角色与稍陌生的日常城市。"
  mood: "都市、怪奇、轻喜剧、时髦"
  world_elements: "海特洛市、异象猎人、古董店、电视头海獭、涂鸦滑板、委托档案、店铺经营"
  colors: "夜色蓝灰/城市灰作底；霓虹粉青作异象；暖黄作店铺生活；红色仅作异常危险"
  materials: "都市玻璃、霓虹招牌、档案纸、古董金属、街头贴纸"
  shape_language: "现代圆角卡、街区标牌、档案夹、轻切角、贴纸叠层"
  background_visual_language: "低对比城市街角/古董店远景，异象元素放边缘，中心干净"
  ui_translation:
    panel: "半透明城市卡+档案夹标签，局部霓虹但不过亮"
    panel_base_style_hint: "浅灰白 / 淡档案纸或低透明城市玻璃底，现代圆角卡和轻档案夹边；可联想异常封条、委托标签角、街头贴纸边角，但霓虹不过亮，电视头、交通牌、涂鸦堆、角色和机械 HUD 不作为基础底板来源"
    state: "selected 用霓虹青粉边；warning 用异常红封条；locked 用档案锁"
    text: "标题现代都市感，正文深浅对比清楚"
    accent: "委托单、古董标签、涂鸦、电视头图标、交通牌、异常封条。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成赛博朋克或纯二次元都市；需保留超自然怪谈和生活经营"
```

## 21. 最强蜗牛

```yaml
style_brief:
  name: "最强蜗牛"
  aliases: []
  tags: ["idle", "absurd", "scrapbook", "collection"]
  source:
    checked: "https://www.taptap.cn/app/187376"
    confidence: "high"
  game_features:
    type: "放置 / 收集 / 荒诞剧情"
    core_gameplay: "蜗牛进化、收集贵重品、探索和无厘头叙事"
  visual_identity: "荒诞手账式蜗牛放置收集"
  genre_presentation: "把放置收集视觉化为蜗牛基地、伪科学档案、博物馆藏品和密集梗图式幽默。"
  mood: "荒诞、搞笑、猎奇、收集癖"
  world_elements: "蜗牛壳、贵重品、基因、抽屉、档案、便签、地球仪、恶搞小物"
  colors: "旧纸黄/木棕作底；红蓝印章作重点；绿色作成长/基因；紫金作稀有；红色作危险/吐槽"
  materials: "旧纸、手账贴纸、木桌、档案夹、复古塑料、瓶罐"
  shape_language: "手作不规则、贴纸堆叠、圆角纸片、旧票据"
  background_visual_language: "桌面手账/收藏柜边缘，中央留出浅纸面"
  ui_translation:
    panel: "纸片堆叠卡+印章角标，大卡可放藏品阴影，小卡不要堆梗"
    panel_base_style_hint: "旧纸黄手账纸片或木桌档案卡底，不规则圆角、贴纸叠层和复古印刷颗粒；可联想旧票边、红蓝印章痕、博物馆标签角，但蜗牛壳、贵重品、瓶罐和恶搞小物不作为基础底板来源"
    state: "selected 用红蓝印章或金色藏品框；warning 用红色吐槽标"
    text: "标题可手账粗体，正文黑棕清晰"
    accent: "蜗牛壳、印章、便签、瓶盖、旧票、博物馆标签。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通可爱动物或硬核科幻进化"
```

## 22. 明日方舟

```yaml
style_brief:
  name: "明日方舟"
  aliases: []
  tags: ["arknights", "tactical", "industrial", "dystopia"]
  source:
    checked: "https://www.taptap.cn/app/70253"
    confidence: "high"
  game_features:
    type: "策略塔防 / 二次元 / 末世科幻"
    core_gameplay: "干员部署、感染者叙事、罗德岛制药和战术关卡"
  visual_identity: "医疗工业末世策略 UI"
  genre_presentation: "把塔防视觉化为制药公司终端、干员档案、灾害警报、黑白灰工业和战术网格。"
  mood: "克制、严肃、专业、压抑"
  world_elements: "罗德岛、感染、源石、医疗箱、干员档案、战术地图、警示标识"
  colors: "黑白灰作主基调；青蓝作系统；橙黄作警示和选中；红色仅作危机/错误"
  materials: "黑白工业终端、医疗档案纸、磨砂玻璃、橙色警示胶带"
  shape_language: "硬边、切角、模块网格、编号、条形码"
  background_visual_language: "暗灰工业墙/战术地图远景，中心低纹理"
  ui_translation:
    panel: "硬边黑白卡+橙色小条，卡片不要过度发光"
    panel_base_style_hint: "黑白灰工业终端或医疗档案卡底，硬边切角、模块网格和橙色短条感；可联想条形码细线、档案夹边、警示胶带片，但罗德岛标识、干员档案头像、源石碎片和医疗十字不作为基础底板来源"
    state: "selected 用橙黄短条/边框；warning 用红色危机，不作选中"
    text: "标题冷峻，正文白/黑高对比，信息层级明确"
    accent: "医疗十字、条形码、源石碎片、战术编号、警示线、档案夹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成泛科幻蓝或废土脏乱；要保留医疗工业和档案克制"
```

## 23. 尘白禁区

```yaml
style_brief:
  name: "尘白禁区"
  aliases: []
  tags: ["sci-fi", "tactical", "shooter", "snow"]
  source:
    checked: "https://www.taptap.cn/app/222089"
    confidence: "high"
  game_features:
    type: "科幻射击 / 二次元 / 动作"
    core_gameplay: "队员协作、枪械射击、泰坦/灾变背景与基地系统"
  visual_identity: "雪域战术科幻美少女射击"
  genre_presentation: "把 TPS 视觉化为白灰冷色、战术装备、隔离区、枪械 HUD 和高洁净实验室质感。"
  mood: "冷峻、紧张、清洁、战斗"
  world_elements: "禁区、雪尘、枪械、战术服、实验室、数据终端、弹药、隔离门"
  colors: "冷白/浅灰作面板；深灰作结构；冰蓝作系统；黄色作战术提示；红色只作伤害/警报"
  materials: "冷白磨砂玻璃、枪灰金属、冰蓝 HUD、橡胶装备"
  shape_language: "硬边圆角混合、切角、数据线、装备插槽"
  background_visual_language: "冷白实验室或雪域禁区远景，中心不放高亮角色"
  ui_translation:
    panel: "冷白半透明卡+灰色硬边，局部冰蓝线条"
    panel_base_style_hint: "冷白磨砂玻璃或枪灰金属卡底，硬边圆角混合、冰蓝数据线和洁净装备槽感；可联想隔离封条边、弹匣槽片、雪粒淡边，但枪械、角色战术服、准星和实验室门体不作为基础底板来源"
    state: "selected 用冰蓝描边；warning 用红色警报；locked 用隔离封条"
    text: "标题窄体科技感，正文深灰/白高可读"
    accent: "弹匣、十字准星、隔离线、雪粒、芯片、装备槽。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用蓝白科幻医疗；需保留枪械战术和禁区冷感"
```

## 24. 球球旅行记

```yaml
style_brief:
  name: "球球旅行记"
  aliases: []
  tags: ["cooking", "time-management", "travel", "airplane"]
  source:
    checked: "https://www.taptap.cn/app/726060"
    confidence: "high"
  game_features:
    type: "烹饪 / 时间管理 / 旅行"
    core_gameplay: "飞机厨房服务、乘客餐食、环球城市关卡和设备升级"
  visual_identity: "环球航班烹饪时间管理"
  genre_presentation: "把烹饪经营视觉化为空中厨房、托盘、城市旅行、快节奏服务和食物满足感。"
  mood: "明快、忙碌、治愈、旅行感"
  world_elements: "飞机舷窗、餐车、托盘、汉堡牛排、城市明信片、登机牌、厨房设备"
  colors: "天空蓝/云白作底；餐食暖黄/橙作奖励；深蓝作航空信息；绿色作完成；红色仅作超时"
  materials: "干净塑料托盘、航空蓝玻璃、菜单纸卡、不锈钢厨房面"
  shape_language: "圆角卡、登机牌切角、托盘格、清爽图标"
  background_visual_language: "云层和机舱厨房远景，边缘放食物/城市明信片，中心清洁"
  ui_translation:
    panel: "白蓝圆角卡+登机牌标签，食物图标小而干净"
    panel_base_style_hint: "云白/航空蓝圆角纸卡或干净塑料托盘卡底，登机牌切角、清爽蓝边和托盘格感；可联想明信片边、云朵贴边、餐盘细线，但食物图标、飞机舷窗、餐车和城市贴纸主体不作为基础底板来源"
    state: "selected 用蓝色登机牌边或金色餐盘；warning 用红色计时器"
    text: "标题亲切圆体，正文深蓝灰"
    accent: "登机牌、云朵、餐盘、城市贴纸、餐车、计时器。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通餐厅经营，必须保留航班和旅行语义"
```

## 25. 绝区零

```yaml
style_brief:
  name: "绝区零"
  aliases: []
  tags: ["urban", "street", "action", "vhs"]
  source:
    checked: "https://www.taptap.cn/app/234493"
    confidence: "high"
  game_features:
    type: "都市动作 / 二次元 / 箱庭"
    core_gameplay: "代理人战斗、录像店、空洞灾害、街区生活与潮流文化"
  visual_identity: "潮流街区录像带式都市动作"
  genre_presentation: "把动作游戏视觉化为街头文化、录像带故障、黑黄警示、贴纸招牌和高节奏打击感。"
  mood: "酷、躁动、幽默、街头"
  world_elements: "录像店、空洞、代理人、邦布、警示条、街头贴纸、CRT 噪声"
  colors: "黑/白/灰作强对比；黄色作核心强调；红色作危险；青粉作为潮流点缀；避免大面积柔粉"
  materials: "粗糙塑料、贴纸胶带、黑黄警示胶、CRT 玻璃、混凝土墙"
  shape_language: "厚块面、粗描边、切角标签、斜贴纸、错位排版"
  background_visual_language: "街区墙面/录像店远景，边缘贴纸和警示条，中心控制对比"
  ui_translation:
    panel: "厚卡+黑白底+黄色标签，贴纸层在 overlay 上方"
    panel_base_style_hint: "黑白厚卡或粗糙塑料卡底，切角标签、粗描边和黑黄警示胶带感；可联想 VHS 边条、街头贴纸角、故障块，但邦布图标、录像店招牌、涂鸦图形和危险条主体不作为基础底板来源"
    state: "selected 用黄色粗边/标签；warning 用红黑危险条；disabled 用灰化故障"
    text: "标题可粗体错位，正文必须黑白高对比"
    accent: "邦布图标、VHS、警示胶带、涂鸦、街牌、故障条。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成赛博朋克霓虹；核心是街头录像带与黑黄警示"
```

## 26. 火影忍者

```yaml
style_brief:
  name: "火影忍者"
  aliases: []
  tags: ["ninja", "anime", "fighting", "scroll"]
  source:
    checked: "https://www.taptap.cn/app/2247"
    confidence: "high"
  game_features:
    type: "横版格斗 / 动漫 IP"
    core_gameplay: "忍者角色、忍术连招、奥义大招、2V2 和竞技格斗"
  visual_identity: "热血忍者卷轴格斗"
  genre_presentation: "把格斗视觉化为忍术、卷轴、查克拉、木叶标识和高速连击的动漫战斗气势。"
  mood: "热血、燃、速度、羁绊"
  world_elements: "木叶、护额、手里剑、卷轴、查克拉、苦无、忍术印、火之意志"
  colors: "橙红作热血/火；黑灰作忍具；米黄作卷轴；蓝色作查克拉；红色用于危险和火系强调"
  materials: "卷轴纸、忍具金属、木牌、布料护额、查克拉能量"
  shape_language: "斜切速度线、卷轴边、粗描边、忍具角标"
  background_visual_language: "忍村屋檐/卷轴纹理远景，中心清爽不放角色战斗"
  ui_translation:
    panel: "卷轴卡+金属护额条，大卡可加查克拉速度线"
    panel_base_style_hint: "米黄卷轴纸或布料护额卡底，斜切速度线、忍具金属条和粗劲动漫描边；可联想卷轴轴头、护额金属片、疾风线，但木叶标识、手里剑、苦无和忍术印主体不作为基础底板来源"
    state: "selected 用蓝色查克拉或橙色火光；warning 用红色爆符"
    text: "标题可粗劲动漫感，正文深棕/白高可读"
    accent: "手里剑、苦无、护额、卷轴、忍术印、火焰、疾风线。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成泛日式武士或普通动漫热血；必须保留忍术和卷轴"
```

## 27. 龙族：卡塞尔之门

```yaml
style_brief:
  name: "龙族：卡塞尔之门"
  aliases: []
  tags: ["urban-fantasy", "dragon", "academy", "strategy-rpg"]
  source:
    checked: "https://www.taptap.cn/app/382099"
    confidence: "high"
  game_features:
    type: "策略 RPG / 都市幻想 / IP"
    core_gameplay: "卡塞尔学院、混血种、屠龙冒险、言灵与角色羁绊"
  visual_identity: "黑金学院派屠龙都市幻想"
  genre_presentation: "把策略 RPG 视觉化为精英学院、龙血秘仪、言灵档案、黑金校徽和青春宿命感。"
  mood: "热血、忧郁、神秘、学院感"
  world_elements: "卡塞尔学院、龙血、言灵、校徽、长剑、档案、论坛、混血种"
  colors: "黑/深棕作底；金色作学院荣耀；暗红作龙血/危机；冷蓝作言灵；米白作档案"
  materials: "黑金金属、皮革档案、学院徽章、暗色玻璃、羊皮纸"
  shape_language: "学院徽章、细金边、暗色卡、锐角装饰"
  background_visual_language: "暗色学院走廊/档案馆远景，中心保持阅读区"
  ui_translation:
    panel: "暗色档案卡+金边徽章，大卡可加封蜡/龙鳞纹"
    panel_base_style_hint: "黑棕暗色玻璃或皮革档案卡底，细金边、锐角学院轮廓和轻龙鳞压纹感；可联想封蜡边、档案夹角、校徽轮廓片，但校徽主体、龙血、长剑、论坛标签和角色符号不作为基础底板来源"
    state: "selected 用金边校徽；warning 用暗红龙血封条"
    text: "标题学院 serif/黑体混合，正文米白/浅灰清晰"
    accent: "校徽、龙鳞、长剑、档案夹、封蜡、论坛标签。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通西幻屠龙或青春校园；需要都市学院和龙族宿命"
```

## 28. TapAim

```yaml
style_brief:
  name: "TapAim"
  aliases: []
  tags: ["fps-training", "utility", "aim", "hud"]
  source:
    checked: "https://www.taptap.cn/app/815993"
    confidence: "high"
  game_features:
    type: "FPS 训练工具 / 靶场"
    core_gameplay: "移动端 FPS 靶场、练枪、反应和天赋测试"
  visual_identity: "专业轻量移动 FPS 靶场工具"
  genre_presentation: "把训练工具视觉化为干净靶场、数据指标、准星、命中率和性能测试界面。"
  mood: "专业、清晰、效率、训练感"
  world_elements: "准星、靶球、命中率、反应时间、灵敏度、排行榜、训练模块"
  colors: "深灰/白作底；蓝色作训练系统；绿色作通过/提升；橙色作注意；红色只作失误"
  materials: "深浅 HUD 面板、磨砂玻璃、靶场墙面、数据卡"
  shape_language: "极简硬边、圆形靶点、网格、进度条"
  background_visual_language: "低对比靶场网格或训练室墙面，中心安全区留白"
  ui_translation:
    panel: "清爽数据卡+靶心 icon，避免战场脏污纹理"
    panel_base_style_hint: "深浅灰数据卡或磨砂 HUD 卡底，极简硬边、靶场网格和蓝色训练线框；可联想准星环片、计时刻度、数据柱短线，但靶心图标、触控点、排行榜和具体训练模块不作为基础底板来源"
    state: "selected 用蓝色准星框；warning 用橙/红失误标识"
    text: "标题科技无衬线，数字大且清晰"
    accent: "准星、靶心、计时器、数据柱、触控点、陀螺仪图标。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成军事战场；它更像训练工具而非战斗场景"
```

## 29. 江南百景图

```yaml
style_brief:
  name: "江南百景图"
  aliases: []
  tags: ["jiangnan", "ink", "city-builder", "ancient-china"]
  source:
    checked: "https://www.taptap.cn/app/179371"
    confidence: "high"
  game_features:
    type: "模拟经营 / 古风建造"
    core_gameplay: "古代江南城市营造、居民、生产与布局"
  visual_identity: "雅致江南水墨市井营造"
  genre_presentation: "把模拟经营视觉化为江南画卷、白墙黛瓦、水路桥巷、印章和古籍账册。"
  mood: "雅致、烟火气、古朴、悠然"
  world_elements: "江南水乡、白墙黛瓦、石桥、船、灯笼、账册、印章、亭台"
  colors: "米白/宣纸黄作面板；墨黑作文字；黛青/水绿作环境；朱砂作印章和重点；金色少量作奖励"
  materials: "宣纸、青砖、木牌、古籍账本、印泥、淡墨"
  shape_language: "卷轴边、细墨线、方形印章、屋檐角、留白"
  background_visual_language: "淡墨江南远景，边缘白墙黛瓦/水波，中心干净宣纸"
  ui_translation:
    panel: "宣纸面板+淡墨边+朱砂印，小卡简洁"
    panel_base_style_hint: "米白宣纸或古籍账本卡底，淡墨细边、卷轴边和朱砂印泥质感；可联想瓦片线、桥纹淡边、印章痕，但江南建筑、船、灯笼、铜钱和折扇主体不作为基础底板来源"
    state: "selected 用朱砂印章；warning 用暗红告示；locked 用木牌锁"
    text: "标题可古风但正文深墨清晰"
    accent: "印章、折扇、船桨、灯笼、瓦片、铜钱、桥纹。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成宫廷金红或仙侠水墨；需保留市井经营和江南雅致"
```

## 30. 名将杀

```yaml
style_brief:
  name: "名将杀"
  aliases: []
  tags: ["card", "history", "strategy", "tabletop"]
  source:
    checked: "https://www.taptap.cn/app/737825"
    confidence: "high"
  game_features:
    type: "卡牌 / 历史 / 桌游策略"
    core_gameplay: "经典杀牌身份策略、战国至秦汉名将策士与多人博弈"
  visual_identity: "战国秦汉谋略杀牌桌游"
  genre_presentation: "把卡牌策略视觉化为竹简、兵符、青铜器、战旗和历史谋略牌局。"
  mood: "谋略、古朴、竞技、烽火"
  world_elements: "名将、策士、竹简、兵符、战旗、杀牌、古战场、秦汉纹样"
  colors: "竹简黄/暗木棕作底；青铜绿作边框；朱砂红作杀/攻击；金色作稀有/胜利；黑墨作文字"
  materials: "竹简、羊皮纸、青铜、暗木桌面、印章"
  shape_language: "矩形牌框、青铜浮雕、竹简横线、旗帜角标"
  background_visual_language: "古战场或木桌牌局边缘，中心留出纸面安全区"
  ui_translation:
    panel: "卡牌式面板+青铜边，普通小卡减少浮雕"
    panel_base_style_hint: "竹简黄/暗木棕卡牌底，青铜细边、矩形牌框和竹简横纹感；可联想兵符边、战旗布角、朱砂印痕，但名将人像、剑戈、杀牌字样和古战场场景不作为基础底板来源"
    state: "selected 用金色牌框或朱砂印；warning 用红色杀牌，不作普通选中"
    text: "标题古朴硬朗，正文深墨清楚"
    accent: "兵符、战旗、印章、剑戈、竹简、火漆。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成三国武将立绘手游；要保留桌游卡牌和秦汉谋略"
```

## 31. 再玩亿关

```yaml
style_brief:
  name: "再玩亿关"
  aliases: []
  tags: ["casual", "mini-games", "satisfying", "cute"]
  source:
    checked: "https://www.taptap.cn/app/247977"
    confidence: "high"
  game_features:
    type: "休闲解压 / 迷你游戏合集"
    core_gameplay: "多类型解压、拼图、收纳、体育、益智和迷你关卡合集"
  visual_identity: "清新可爱解压迷你游戏合集"
  genre_presentation: "把休闲合集视觉化为柔和卡通、满足感小物、收纳格、拼图块和轻量挑战。"
  mood: "轻松、满足、治愈、亲民"
  world_elements: "拼图、收纳盒、按钮机关、小球、清洁、体育小物、关卡卡片"
  colors: "浅米白/浅蓝作底；马卡龙粉绿黄作分类；橙黄作完成奖励；红色仅作失败"
  materials: "柔软塑料、纸卡、橡皮泥、浅色收纳盒"
  shape_language: "圆润块面、卡片网格、拼图边、软按钮"
  background_visual_language: "浅色桌面/收纳盒远景，中央清爽"
  ui_translation:
    panel: "白色圆角卡+马卡龙分类标签，小卡极简"
    panel_base_style_hint: "浅米白/浅蓝软塑料或纸卡底，圆润块面、马卡龙分类边和轻收纳格感；可联想拼图齿边、小星贴片、勾选贴纸角，但手指提示、清洁刷、体育小物和具体关卡物件不作为基础底板来源"
    state: "selected 用黄色星星/勾选；warning 用小红叉"
    text: "标题可圆润亲切，正文深灰"
    accent: "拼图块、收纳格、小星星、手指提示、清洁刷、勾选贴纸。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成幼儿教育或单一三消；需保留多玩法合集和解压感"
```

## 32. 遗弃之地

```yaml
style_brief:
  name: "遗弃之地"
  aliases: []
  tags: ["folk-horror", "shadow-puppet", "tower-defense", "chinese"]
  source:
    checked: "https://www.taptap.cn/app/753196"
    confidence: "high"
  game_features:
    type: "中式民俗恐怖 / 横版策略塔防"
    core_gameplay: "修法者、符咒天书、阴阳两界、随机法术和横版策略防守"
  visual_identity: "黑白皮影中式民俗轻恐怖塔防"
  genre_presentation: "把策略塔防视觉化为黑白剪影、皮影傀儡、符咒、阴阳界和低饱和恐怖民俗。"
  mood: "阴冷、诡异、克制、玄秘"
  world_elements: "皮影、符咒、天书、阴阳、修法者、纸人、傀儡线、法术手牌"
  colors: "黑白灰作主视觉；旧纸黄作面板；朱砂红作法力/危险；暗青作阴气；避免鲜艳彩色"
  materials: "旧纸符、黑墨剪影、朱砂印、木刻皮影、暗色纸屏"
  shape_language: "剪影边、符纸条、破损纸边、横版层次、细红线"
  background_visual_language: "黑白纸屏/村落剪影远景，中央旧纸低纹理安全区"
  ui_translation:
    panel: "旧纸符面板+黑墨边，大卡可加皮影剪影，小卡克制"
    panel_base_style_hint: "旧纸符或暗色纸屏卡底，黑墨剪影边、破损纸边和朱砂红线质感；可联想符纸纤维、皮影刻边、铜钱印痕，但纸人、傀儡、香炉和法术手牌不作为基础底板来源"
    state: "selected 用朱砂印/红线；warning 用暗红符火；disabled 用灰白褪色"
    text: "标题可篆刻/民俗感，正文黑墨清晰"
    accent: "符箓、纸人、傀儡线、铜钱、香炉、朱砂印。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成血腥恐怖或普通水墨仙侠；应是民俗轻恐怖+皮影"
```

## 33. 吉星派对

```yaml
style_brief:
  name: "吉星派对"
  aliases: []
  tags: ["party", "anime", "cards", "chaos"]
  source:
    checked: "https://www.taptap.cn/app/524022"
    confidence: "high"
  game_features:
    type: "联机派对 / 卡牌 / 二次元"
    core_gameplay: "4 人派对、角色技能、手牌攻击、随机事件和友尽博弈"
  visual_identity: "恶搞卡通二次元友尽派对"
  genre_presentation: "把多人派对视觉化为卡牌陷阱、随机事件、夸张表情和明亮可爱的恶搞舞台。"
  mood: "欢乐、混乱、恶搞、竞技"
  world_elements: "4人棋盘、角色技能牌、随机事件、骰子、表情、恶作剧道具"
  colors: "亮蓝/粉/黄作派对底；白色作信息卡；紫色作事件；红色只作攻击/惩罚"
  materials: "亮面纸牌、塑料棋子、贴纸、舞台灯牌"
  shape_language: "圆润卡牌、厚描边、夸张表情框、彩色标签"
  background_visual_language: "派对棋盘或舞台远景，边缘放卡牌/骰子，中心清爽"
  ui_translation:
    panel: "纸牌面板+彩色标签，小卡可像手牌"
    panel_base_style_hint: "亮面纸牌或白底派对卡底，圆润厚描边、彩色标签边和轻舞台灯牌感；可联想骰子点边、手牌页角、表情贴纸边，但角色技能牌、炸弹玩具、随机事件图标和表情主体不作为基础底板来源"
    state: "selected 用黄色聚光灯/卡牌边；warning 用红色恶作剧标"
    text: "标题活泼粗体，正文高对比"
    accent: "骰子、卡牌、炸弹玩具、星星、表情贴纸、舞台灯。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通美少女卡牌或低龄派对；需保留友尽随机事件"
```

## 34. 铃兰之剑：为这和平的世界

```yaml
style_brief:
  name: "铃兰之剑：为这和平的世界"
  aliases: []
  tags: ["tactics-rpg", "medieval", "retro", "hd2d"]
  source:
    checked: "https://www.taptap.cn/app/209866"
    confidence: "high"
  game_features:
    type: "战棋 RPG / 中世纪幻想"
    core_gameplay: "像素/HD-2D 风格战棋、佣兵团、国家冲突与叙事选择"
  visual_identity: "温厚复古中世纪战棋史诗"
  genre_presentation: "把战棋 RPG 视觉化为羊皮地图、佣兵旗帜、石质城镇、古典战术棋盘与怀旧像素质感。"
  mood: "史诗、温厚、策略、怀旧"
  world_elements: "佣兵团、棋盘格、羊皮地图、旗帜、长剑、石墙、酒馆、命运选择"
  colors: "羊皮黄作面板；棕/石灰作结构；深蓝/红作阵营；金色作奖励；红色用于敌对和危险"
  materials: "羊皮纸、旧木、石板、金属徽章、布旗"
  shape_language: "厚实纸卡、棋盘格、旗帜角、像素细节"
  background_visual_language: "羊皮地图/石城远景，中心低纹理"
  ui_translation:
    panel: "羊皮纸面板+旧木边，大卡可加棋盘格淡纹"
    panel_base_style_hint: "羊皮地图纸或旧木边战棋卡底，厚实纸卡、棋盘淡纹和复古像素细节；可联想佣兵旗角、地图针边、金属徽章片，但长剑、酒馆杯、命运标识和角色像素形象不作为基础底板来源"
    state: "selected 用金色棋格边；warning 用红色敌军旗"
    text: "标题古典，正文深棕清晰"
    accent: "旗帜、剑盾、地图针、酒馆杯、像素小徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成日系轻幻想或暗黑西幻；需保留复古战棋和温厚叙事"
```

## 35. 无限升级

```yaml
style_brief:
  name: "无限升级"
  aliases: []
  tags: ["idle-rpg", "progression", "loot", "equipment"]
  source:
    checked: "https://www.taptap.cn/app/241449"
    confidence: "high"
  game_features:
    type: "放置 RPG / 数值养成"
    core_gameplay: "传统 RPG 养成、刷怪爆装、装备强化附魔、转生和御灵养成"
  visual_identity: "传统数值放置 RPG 爆装成长"
  genre_presentation: "把放置 RPG 视觉化为装备栏、属性面板、强化石、转生印记和持续升级的数字爽感。"
  mood: "爽快、直给、成长、收集"
  world_elements: "装备、宝石、神器、御灵、转生、技能、强化、属性数值"
  colors: "深蓝黑/暗棕作底；金色作品质和提升；紫色作神器；绿色作成长；红色只作战斗伤害"
  materials: "暗色面板、金属装备槽、宝石按钮、卷轴属性卡"
  shape_language: "矩形装备槽、硬边、等级标签、进度条、数值高亮"
  background_visual_language: "暗色装备仓/属性页远景，中央留出数值安全区"
  ui_translation:
    panel: "深色装备卡+金属边，小卡像装备格"
    panel_base_style_hint: "深蓝黑/暗棕装备卡底，金属装备槽边、硬边矩形和宝石品质光泽；可联想等级牌边、强化箭头短线、转生印纹，但装备图标、技能书、御灵和大块数值不作为基础底板来源"
    state: "selected 用金色品质框；warning 用红色伤害/失败强化"
    text: "标题硬朗，数字要大且清楚"
    accent: "装备槽、宝石、箭头上升、等级牌、转生印、技能书。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用魔幻 RPG；要保留纯数值成长和装备爆装"
```

## 36. 重返未来：1999

```yaml
style_brief:
  name: "重返未来：1999"
  aliases: []
  tags: ["retro", "occult", "1999", "strategy-rpg"]
  source:
    checked: "https://www.taptap.cn/app/221062"
    confidence: "high"
  game_features:
    type: "策略 RPG / 神秘学 / 复古"
    core_gameplay: "时间逆流、神秘学家、20 世纪文化切片与卡牌战斗"
  visual_identity: "英伦复古神秘学时间旅行 RPG"
  genre_presentation: "把策略 RPG 视觉化为旧报纸、胶片、唱片、雨夜、档案与超现实神秘符号。"
  mood: "怀旧、神秘、优雅、忧郁"
  world_elements: "1999、暴雨、旧报纸、神秘学、唱片、胶片、手提箱、时钟"
  colors: "旧纸黄/褐色作面板；墨黑作文字；酒红作重点；黄铜金作高级；冷绿/蓝作神秘"
  materials: "旧纸、打字稿、皮革箱、黄铜、胶片、磨砂玻璃"
  shape_language: "复古矩形、邮票齿边、档案夹、黄铜圆角"
  background_visual_language: "雨夜窗边/旧档案室远景，中心纸面清爽"
  ui_translation:
    panel: "旧纸档案卡+黄铜细边，大卡可加胶片/邮票"
    panel_base_style_hint: "旧纸打字稿或皮革档案卡底，黄铜细边、邮票齿边和复古矩形轮廓；可联想胶片孔边、雨滴痕、时钟刻度，但唱片、手提箱、神秘学符号和 1999 字样不作为基础底板来源"
    state: "selected 用酒红邮戳或黄铜框；warning 用暗红异常标"
    text: "标题复古 serif，正文黑色打字稿清晰"
    accent: "邮票、胶片、唱片、时钟、雨滴、打字机字条。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通蒸汽朋克或欧式宫廷；需保留 20 世纪现代复古与神秘学"
```

## 37. 瞬搭

```yaml
style_brief:
  name: "瞬搭"
  aliases: []
  tags: ["fashion", "social", "dress-up", "modern"]
  source:
    checked: "https://www.taptap.cn/app/712212"
    confidence: "high"
  game_features:
    type: "时尚换装 / 社交 / 生活"
    core_gameplay: "自由搭配、捏脸、社区发帖、穿搭博主与多风格服饰收集"
  visual_identity: "全球潮流时尚社交换装"
  genre_presentation: "把换装视觉化为手机社媒、杂志版式、穿搭卡、抽奖礼盒和多元审美。"
  mood: "潮流、自我表达、精致、社交"
  world_elements: "SU市、穿搭博主、衣橱、社区帖子、搭配挑战、抽奖券、时尚杂志"
  colors: "白/浅灰作平台底；黑色作时尚对比；粉紫/湖蓝作活动；金色作稀有；红色仅作限时提示"
  materials: "白色手机卡、磨砂玻璃、杂志纸、亮面贴纸、金属小扣"
  shape_language: "现代圆角、瀑布流卡片、标签化、轻薄层叠"
  background_visual_language: "柔焦城市/衣橱/社交 feed 边缘，中心干净"
  ui_translation:
    panel: "手机 feed 风圆角卡+时尚标签，小卡像穿搭帖子"
    panel_base_style_hint: "白/浅灰手机 feed 卡或杂志纸卡底，现代圆角、轻薄层叠和黑白时尚线框；可联想标签边、抽奖券角、亮面贴纸，但衣架、相机、点赞图标和穿搭内容不作为基础底板来源"
    state: "selected 用黑白高对比或金色细边；warning 用红色限时角标"
    text: "标题现代时尚，正文清爽无衬线"
    accent: "衣架、相机、点赞、标签、抽奖券、闪光、杂志贴纸。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成传统少女粉或单一换装；需保留潮流社交和多元审美"
```

## 38. 梦幻消除战

```yaml
style_brief:
  name: "梦幻消除战"
  aliases: []
  tags: ["merge", "teahouse", "chinese", "cozy"]
  source:
    checked: "https://www.taptap.cn/app/747820"
    confidence: "high"
  game_features:
    type: "合成 / 模拟经营 / 古风茶楼"
    core_gameplay: "合成茶点、茶楼经营、家具装修、商海与情缘叙事"
  visual_identity: "国风茶点合成茶楼经营"
  genre_presentation: "把合成经营视觉化为中式茶点、华庭茶楼、糕点盒、木格柜和烟火人间。"
  mood: "温润、甜美、市井、经营感"
  world_elements: "茶楼、芙蓉糕、玉兔点心、茶叶、糕点盒、家具、自宅、商铺"
  colors: "米黄/茶白作面板；木棕作边框；茶绿作辅助；桃粉/糕点色作奖励；朱红作印章/限时"
  materials: "宣纸菜单、木质柜台、瓷盘、糕点盒、绸布、铜钱"
  shape_language: "圆润木格、糕点盒格子、卷草角花、柔和卡牌"
  background_visual_language: "茶楼室内/庭院边缘，中央浅纸面留白"
  ui_translation:
    panel: "菜单式纸卡+木格边，大卡可加瓷盘/糕点盒"
    panel_base_style_hint: "米黄/茶白菜单纸卡或木格茶楼卡底，圆润木格边、柔和卡牌轮廓和瓷盘光泽；可联想糕点盒格边、茶盏线、花窗淡边，但茶点主体、铜钱、灯笼和家具不作为基础底板来源"
    state: "selected 用茶绿/金色细边；warning 用朱红限时牌"
    text: "标题国风温润，正文深棕清晰"
    accent: "茶盏、点心、木格、铜钱、灯笼、菜单牌、花窗。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通三消糖果或宫廷经营；需保留茶点文化和茶楼装修"
```

## 39. 一念逍遥

```yaml
style_brief:
  name: "一念逍遥"
  aliases: []
  tags: ["xianxia", "idle", "ink", "cultivation"]
  source:
    checked: "https://www.taptap.cn/app/159465"
    confidence: "high"
  game_features:
    type: "修仙放置 / 水墨 RPG"
    core_gameplay: "放置修炼、炼丹炼器、飞升、宗门和仙魔抉择"
  visual_identity: "水墨国风放置修仙"
  genre_presentation: "把放置修仙视觉化为云海、洞府、丹炉、灵气、仙鹤和境界突破。"
  mood: "飘逸、悠远、修行、清静"
  world_elements: "洞府、丹炉、飞升、宗门、灵兽、云海、法宝、境界"
  colors: "米白/淡墨作面板；青绿作灵气；淡金作仙缘；朱砂作印章；暗紫/红用于魔修/劫难"
  materials: "宣纸、水墨、玉石、青铜丹炉、淡金云纹"
  shape_language: "卷轴、云纹圆角、法阵圆环、印章"
  background_visual_language: "水墨云海山峦远景，中央宣纸留白"
  ui_translation:
    panel: "宣纸卡+淡墨边+云纹角，大卡可加法阵淡纹"
    panel_base_style_hint: "米白宣纸或淡墨修行卡底，云纹圆角、淡金/青绿细边和水墨呼吸感；可联想法阵弧线、玉佩边、朱砂印痕，但仙鹤、丹炉、法宝、符箓主体不作为基础底板来源"
    state: "selected 用淡金/青绿灵气；warning 用暗红天劫符"
    text: "标题书法感，正文深墨清楚"
    accent: "仙鹤、丹炉、符箓、法宝、云纹、玉佩。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成冷蓝仙侠或硬核武侠；应是放置修行的淡雅水墨"
```

## 40. 恋与深空

```yaml
style_brief:
  name: "恋与深空"
  aliases: []
  tags: ["romance", "sci-fi", "deep-space", "otome"]
  source:
    checked: "https://www.taptap.cn/app/201633"
    confidence: "high"
  game_features:
    type: "女性向 / 3D 恋爱 / 科幻"
    core_gameplay: "近未来恋爱、战斗、陪伴、深空设定和高沉浸角色互动"
  visual_identity: "近未来深空浪漫恋爱科幻"
  genre_presentation: "把恋爱互动视觉化为深空、星舰感、柔光玻璃、心率数据和亲密通信界面。"
  mood: "浪漫、沉浸、神秘、精致"
  world_elements: "深空、星轨、通讯、心率、约会、战斗搭档、记忆碎片、光粒"
  colors: "深蓝/银白作深空底；粉紫作浪漫；冰蓝作科技；金色作珍贵；红色只作心率/警告"
  materials: "柔光玻璃、星空金属、丝绸渐变、半透明通信卡"
  shape_language: "圆角薄卡、光带、星轨弧线、轻奢细边"
  background_visual_language: "深空/城市夜景柔焦远景，中心纯净低纹理"
  ui_translation:
    panel: "半透明柔光卡+细边，背景粒子必须少"
    panel_base_style_hint: "深蓝/银白柔光玻璃或半透明通信卡底，圆角薄卡、星轨弧线和轻奢细边；可联想心跳线片段、记忆碎片边、柔粉光带，但角色、约会物件、通讯气泡图标和星舰场景不作为基础底板来源"
    state: "selected 用粉紫/金色柔光；warning 用红色心率警示"
    text: "标题优雅现代，正文白/深灰清晰"
    accent: "星轨、心跳线、通讯气泡、花、记忆碎片、光点。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成通用粉色乙女或硬科幻；需保留浪漫与深空并重"
```

## 41. 出发吧麦芬

```yaml
style_brief:
  name: "出发吧麦芬"
  aliases: []
  tags: ["idle-rpg", "cozy", "travel", "muffin"]
  source:
    checked: "https://www.taptap.cn/app/222034"
    confidence: "high"
  game_features:
    type: "放置 RPG / 治愈冒险"
    core_gameplay: "房车旅行、双人同行、放置成长、轻松社交与幻想冒险"
  visual_identity: "治愈房车异世界放置冒险"
  genre_presentation: "把放置 RPG 视觉化为房车旅途、伙伴同行、日式西幻、柔软云朵和轻松战斗。"
  mood: "治愈、轻松、友爱、旅行"
  world_elements: "房车、麦芬、营地、伙伴、云朵、地图、技能、冒险行囊"
  colors: "暖白/奶油黄作底；草绿/天蓝作旅途；木棕作结构；金色作奖励；橙色作技能提示"
  materials: "浅木、帆布、旅行手账、圆角纸卡、软糖色徽章"
  shape_language: "圆润、手账标签、地图虚线、软木边框"
  background_visual_language: "晴朗旅途/营地远景，中央云白留白"
  ui_translation:
    panel: "手账纸卡+浅木边，大卡可加地图虚线"
    panel_base_style_hint: "暖白/奶油黄手账纸卡或帆布旅行卡底，浅木边、圆润标签和地图虚线感；可联想背包缝线、地图针边、云朵纸片，但房车、篝火、麦芬和伙伴形象不作为基础底板来源"
    state: "selected 用金色地图针/云朵边；warning 用橙色提示牌"
    text: "标题圆润冒险感，正文深棕清晰"
    accent: "房车、背包、地图针、云朵、篝火、面包/麦芬。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成硬核异世界或低龄宠物；需保留治愈房车旅行"
```

## 42. 金铲铲之战

```yaml
style_brief:
  name: "金铲铲之战"
  aliases: []
  tags: ["autobattler", "strategy", "board", "competitive"]
  source:
    checked: "https://www.taptap.cn/app/176937"
    confidence: "high"
  game_features:
    type: "自走棋 / 策略 / 英雄"
    core_gameplay: "棋盘布阵、羁绊、装备合成、小小英雄和赛季主题"
  visual_identity: "华丽棋盘自走棋策略竞技"
  genre_presentation: "把自走棋视觉化为战术棋盘、装备合成、羁绊徽章、金铲铲和小小英雄收藏。"
  mood: "策略、竞技、收集、轻幻想"
  world_elements: "棋盘、金铲铲、装备、羁绊、商店、金币、小小英雄、传送门"
  colors: "深蓝/紫作棋盘底；金色作金币和选中；宝石色作羁绊；绿色作经济；红色只作失败/敌方"
  materials: "深色棋盘石板、金属边、宝石徽章、商店卡"
  shape_language: "棋格、徽章、金边、装备槽、六边形/方格"
  background_visual_language: "棋盘/竞技场远景，边缘装备和金币，中心安全区低纹理"
  ui_translation:
    panel: "棋盘纹理卡+金边，大卡可加羁绊徽章"
    panel_base_style_hint: "深蓝/紫棋盘石板或商店卡底，金属金边、棋格纹和装备槽轮廓；可联想金币压纹、羁绊徽章边、六边形格片，但金铲铲、装备图标、小小英雄和传送门不作为基础底板来源"
    state: "selected 用金色棋格框；warning 用红色连败/敌方标"
    text: "标题竞技华丽，正文白/金高对比"
    accent: "金铲铲、金币、装备、小小英雄脚印、羁绊徽章。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通 MOBA 或卡牌；核心是棋盘布阵和经济策略"
```

## 43. 光·遇

```yaml
style_brief:
  name: "光·遇"
  aliases: []
  tags: ["poetic", "social", "cloud", "light"]
  source:
    checked: "https://www.taptap.cn/app/62448"
    confidence: "high"
  game_features:
    type: "社交冒险 / 探索"
    core_gameplay: "飞行、烛火、牵手社交、云中王国探索"
  visual_identity: "诗意云海光旅社交冒险"
  genre_presentation: "把社交冒险视觉化为光、风、云海、斗篷、烛火和极简精神性空间。"
  mood: "宁静、温柔、诗意、治愈"
  world_elements: "云海、烛火、光翼、斗篷、牵手、神庙、风、星光"
  colors: "暖白/云灰作底；金色作光和奖励；天蓝/霞粉作空间；深蓝作夜；红色极少使用"
  materials: "柔光玻璃、雾面纸、云层渐变、金色光尘"
  shape_language: "大留白、圆角、轻薄卡、柔和弧线"
  background_visual_language: "云海/霞光远景，中央空灵留白，避免强光中心"
  ui_translation:
    panel: "极简半透明卡+柔光边，装饰极少"
    panel_base_style_hint: "暖白/云灰柔光玻璃或雾面纸卡底，轻薄圆角、柔和弧线和金色光尘感；可联想烛火光边、风线、星点，但斗篷剪影、光翼、神庙和社交手势不作为基础底板来源"
    state: "selected 用金色烛光环；warning 几乎不用红，必要时用柔橙"
    text: "标题轻盈，正文高对比但不锐利"
    accent: "烛火、光翼、星星、云朵、斗篷剪影、风线。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通天空治愈或儿童绘本；需保留社交与光的仪式感"
```

## 44. 奥比岛：梦想国度

```yaml
style_brief:
  name: "奥比岛：梦想国度"
  aliases: []
  tags: ["cute", "island", "social-sim", "fairy-tale"]
  source:
    checked: "https://www.taptap.cn/app/88544"
    confidence: "high"
  game_features:
    type: "社交模拟 / 岛屿生活"
    core_gameplay: "小岛生活、家园装扮、社交、宠物和童话活动"
  visual_identity: "童话玩偶岛屿社交模拟"
  genre_presentation: "把岛屿生活视觉化为可爱小屋、童话森林、玩偶质感、梦幻活动和强社交装扮。"
  mood: "可爱、梦幻、童年、热闹"
  world_elements: "奥比小岛、小屋、家具、精灵、气球、花园、宠物、派对"
  colors: "粉蓝/奶黄作底；草绿作岛屿；紫粉作梦幻；金色作奖励；红色仅限警示"
  materials: "棉花糖色塑料、木牌、布艺、彩纸、亮面贴纸"
  shape_language: "圆润、花边、玩偶厚块、云朵边、糖果按钮"
  background_visual_language: "童话岛屿/花园边缘，中心浅色安全区"
  ui_translation:
    panel: "软糖色圆角卡+花边，小卡保持干净"
    panel_base_style_hint: "粉蓝/奶黄软糖色塑料或彩纸卡底，圆润花边、玩偶厚块和云朵边感；可联想气球色块、花朵贴边、宠物脚印痕，但小屋、精灵、宠物、蝴蝶结和具体派对物件不作为基础底板来源"
    state: "selected 用金色星星/粉蓝描边；warning 用小红感叹"
    text: "标题圆润梦幻，正文深色高可读"
    accent: "气球、花朵、云朵、蝴蝶结、小屋、星星、宠物脚印。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成低龄儿童 UI；需做成童话社交而非幼教"
```

## 45. 苍翼：混沌效应

```yaml
style_brief:
  name: "苍翼：混沌效应"
  aliases: []
  tags: ["cyber", "action", "roguelite", "anime"]
  source:
    checked: "https://www.taptap.cn/app/233514"
    confidence: "high"
  game_features:
    type: "动作 Roguelite / 科幻动漫"
    core_gameplay: "高速动作、角色流派、意识空间、连招与 Roguelite 构筑"
  visual_identity: "电蓝赛博动漫动作 Roguelite"
  genre_presentation: "把动作 Roguelite 视觉化为高对比电光、代码故障、硬边 UI、角色数据和高速连击反馈。"
  mood: "凌厉、炫酷、高速、压迫"
  world_elements: "意识空间、代码、连击、技能芯片、机甲感、蓝色电弧、数据故障"
  colors: "黑/深蓝作底；电蓝作核心能量；紫色作混沌；白色作高亮；红色只作受击/危险"
  materials: "暗色玻璃、金属切片、电蓝能量、故障屏幕"
  shape_language: "锐角、斜切、碎片化、扫描线、能量条"
  background_visual_language: "深色数据空间远景，边缘电蓝碎片，中心可读"
  ui_translation:
    panel: "深色切角卡+电蓝边，特效控制在边缘"
    panel_base_style_hint: "黑/深蓝暗色玻璃或金属切片卡底，锐角斜切、电蓝能量边和轻故障扫描感；可联想芯片边、故障块、连击数字轮廓，但角色数据、技能晶片主体、电弧大特效和机甲物件不作为基础底板来源"
    state: "selected 用电蓝能量边；warning 用红色故障警示"
    text: "标题锋利科技感，正文白灰清晰"
    accent: "电弧、芯片、故障块、连击数字、技能晶片。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通赛博霓虹；需要格斗动漫速度和 Roguelite 芯片"
```

## 46. 泰拉瑞亚

```yaml
style_brief:
  name: "泰拉瑞亚"
  aliases: []
  tags: ["pixel", "sandbox", "side-scroller", "adventure"]
  source:
    checked: "https://www.taptap.cn/app/194610"
    confidence: "high"
  game_features:
    type: "像素沙盒 / 冒险 / 生存"
    core_gameplay: "横版挖掘、建造、探索、Boss 战和装备收集"
  visual_identity: "复古像素横版沙盒冒险"
  genre_presentation: "把沙盒冒险视觉化为像素地层、物品栏、矿洞、工作台和昼夜地表探索。"
  mood: "探索、自由、复古、惊喜"
  world_elements: "泥土层、矿石、洞穴、火把、木屋、工作台、史莱姆、Boss 战利品"
  colors: "泥棕/草绿作自然；深蓝作夜和洞穴；木棕作面板；金/紫作稀有；红色作危险"
  materials: "像素木板、石块、泥土、物品栏槽、复古蓝窗"
  shape_language: "像素直角、格子、块状边框、物品槽"
  background_visual_language: "像素地层/木屋/洞穴边缘，中心清晰块状安全区"
  ui_translation:
    panel: "像素木板/石砖面板，小卡像物品槽"
    panel_base_style_hint: "像素木板/石块或复古蓝窗卡底，直角块状边框、物品槽和地层拼接感；可联想矿石色块、火把暖边、星星小片，但镐子、史莱姆、Boss 战利品和洞穴场景不作为基础底板来源"
    state: "selected 用金色像素框；warning 用红色 Boss/伤害"
    text: "标题像素体，正文若小需改用清晰字体"
    accent: "火把、矿石、镐子、史莱姆、物品槽、星星。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成我的世界体素；它是横版像素冒险和地层挖掘"
```

## 47. 凹凸世界：彩虹收藏家

```yaml
style_brief:
  name: "凹凸世界：彩虹收藏家"
  aliases: []
  tags: ["match-3", "collection", "merch", "cyber"]
  source:
    checked: "https://www.taptap.cn/app/749006"
    confidence: "high"
  game_features:
    type: "三消 RPG / IP 收藏 / 赛博吃谷"
    core_gameplay: "三消冒险、参赛者信息与周边收藏、盲盒和 DIY 收藏板"
  visual_identity: "赛博吃谷三消收藏"
  genre_presentation: "把三消和 IP 收藏视觉化为虚拟谷子展柜、盲盒、彩虹膜工艺、元力战斗和几何 IP 视觉。"
  mood: "轻快、收藏癖、潮流、热血"
  world_elements: "谷子、盲盒、收藏图册、参赛者、元力、彩窗、反光、DIY 展板"
  colors: "白/浅灰作展柜底；彩虹膜高光作稀有；黑色/亮色几何作 IP 对比；金色作限定；红色少用"
  materials: "亮面亚克力、彩虹膜、白瓷、银葱、透明展柜、卡牌纸"
  shape_language: "几何切面、展示格、圆角盲盒、彩虹折射边"
  background_visual_language: "虚拟收藏展柜边缘，中央留出清爽展示区"
  ui_translation:
    panel: "展柜格卡+虹彩边，小卡像收藏品槽"
    panel_base_style_hint: "白/浅灰展柜卡或亮面亚克力卡底，几何切面、彩虹折射边和收藏格轮廓；可联想盲盒盒角、银葱反光、徽章压痕，但参赛者、元力符号、亚克力牌主体和收藏图册图形不作为基础底板来源"
    state: "selected 用彩虹膜高光；warning 用红色库存/限时小角标"
    text: "标题可潮流几何，正文黑灰清晰"
    accent: "盲盒、徽章、亚克力牌、彩虹反光、收藏格、元力符号。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通三消糖果或单纯二次元卡牌；需保留赛博吃谷和收藏工艺"
```

## 48. 前线模拟战

```yaml
style_brief:
  name: "前线模拟战"
  aliases: []
  tags: ["war-sim", "shooter", "tactical", "hud"]
  source:
    checked: "https://www.taptap.cn/app/753312"
    confidence: "high"
  game_features:
    type: "现代战争模拟 / 射击"
    core_gameplay: "AI 敌人、多模式、经典武器、真实弹道后坐力和团队战术对抗"
  visual_identity: "轻体量现代战争模拟靶场战术"
  genre_presentation: "把战争模拟视觉化为真实枪械、弹道参数、战术地图、背包仓库和低成本但清晰的军用 HUD。"
  mood: "紧张、实用、粗粝、战术"
  world_elements: "经典武器、弹药、护甲、矿洞撤离点、仓库、流浪商人、AI 敌人"
  colors: "军绿/灰黑作底；土黄作任务；白灰作参数；红色作受伤/危险；蓝色少量作系统"
  materials: "军绿终端、粗糙金属、橡胶、仓库格、纸质任务单"
  shape_language: "硬边、网格、装备槽、粗线框、切角"
  background_visual_language: "低对比仓库/战术地图/靶场边缘，中心清晰"
  ui_translation:
    panel: "硬边装备卡+仓库格，大卡可加任务纸"
    panel_base_style_hint: "军绿终端或粗糙金属装备卡底，硬边切角、仓库格和粗线框感；可联想任务单纸角、撤离箭头短线、护甲槽片，但枪械、弹匣、矿洞标记、AI 敌人和战场场景不作为基础底板来源"
    state: "selected 用土黄战术边；warning 用红色伤害/撤离倒计时"
    text: "标题硬朗，正文和数字清楚"
    accent: "弹匣、准星、护甲、撤离箭头、仓库锁、矿洞标记。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成 AAA 战争大片或赛博；应保持轻量模拟和实用工具感"
```

## 49. 境·界 刀鸣

```yaml
style_brief:
  name: "境·界 刀鸣"
  aliases: []
  tags: ["bleach", "sword", "anime-action", "soul"]
  source:
    checked: "https://www.taptap.cn/app/374914"
    confidence: "high"
  game_features:
    type: "动作 / 动漫 IP / 格斗"
    core_gameplay: "BLEACH 正版授权、剧情体验、拼刀对决、卍解、极速攻防和角色连携"
  visual_identity: "黑白魂界刀鸣热血动作"
  genre_presentation: "把动作手游视觉化为斩魄刀、灵压、黑白死神服、墨线斩击和卍解爆发。"
  mood: "热血、锋利、肃杀、浪漫"
  world_elements: "尸魂界、斩魄刀、卍解、灵压、守护、拼刀、黑崎一护式魂力"
  colors: "黑白作主色；蓝白作灵压；橙色作主角热血点；紫/红作敌方或危险；金色少量作稀有"
  materials: "黑白纸墨、刀刃金属、灵压光、暗色玻璃、和风纹"
  shape_language: "锐角斩击、墨刷边、切角卡、速度线"
  background_visual_language: "黑白魂界/夜空远景，边缘斩击线，中心干净"
  ui_translation:
    panel: "暗色切角卡+墨刷边+刀痕角标"
    panel_base_style_hint: "黑白纸墨或暗色玻璃切角卡底，墨刷边、锐角斩击轮廓和刀刃金属冷光；可联想斩击弧边、队章轮廓片、符纸纤维，但斩魄刀主体、卍解符号、角色标志和灵压火焰不作为基础底板来源"
    state: "selected 用蓝白灵压光；warning 用红紫敌意，不作普通强调"
    text: "标题锋利动漫感，正文白/黑高对比"
    accent: "刀痕、灵压火焰、墨线、队章、符纸、斩击弧。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通和风武士；必须保留 BLEACH 的黑白死神与灵压"
```

## 50. 无限暖暖

```yaml
style_brief:
  name: "无限暖暖"
  aliases: []
  tags: ["dress-up", "open-world", "fairy-tale", "fabric"]
  source:
    checked: "https://www.taptap.cn/app/247283"
    confidence: "high"
  game_features:
    type: "换装 / 开放世界 / 冒险"
    core_gameplay: "开放世界探索、能力套装、拍照、轻冒险和时装收集"
  visual_identity: "童话织物开放世界换装冒险"
  genre_presentation: "把开放世界换装视觉化为布料、花朵、阳光、奇想大陆、能力服装和柔软的梦幻旅行。"
  mood: "梦幻、温柔、自由、精致"
  world_elements: "奇想大陆、服装能力、花朵、蝴蝶、相机、缎带、旅行、织物"
  colors: "暖白/浅粉作底；草绿/天空蓝作旅行；金色作高级；薰衣草紫作梦幻；红色仅作少量提示"
  materials: "丝绸、蕾丝、柔光玻璃、纸卡、花纹布料、珍珠"
  shape_language: "柔圆、花瓣边、缎带曲线、轻薄层叠"
  background_visual_language: "童话草地/衣橱/花园远景，边缘织物花朵，中心留白"
  ui_translation:
    panel: "柔光纸卡+蕾丝/缎带边，小卡保留干净"
    panel_base_style_hint: "暖白 / 极淡粉白纸卡或低饱和柔光底，柔圆纸卡边，细珍珠或淡金边；可联想缎带折角、小纽扣、蕾丝角片或单片花瓣边角，但大蝴蝶结、心形吊坠、花束、衣服、相机、角色和复杂蕾丝不作为基础底板来源"
    state: "selected 用金色花瓣/缎带边；warning 用柔红小提示"
    text: "标题优雅圆润，正文深灰清晰"
    accent: "缎带、花瓣、蝴蝶、衣架、相机、纽扣、蕾丝。装饰应具有功能或世界观语义，避免每张卡都堆强装饰。"
  forbidden_mistakes: "误判成普通粉色换装或儿童童话；需保留开放世界旅行和能力服装"
```
