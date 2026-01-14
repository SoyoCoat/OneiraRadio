# 语义世界模型：数字文明的物理法则从哪里来？

## 源动力：创建第二个可居住的世界

我想做的事情很简单：**让人类拥有第二个可居住的世界——一个会自我延续的数字文明。**

不是游戏，不是元宇宙概念片，而是一个真正能运转的世界。

这个世界里有NPC，它们有记忆、有目标、有情绪。它们会自己做决定，会互相影响，会对玩家的行为产生涟漪反应。玩家离开后，这个世界继续运转。

但当我真正动手做的时候，撞上了一堵墙：

> **现在的AI不理解"愤怒地摔杯子"。**

## 一个简单的问题

假设NPC需要执行这个动作：

> "愤怒地把杯子摔在地上"

你期望的结果是：NPC手臂猛抬、杯子高速飞出、落地碎裂、碎片四溅、周围NPC被吓到转头看。

问题是：**现在没有任何AI系统能端到端地做到这件事。**

如果连这个都做不到，谈什么数字文明？

## 现有技术能做什么？

我调研了当前主流的World Model方案：

| 系统 | 输入 | 输出 | 局限 |
|------|------|------|------|
| Dreamer | 状态 + 动作向量 | 下一个物理状态 | 动作是预定义的数字，没有语义 |
| Cosmos | 文本/图像/视频 | 生成视频 | 没有动作输入接口，无法交互控制 |
| Sora/Genie | 文本/图像 | 生成视频 | 同上，纯生成，不接受动作 |
| LLM | 语义文本 | 语义文本 | 理解语义，但不懂物理世界怎么变化 |

**核心矛盾**：
- Dreamer有物理预测能力，但它不懂"愤怒"是什么
- LLM懂"愤怒"是什么，但它不知道杯子会碎成几片

两者之间没有桥。

## 把问题拆成三层

```
第1层：语义 → 动作    "愤怒地摔" → 具体怎么动
第2层：动作 → 物理    怎么动 → 杯子碎了
第3层：物理 → 反应    杯子碎了 → 周围NPC转头看
```

现状：
- 第1层：**空白**
- 第2层：Dreamer能做，但动作没语义
- 第3层：这是我正在做的事情（VIBE Engine）

我在做第3层——让NPC能感知环境变化并产生反应。但我发现，如果上游两层不通，第3层做得再好也是空中楼阁。

## 两条技术路径，都有问题

**路径一：让World Model原生理解语义**

让Dreamer直接接受"愤怒地摔杯子"作为输入。

问题：没有数据。你去哪找几百万条"语义动作描述 + 对应物理后果视频"的配对数据？

**路径二：级联一层翻译**

用LLM把语义拆解成动作序列：

```
"愤怒地摔杯子" → [抬手, 挥臂, 释放] → Dreamer
```

问题：**"先A再B"和"非常A地做B"不是一回事。**

"愤怒地摔"不是"抬手→挥臂→释放"的序列，而是整个动作被情绪调制——肌肉更紧、速度更快、轨迹更直接、力度更大。这是连续的、整体的，拆成离散步骤就丢失了。

## 第三条路：动作原语 + 风格调制

参考AI音乐生成的做法。

音乐AI不是把"悲伤的爵士乐"拆成音符序列，而是：

```
基础旋律 × 风格LoRA（爵士） × 情绪参数（悲伤=0.8）
```

同样的思路用在动作上：

```json
{
  "action": "throw",
  "object": "cup",
  "style": {
    "emotion": "angry",
    "intensity": 0.9
  }
}
```

动作 = 基础原语 × 连续风格调制

这比纯序列拆解表达力更强，比原生语义理解更可实现。

## 但这条路也有难点

1. **数据**：需要"同一动作、不同情绪强度"的配对数据，现在不存在
2. **标注**：什么算"愤怒"vs"烦躁"？谁来定义边界？
3. **物理连续性**：intensity从0.7到0.9，杯子碎裂方式是平滑变化还是离散跳变？

这些都是open problem。

## 回到源动力

我想创建一个数字文明，但发现连"愤怒地摔杯子"这么基础的事情都没有解决方案。

这不是一个功能缺失，是一层基础设施空白。

谁能定义出：
- 语义动作的标准schema
- 风格调制的参数空间
- 语义到物理的映射机制

谁就是在定义**数字世界的物理法则协议**。

游戏引擎定义了虚拟世界的渲染和碰撞。但数字文明需要的不止是物理，还有**语义到物理的因果链**。

这是我接下来要啃的硬骨头。

---

# Semantic World Models: Where Do the Physical Laws of Digital Civilization Come From?

## The Driving Force: Creating a Second Habitable World

What I want to do is simple: **Give humanity a second habitable world—a self-sustaining digital civilization.**

Not a game. Not a metaverse pitch deck. A world that actually runs.

This world has NPCs with memories, goals, and emotions. They make their own decisions, influence each other, create ripple effects in response to player actions. When players leave, the world keeps running.

But when I actually started building, I hit a wall:

> **Current AI doesn't understand "throw the cup angrily."**

## A Simple Question

Imagine an NPC needs to perform this action:

> "Throw the cup on the ground angrily"

What you expect: The NPC's arm swings up forcefully, the cup flies out at high speed, shatters on impact, fragments scatter everywhere, nearby NPCs get startled and turn to look.

The problem: **No AI system today can do this end-to-end.**

If we can't even do this, what digital civilization are we talking about?

## What Can Current Technology Do?

I surveyed the major World Model approaches:

| System | Input | Output | Limitation |
|--------|-------|--------|------------|
| Dreamer | State + action vector | Next physical state | Actions are predefined numbers, no semantics |
| Cosmos | Text/image/video | Generated video | No action input interface, no interactive control |
| Sora/Genie | Text/image | Generated video | Same as above, pure generation, no action input |
| LLM | Semantic text | Semantic text | Understands semantics, but doesn't know physics |

**The core contradiction**:
- Dreamer can predict physics, but doesn't understand what "angry" means
- LLM understands what "angry" means, but doesn't know how many pieces the cup will break into

There's no bridge between them.

## Breaking Down the Problem into Three Layers

```
Layer 1: Semantics → Action    "throw angrily" → specific motion
Layer 2: Action → Physics      motion → cup shatters
Layer 3: Physics → Reaction    cup shatters → nearby NPCs turn to look
```

Current status:
- Layer 1: **Blank**
- Layer 2: Dreamer can do this, but actions lack semantics
- Layer 3: This is what I'm working on (VIBE Engine)

I'm building Layer 3—enabling NPCs to perceive environmental changes and react. But I've realized that if the upstream layers aren't connected, Layer 3 is a castle in the air.

## Two Technical Paths, Both Problematic

**Path One: Make World Models natively understand semantics**

Let Dreamer directly accept "throw the cup angrily" as input.

Problem: No data. Where do you find millions of paired examples of "semantic action descriptions + corresponding physical outcome videos"?

**Path Two: Add a translation layer**

Use LLM to decompose semantics into action sequences:

```
"throw cup angrily" → [raise arm, swing, release] → Dreamer
```

Problem: **"First A then B" is not the same as "do B very A-ly."**

"Throw angrily" isn't a sequence of "raise→swing→release". It's the entire motion modulated by emotion—muscles tenser, speed faster, trajectory more direct, force greater. This is continuous and holistic; decomposing into discrete steps loses it.

## A Third Path: Action Primitives + Style Modulation

Reference how AI music generation works.

Music AI doesn't decompose "sad jazz" into note sequences. Instead:

```
Base melody × Style LoRA (jazz) × Emotion parameter (sadness=0.8)
```

Apply the same idea to actions:

```json
{
  "action": "throw",
  "object": "cup",
  "style": {
    "emotion": "angry",
    "intensity": 0.9
  }
}
```

Action = Base primitive × Continuous style modulation

More expressive than pure sequence decomposition, more achievable than native semantic understanding.

## But This Path Has Challenges Too

1. **Data**: Need paired data of "same action, different emotion intensities"—doesn't exist
2. **Labeling**: What counts as "angry" vs "irritated"? Who defines the boundaries?
3. **Physical continuity**: As intensity goes from 0.7 to 0.9, does cup shattering change smoothly or discretely?

These are all open problems.

## Back to the Driving Force

I want to create a digital civilization, but I've discovered that even something as basic as "throw the cup angrily" has no solution.

This isn't a missing feature. It's a missing layer of infrastructure.

Whoever can define:
- Standard schema for semantic actions
- Parameter space for style modulation
- Mapping mechanism from semantics to physics

Is defining **the protocol for physical laws of digital worlds**.

Game engines defined rendering and collision for virtual worlds. But digital civilization needs more than physics—it needs **the causal chain from semantics to physics**.

This is the hard problem I'm chewing on next.

---

*January 2026*
