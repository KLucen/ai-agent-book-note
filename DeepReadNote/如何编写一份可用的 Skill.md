---
书名: 深入理解 AI Agent：设计原理与工程实践
类型: 指南（Skill 编写方法）
来源: 原书第 2 章 §「动态提示词与 Agent Skills」；实例对照 anthropics/skills · frontend-design
回应日期: 2026-09-13
tags:
  - AI-Agent
  - 精读笔记
  - DeepReadNote
---

# 如何编写一份可用的 Skill

> [!info] 本文是什么
> 因你在 [[QuickNote/Quick_note_2]] 中的请求而写：把原书第 2 章关于 Agent Skills 的规范整理成**一份可操作的编写指南**，并用真实示例 **`anthropics/skills` 的 `frontend-design`** 逐段对照，看清「一份好 Skill 的骨架长什么样」。
> 标注：〔原书〕= 书中规范（带锚点）；〔对照〕= 真实 Skill 示例分析；〔补充〕= 原书之外的工程经验。

---

## 一、先明确 Skill 的运行时契约（写之前必须知道的约束）

写 Skill 不是写文档，而是写**要被按需加载进上下文的指令**。三条约束决定了写法：

1. **三层结构、按需加载**：元数据目录常驻 → 判断需要后加载 `SKILL.md` 正文 → 正文引用的子文档按需再读〔原书 §「Skills：领域能力的可组合单元」〕。所以正文不能假设「读者一定会读完」，要靠元数据被选中。
2. **元数据 `description` 是路由条件，不是功能介绍**：

> 它应当足够短（控制常驻的 token 量），但写法要像路由条件而非功能介绍。（原书 §「Skills：领域能力的可组合单元」）
>
> 真正有效的描述是路由条件——“何时该用我”比“我能做什么”重要得多。（原书 §「Skills：领域能力的可组合单元」）

3. **正文首次加载有成本，之后并入前缀**：正文在调用位置作为 user message（或 `<skill>` 标记片段）注入，一次写入、后续轮次持续命中——所以**正文里的稳定内容不会每轮重复付费**，但首轮加载与目录渲染有成本〔原书 §「Skills 在上下文中的位置」〕。

Skill 的本质是“把外部内容当作指令加载”的制度化形式（原书 §「提示注入」）——因此**第三方 Skill 必须先审查再安装**，其地位等同于「将要执行的代码」。

---

## 二、写作四部分（原书规范）

原书给出的四段式结构：

| 部分 | 写什么 | 判断标准 |
| --- | --- | --- |
| **角色与读者** | 这份 Skill 服务谁、面向什么任务、输出要达到什么标准 | 新员工读完知道自己被放在什么位置 |
| **核心原则** | 只保留 3–5 条最重要的判断，**每条配正例与反例** | 删掉不影响决策的都不该留 |
| **禁止清单** | 高频错误、越权动作、容易误解的表达，**同时写清合法例外** | 有例外规则，而不是一张越长越好的禁用词表 |
| **参考资料** | 术语表、模板、范文、更详细的子文档 | 能按需加载，不占用正文常驻成本 |

规则句式建议：**「作用域 + 动作 + 例外 + 验证方式」**〔原书 §「如何编写一份可用的 Skill」〕。

两条起步路径：

- **写作型**（原书示范）：从 3–5 篇自己最满意的原创文章出发 → 让 Agent 归纳用词/句式/段落/语气生成约 20 行初版 → 用它处理真实任务、作者逐句改稿 → **把反复出现的改动整理回 Skill**，并为每条规则保留正例、反例与适用范围。
- **工程型**（原书 Skill 定义的另一半）：把重复操作脚本化——Skill 可以捆绑可执行代码与模板文件（如 PPT Skill 带模板与解析脚本），让「流程 + 工具 + 模板」一起按需加载。

---

## 三、真实示例逐段解剖：`frontend-design`

示例仓库：`anthropics/skills` → `skills/frontend-design/SKILL.md`（安装：`npx skills add https://github.com/anthropics/skills --skill frontend-design`）。

### 3.1 元数据：路由条件式 description

```yaml
name: frontend-design
description: Guidance for distinctive, intentional visual design when building new UI or reshaping an existing one. Helps with aesthetic direction, typography, and making choices that don't read as templated defaults.
license: Complete terms in LICENSE.txt
```

〔对照〕三点值得照抄：

1. **触发场景写进 description**：「when building new UI or reshaping an existing one」——这是「何时该用我」，不是「我很强」；
2. **反向边界也写进去**：「making choices that don't read as templated defaults」——直接说明它**要避免**什么，等于给路由加了一个判别条件；
3. **`license` 字段**：来源与授权写在元数据里，正是「第三方 Skill 要可审查」的落地方式。

### 3.2 开场：用角色设定替换「背景介绍」

> Approach this as the design lead at a design studio known for giving every client a distinct visual identity that is not mistaken for anyone else's.（frontend-design SKILL.md）

〔对照〕它没有一句「前端设计很重要」的铺垫，而是直接给模型一个**职位 + 成功标准 + 失败画像**（「已经否决过看起来像模板的方案」）。这对应原书「角色与读者」：**先让执行者知道自己被期望达到什么标准**。

### 3.3 输入澄清：缺信息时先对齐，而不是硬编

> If the brief does not identify what the product or subject matter is, identify it yourself before designing, and confirm with the client.…If there's any information in your memory about the client's preferences…, use that as a hint.

〔对照〕这解决了 Skill 类指令的一个通病——**指令依赖的输入常常缺失**。它的处理方式很规范：自拟一个具体提案（产品/受众/首要任务）→ 与用户确认 → 才开工；并显式提示「可以用记忆里的偏好作为线索」（呼应第 3 章用户记忆的用途）。

### 3.4 核心原则：少而可判定

正文的设计原则集中在五条（hero / typography / structure / motion / copy），且**每条都配了「反例」**：

- 排版：「Avoid these default typographic treatments; they are the commonest tells of a generated page」，随后列出三条具体反例（只给一个词加粗/斜体、标签全大写、内容上方加多余标签）；
- 结构：「numbered markers (01 / 02 / 03)…only appropriate if the content actually is a sequence」；
- 动效：逐个模块的入场动画与卡片 hover 是「the generic default and read as AI-generated」；
- 文案：按钮写「Save changes」而非「Submit」，动作名在整条流程中保持一致。

〔对照〕这正是原书要求的「核心原则 + 正例反例」的教科书式写法：**原则可判定**（能对照具体产物检查），而不是「要美观、要专业」这类无法验证的形容词。

### 3.5 禁止清单 + 合法例外（本示例最值得学的一点）

它列了五类「AI 生成设计的默认特征」（暖奶油底 + 衬线高对比 + 陶土色强调；近黑底 + 单一荧光色；报纸式细线栏；SaaS 圆角卡片全家桶；模板化「ALL-CAPS 眉标 + 中点连接 + ⇢」装饰），但紧接着给了**例外规则**：

> All traits are legitimate for some briefs, but they are defaults rather than choices…Where the brief pins down a visual direction, follow it exactly — the brief's own words always win, including when it asks for one of these looks.

〔对照〕这就是原书说的「禁止清单要写清合法例外」：**禁止的是「未经思考的默认」，不是这些风格本身**。少了这句例外，Skill 会把合理需求也误杀（等价于第 1 章说的「误拒绝」）。

### 3.6 流程：两遍工作法 + 自审门禁

> Work in two passes. First, brainstorm a short design plan…Then review that plan against the brief before building: if any part of it reads like the generic default you would produce for any similar page…revise that part, say what you changed and why. Only after you've confirmed the relative uniqueness of your design plan should you start to write the code.

〔对照〕这是可迁移到任何领域的**质量门禁**写法：先产出轻量计划（token 成本低）→ 用明确判据（「换个 prompt 是否也会走到这里」）自检 → 修订并说明理由 → 再动手做昂贵的那步（写代码）。对工程类 Skill，把「写代码」换成「改文件/跑命令/发布」即可。

它也给了自审的**证据形式**：

> Critique your own work as you build, taking screenshots to review if your environment supports it — a picture is worth 1000 tokens.

〔对照〕这与第 1 章「提议者—审核者」原则一致：**评判方看产物（渲染结果/截图），而不是看自己的推理过程**；同时它的「quality floor」（响应式、键盘焦点可见、尊重 reduced motion、可访问、配色和谐）是**不要宣称、直接做到**的验收底线。

### 3.7 参考资料：不是必需品

`frontend-design` 目录下只有 `SKILL.md` 与 `LICENSE.txt`，没有脚本与模板——因为它的知识是**判断规则**，不需要资源文件。对照 `pptx` 这类 Skill 会捆绑模板与脚本，用于「必须产出具体文件」的任务。

〔对照〕结论：**「参考资料」按需添加**。判断标准是：这条知识是「一次性读懂的规则」还是「需要反复调用的素材/工具」——后者才需要文件化。

---

## 四、把示例映射回原书四部分（自查表）

| 原书要求 | frontend-design 的做法 | 你写自己的 Skill 时是否具备 |
| --- | --- | --- |
| 角色与读者 | 开场给「设计主管」立场 + 客户已否决模板方案的成功标准 | ☐ 你的执行者知道被期望达到什么标准吗 |
| 核心原则（3–5 条 + 正反例） | 五条原则，每条带具体反例清单 | ☐ 原则能用产物验证吗 |
| 禁止清单 + 合法例外 | 五类 AI 默认特征 + 「brief 的话优先」例外规则 | ☐ 有没有写清例外，避免误杀 |
| 参考资料 | 无文件（判断型知识）；对比 pptx 带模板脚本 | ☐ 你的知识需要素材/脚本支撑吗 |
| 规则句式（作用域+动作+例外+验证） | 「Where the brief pins down a direction, follow it exactly」= 作用域+例外+动作 | ☐ 每条规则能落到这四要素吗 |
| 路由描述 | description 同时写「何时用」与「要避免什么」 | ☐ 你的 description 是路由条件还是功能列表 |
| 安全与来源 | `license` 字段 + 依赖审查传统 | ☐ 来源可追溯、内容已审查吗 |

---

## 五、可复制的 SKILL.md 模板

```markdown
---
name: <kebab-case 名称>
description: <何时使用：触发场景 + 任务类型 + 要避免的典型失败；一句话，像路由条件>
license: <授权说明或指向 LICENSE>
---

# <Skill 名称>

<一句话：这份 Skill 让执行者在什么位置、达到什么标准。>

## 何时使用 / 何时不使用
- 使用：<具体场景 1>；<场景 2>
- 不使用：<容易误触发的相近场景>（反例：<具体反例>）

## 核心原则（3–5 条）
1. **<原则>**：<为什么 + 正例>
   - ✅ <正例>
   - ❌ <反例>
2. …

## 禁止清单（含合法例外）
- 禁止：<高频错误 / 越权动作 / 易误解表达>
- 例外：<什么条件下允许，且需要什么额外确认>
- 验证方式：<怎么检查自己是否违规>

## 流程（含自审门禁）
1. 计划：<轻量产出>
2. 自审：<对照什么判据检查；不通过就修订并说明理由>
3. 执行：<昂贵步骤>
4. 验收：<质量地板，不需宣称>

## 参考资料
- <子文档 / 模板 / 脚本 / 术语表，按需加载并说明何时读>
```

---

## 六、写完后的自检清单

- [ ] `description` 写的是**路由条件**（何时用/何时不用），长度受控；
- [ ] 正文能用一句话说清「执行者应达到的标准」；
- [ ] 核心原则 ≤5 条，每条都有可判定的正例与反例；
- [ ] 禁止清单配了**合法例外**与验证方式（避免误拒绝）；
- [ ] 有明确的流程与**自审门禁**（先便宜后昂贵）；
- [ ] 昂贵/高风险动作有独立验证或确认（对应第 1 章护栏执行层）；
- [ ] 需要反复使用的素材/脚本已文件化并按需加载；不需要的没有硬塞；
- [ ] 来源与许可能追溯；第三方内容已审查（视同代码）；
- [ ] 用一个**真实任务**跑过一次，并据改稿/失败点修订过至少一轮；
- [ ] 没有把可以写成代码校验的规则，只写成「请注意」。

〔精读批注〕最后一条最容易被忽视：原书反复强调的取舍是「确定性的事交给代码与工具，语义判断交给模型」。如果你的 Skill 里在教模型「数一数、算一算」，那多半应该是脚本而不是提示词。

---

## 七、安装与试用

```bash
# 安装示例 Skill（Claude Code / 兼容运行时）
npx skills add https://github.com/anthropics/skills --skill frontend-design
```

- **你的 Obsidian 场景**：把本文第五节模板复制成 `你的库/skills/<name>/SKILL.md`，配合原书实验 2-6（Paper→PPTX）与实验 2-7（从个人范文建"去 AI 味"写作 Skill）走一轮真实任务，再按第四节自查表补漏。
- 配套实验与记录区：[[实验解读/第2章 上下文工程]] 的 2-6 / 2-7。

---

*来源与版权：`frontend-design` 的文字引用摘自 `github.com/anthropics/skills`（`skills/frontend-design/SKILL.md`），仅用于方法教学与对照分析，版权归 Anthropic，依其仓库许可（见该目录 `LICENSE.txt`）使用；原书引用取自 `D:\ObsidianRepository\book\chapter2.md` 并逐字核验。*
