# Codex Project Handoff
## AI Target Language Purity System
### Phase 1 — AI Target Language Purity Engine
#### 当前认可设计、技术路线与实施说明

---

## 0. 给 Codex 的最高优先级说明

请先完整理解以下约束，再开始提出代码或架构建议。

本文档完全属于 **Phase 1 — AI Target Language Purity Engine**。文中的 `Phase 0`–`Phase 11` 是 Engine 内部的实施步骤，不是整个系统的长期发展阶段。

Milestone 0 的权威规范是
[`MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md`](milestones/milestone_0/MILESTONE_0_PROBLEM_DEFINITION_AND_OUTPUT_PROTOCOL.md)。
本文较早形成的示例若与该规范冲突，以 Milestone 0 规范为准。

### 核心约束

1. 本项目只训练一个 Transformer 模型。
2. 训练数据主要是三类目标语言参考语料：
   - 上海话参考语料
   - 粤语参考语料
   - 普通话参考语料
3. 不训练第二个“混合检测模型”。
4. 不需要人工做 token-level language labels。
5. 不要把某个词直接断言为“绝对属于某语言”。
6. 输出应使用：
   - evidence for Shanghainese
   - evidence for Cantonese
   - evidence for Mandarin
7. 解释方法以 occlusion / leave-one-out attribution 为核心。
8. 训练语料检索可以作为辅助证据，但不能替代主模型。
9. 第一版不要求普通话翻译或平行语料。
10. 第一版不要求自动改写，也不要求判断语义是否保持。
11. 混合句主要用于测试和评估，不是训练主模型的必要条件。
12. 不要把 softmax 分数解释为“句子中有多少百分比属于某语言”。

---

# 1. 项目目标

## 1.1 使用场景

一个句子被写作或生成时带有明确的目标语言变体，例如上海话、粤语或普通话。

文本来源可以包括：

- 其他大模型生成的目标方言文本；
- 学习者尝试写出的目标方言文本；
- 方言母语者或继承语者在普通话书写教育影响下写出的目标方言文本；
- 人工构造的混合或挑战样例。

本项目分析的核心不是文本是否由大模型生成，而是：

> 当一个句子以某个目标语言变体为 intended target variety 时，它是否表现出目标变体证据，或明显表现出竞争变体证据？

例如，目标语言是上海话，但输入句子是：

> 老师，唔该侬帮我睇下这份作业。

系统需要完成以下分析：

1. 这句话是否充分符合目标语言；
2. 如果不像纯上海话，最强的竞争语言是什么；
3. 哪些词或短语推动了模型作出粤语、普通话或上海话判断；
4. 可选：从训练语料中检索真实相似用例，支持解释。

---

## 1.2 推荐项目名称

AI Target Language Purity System

也可以在代码仓库中简写为：

`ai-target-language-purity-system`

---

# 2. 核心研究问题

本项目不是简单回答：

> 这句话属于哪种语言？

而是回答：

> 当 intended target variety 是上海话时，这句话是否表现出明显的粤语或普通话证据？  
> 哪些词或短语最推动了这些语言判断？

因此，本项目的研究对象可以是 LLM-generated text，也可以是 learner-produced 或 human-written text。输入来源不同，但核心任务相同：分析目标语言变体一致性和竞争变体证据。

---

# 3. 任务定义

## 3.1 训练任务

训练一个三分类 Transformer：

- Shanghainese
- Cantonese
- Mandarin

输入：

```text
一句纯语言文本
```

输出：

```text
三个语言类别的 logits / probabilities
```

例如：

```text
Shanghainese: 0.74
Cantonese:    0.18
Mandarin:     0.08
```

---

## 3.2 分析 / 测试任务

分析输入可能是混合句、学习者句子、母语者书写句子或 LLM 生成句子：

```text
阿拉今日去学校。
```

目标语言：

```text
Shanghainese
```

模型可能输出：

```text
Shanghainese: 0.42
Cantonese:    0.19
Mandarin:     0.39
```

解释：

- 句子仍有上海话证据；
- 普通话是最强竞争语言；
- 不能把 42% 解释为“42%的句子是上海话”；
- 这只是三分类器对整句的相对支持分布。

---

# 4. 为什么只用纯语料训练也成立

纯语料训练的目的是让模型学习三种语言的分布差异，例如：

上海话中经常出现：

```text
阿拉
今朝
学堂
侬
```

普通话中经常出现：

```text
我们
今天
学校
你
```

粤语中经常出现：

```text
我哋
今日
返学
你
```

模型不需要知道：

```text
学堂 = 学校
```

也能学到：

```text
学堂更支持上海话分类
学校更支持普通话分类
```

这是语言分布学习，不是跨语言语义对齐。

---

# 5. 第一版不需要翻译

## 5.1 不需要翻译的功能

以下任务不需要给上海话语料添加普通话翻译：

- 三语言分类；
- 判断目标语言是否受到竞争语言影响；
- 分析哪些词推动了某语言判断；
- 展示训练语料中的相似语言用例。

---

## 5.2 需要翻译或语义层的功能

以下任务超出第一版范围：

- 判断“学堂”和“学校”语义相同；
- 判断生成句是否保持原始意思；
- 推荐“学校”应替换为“学堂”；
- 自动将混合句改写成地道上海话；
- 验证改写前后语义保持。

这些功能未来需要：

- 平行语料；
- 语义标签；
- 固定的多语语义模型；
- 或外部 LLM。

它们必须被视为独立语义层，不能混入第一版语言纯度分类任务。

---

# 5A. V1 关键设计决定

## 5A.1 Softmax 三分类，而不是 Sigmoid 多标签

V1 采用 softmax 三分类：

```text
Shanghainese
Cantonese
Mandarin
```

训练目标是：

```text
纯上海话 → Shanghainese
纯粤语 → Cantonese
纯普通话 → Mandarin
```

也就是说，模型学习的是：

> 如果必须在三个纯语言分布中选择一种，这句话相对最像哪一种？

Softmax 输出始终相加为 1。例如：

```text
Shanghainese: 0.63
Cantonese:    0.27
Mandarin:     0.10
```

这不表示：

> 这句话 63% 是上海话。

更准确的解释是：

> 在三个纯语言分布中，这句话相对最接近上海话，但也表现出一定粤语证据，因此模型没有给出极高的上海话置信度。

因此，softmax 分数表示：

> relative affinity to each learned language distribution

而不是：

> percentage of language composition

Sigmoid 多标签方案理论上更贴近“一句话可以同时具有多种语言特征”的直觉，但 V1 的训练数据是单标签纯语料，没有可靠的 multi-label 混合训练数据。因此 sigmoid 与当前训练目标不一致，暂不作为 V1 方案。

未来可以把 softmax vs sigmoid 作为后续研究问题，而不是 V1 的设计重点。

## 5A.2 V1 Explainability 的目标

V1 的解释模块不是为了完整解释 Transformer 内部机制，而是为了解释：

> 模型为什么认为一句话表现出某种语言变体证据。

因此解释应尽量简单、可复现，并依赖同一个分类器。

V1 保留两种 occlusion：

1. Deletion occlusion：删除某个 span 后重新分类。
2. Mask occlusion：用 `[MASK]` 保留位置但遮蔽内容后重新分类。

如果两种 occlusion 都使某个语言分数明显下降，则该 span 对该语言的 evidence 更稳定。

如果两种 occlusion 结果不一致，则解释需要更谨慎：

- deletion 可能破坏句法结构；
- mask 可能引入模型未自然见过的输入；
- 两种方法的差异可作为 attribution 稳定性的参考。

V1 不试图证明哪一种 perturbation 绝对正确，而是用二者互相检查解释是否稳定。

## 5A.3 V1 暂不做 Replacement / Counterfactual

V1 暂不自动做 replacement、counterfactual replacement 或 semantic replacement。

原因是这些方法会引入额外问题：

- 如何确定被替换 span 的词性和结构位置；
- 如何保持句法自然；
- 如何保证语义一致；
- 如何保证 replacement 本身不带来新的语言偏向；
- 如何在低资源方言场景中自动生成高质量反事实。

因此，automatic replacement 会把项目扩展成：

> 如何生成高质量语言反事实。

这超出 V1 的核心目标。

如果未来拥有可靠的平行语料、人工词汇对齐或其他语义资源，可以把人工或半自动 counterfactual 作为后续分析实验。但它不属于 V1 自动 pipeline。

## 5A.4 Retrieval 优先于 Replacement

相比自动 replacement，training corpus retrieval 更符合 V1 的解释目标。

例如，如果模型认为：

```text
睇下
```

支持 Cantonese 判断，系统可以进一步展示训练语料中的相似用例。

这种辅助解释来自训练数据分布，而不是声称该 span 具有绝对语言归属。

因此 retrieval 的推荐定位是：

> similar training examples that support the model evidence

而不是：

> dictionary proof that this word belongs to one language

Retrieval 是辅助解释模块，不是第二个训练模型，也不替代 occlusion evidence。

---

# 6. 系统总体架构

```text
Pure Shanghainese Corpus
Pure Cantonese Corpus
Pure Mandarin Corpus
           |
           v
Data Cleaning and Splitting
           |
           v
One Transformer 3-way Classifier
           |
           +----------------------+
           |                      |
           v                      v
Sentence-level Prediction     Embedding Extraction
           |                      |
           v                      v
Target-language Analysis      Training Corpus Index
           |
           v
Occlusion / Leave-one-out Attribution
           |
           v
Evidence for Each Language
           |
           v
Optional Nearest-neighbor Examples
           |
           v
Final Explainable Report
```

---

# 7. Step-by-step 实施体系

## Phase 0：问题定义和输出协议

先固定系统输出，避免边做边改任务。

### 输入

```json
{
  "target_language": "Shanghainese",
  "text": "阿拉今日去学校。"
}
```

### 推荐输出

```json
{
  "request": {
    "target_language": "Shanghainese",
    "original_text": "阿拉今日去学校。",
    "normalized_text": "阿拉今日去学校。"
  },
  "language_scores": {
    "Shanghainese": 0.42,
    "Cantonese": 0.19,
    "Mandarin": 0.39
  },
  "assessment": {
    "status": "provisional_status_name"
  },
  "evidence": [
    {
      "original_span": "学校",
      "normalized_span": "学校",
      "deletion_scores": {
        "Shanghainese": 0.48,
        "Cantonese": 0.17,
        "Mandarin": 0.31
      },
      "mask_scores": {
        "Shanghainese": 0.46,
        "Cantonese": 0.18,
        "Mandarin": 0.33
      }
    }
  ]
}
```

这里的状态名是占位符。最终状态集合和措辞需要开发数据验证，不在
Milestone 0 中提前固定。三个语言分数已经包含竞争语言信息，因此不另设
冗余的 `strongest_competing_language` 字段。

说明：

- `deletion_evidence = original_score - score_after_deletion`
- 如果删除某词后 Mandarin 分数下降 0.22，则 `deletion_evidence = 0.22`，该 span 是 Mandarin evidence。
- 正 evidence 表示该 span 支持对应语言的模型判断。
- 最终 UI 可以把下降幅度转成星级或条形图。
- 不要把该数值称为“词属于某语言的概率”。

---

## Phase 1：数据准备

### 1.1 数据格式

统一为：

```text
text,label,source,split
```

示例：

```csv
阿拉今朝去学堂。,Shanghainese,corpus_a,train
我今日返学。,Cantonese,corpus_b,train
我们今天去学校。,Mandarin,corpus_c,train
```

### 1.2 数据清洗

至少检查：

- 空文本；
- 重复句；
- 极短句；
- HTML 或异常符号；
- 编码问题；
- 标点是否严重失衡；
- 是否存在明显跨数据集重复；
- 是否存在来源泄漏；
- 是否存在某一类别大量固定模板。

### 1.3 数据平衡

需要记录每类：

- 句子数量；
- token 数量；
- 平均句长；
- 字符覆盖率；
- 来源数量。

类别不平衡时可考虑：

- class weights；
- 下采样；
- 合理的数据增强。

不要为了平衡而删除大量低资源上海话数据。

### 1.4 划分原则

优先按来源、说话人、文档或对话分组划分，而不是随机逐句划分。

目标：

- 避免同一来源的近重复句同时进入 train 和 test；
- 避免模型只学来源格式。

建议：

- train
- validation
- target-language reference test
- mixed-language challenge test

---

## Phase 2：建立非神经 baseline

在训练 Transformer 之前，先做一个简单 baseline。

推荐：

- character n-gram TF-IDF
- logistic regression

目的不是最终使用，而是：

- 检查数据是否可分；
- 检查是否存在格式泄漏；
- 提供 Transformer 的比较基线；
- 帮助发现某些类别是否被奇怪符号轻松识别。

Baseline 输出同样是三分类。

---

## Phase 3：训练唯一的 Transformer

### 3.1 模型形式

使用一个 sequence classification Transformer。

候选：

- Chinese BERT / RoBERTa 类模型；
- multilingual BERT；
- XLM-R；
- 其他适合中文字符输入的 encoder。

模型选择应依据：

- 上海话书写方式；
- tokenizer 对方言字符和口语写法的覆盖；
- 算力；
- 数据量。

### 3.2 Tokenizer 检查

训练前必须检查：

- 上海话特有字是否被 `[UNK]`；
- 字符是否被拆得过碎；
- 罗马字或特殊标记如何处理；
- 粤语字如“唔、冇、佢”等是否合理编码。

如果 tokenizer 覆盖极差，再考虑：

- 添加新 token；
- character-level 模型；
- byte-level 模型。

不要一开始就过度复杂化。

### 3.3 训练目标

标准 cross-entropy 三分类。

标签只有：

```text
Shanghainese
Cantonese
Mandarin
```

不添加：

```text
Mixed
Shanghai+Mandarin
Shanghai+Cantonese
```

因为当前设计希望模型只通过纯语言分布学习。

### 3.4 训练监控

记录：

- train loss；
- validation loss；
- macro F1；
- per-class precision / recall / F1；
- confusion matrix；
- calibration metrics；
- checkpoint；
- random seed；
- config。

---

## Phase 4：纯语言分类评估

先确认模型能正确识别纯语言。

报告：

- accuracy；
- macro F1；
- per-class F1；
- confusion matrix；
- 按句长表现；
- 按来源表现；
- 按罕见词比例表现。

重点观察：

- 上海话是否被大量误判为普通话；
- 粤语和上海话是否因共享汉字而混淆；
- 模型是否只依赖少量明显方言字。

---

## Phase 5：构建混合句测试集

混合句主要用于测试，不一定用于训练。

建议同时有两种：

### 5.1 自动合成混合句

从纯句中替换词或短语。

例如：

上海话：

```text
阿拉今朝去学堂。
```

替换成：

```text
阿拉今朝去学校。
```

或：

```text
阿拉今日去学校。
```

需要保存替换记录：

```json
{
  "base_language": "Shanghainese",
  "source_text": "阿拉今朝去学堂。",
  "mixed_text": "阿拉今朝去学校。",
  "inserted_language": "Mandarin",
  "inserted_span": "学校",
  "replaced_span": "学堂"
}
```

这些标签来自构造过程，不需要人工逐词标注。

### 5.2 人工、学习者或真实 LLM 生成混合句

收集真实模型输出、学习者输出或其他人类书写样例，由懂语言的人进行句级评审：

- acceptable target language；
- questionable；
- clearly mixed；
- strongest competing-language pattern when identifiable。

不一定需要完整 token 标注。

---

## Phase 6：定义“目标语言状态”

由于模型没有训练 Mixed 类，不能直接说它学会了纯/混二分类。

因此需要基于开发集建立一个可解释的判定规则。

例如目标语言为 Shanghai：

```text
target_score = P(Shanghai)
competitor_score = max(P(Cantonese), P(Mandarin))
margin = target_score - competitor_score
```

可能规则（仅用于说明后续需要校准，不是已批准协议）：

```text
if target_score >= threshold_high and margin >= margin_high:
    status = "provisional_target_consistent_status"

elif competitor_score >= threshold_competitor:
    status = "provisional_mismatch_status"

else:
    status = "uncertain"
```

阈值必须根据 validation / challenge set 调整，不能凭空设定。

最终状态名称、数量和用户措辞推迟到开发与校准阶段决定。

不要轻易使用：

- pure
- impure

因为“纯”是很强的语言学断言。

---

## Phase 7：Occlusion / Leave-one-out Attribution

这是本项目解释模块的核心。

### 7.1 基本原理

对原句运行模型：

```text
阿拉今日去学校。
```

得到：

```text
Shanghai: 0.42
Cantonese: 0.19
Mandarin: 0.39
```

遮住“学校”：

```text
阿拉今日去[MASK]。
```

重新预测：

```text
Shanghai: 0.61
Cantonese: 0.22
Mandarin: 0.17
```

则：

```text
Mandarin deletion evidence = 0.39 - 0.17 = 0.22
```

因此：

> “学校”是支持 Mandarin 判断的重要 evidence。

遮住“阿拉”后，如果上海话分数大幅下降：

> “阿拉”是支持 Shanghainese 判断的重要 evidence。

---

### 7.2 不要只遮单个 tokenizer token

需要同时测试：

- 单字；
- tokenizer token；
- 词；
- 2–4 token phrase；
- linguistic span。

原因：

```text
睇下
唔该
这份作业
```

可能必须作为短语才有稳定贡献。

---

### 7.3 遮蔽方式

需要实验比较：

1. `[MASK]`
2. 删除 span
3. 替换为 neutral placeholder
4. span deletion with punctuation cleanup

注意：

- `[MASK]` 可能制造训练时不存在的输入；
- 直接删除可能改变语法结构；
- 第一版可以同时实现 mask 和 delete，并比较稳定性。

---

### 7.4 Evidence 定义

对语言 L：

```text
evidence_L(span) = original_score_L - occluded_score_L
```

值越大，说明该 span 越支持语言 L 的判断。

如果值为负：

- 移除该 span 后语言 L 分数反而上升；
- 说明该 span 可能抑制语言 L 判断。

最终输出只显示超过阈值的正 evidence。

---

### 7.5 Span 排名

对所有候选 span：

- 计算三个语言的 score delta；
- 按语言分别排序；
- 去除高度重叠 span；
- 优先保留信息更完整的短语；
- 限制每种语言最多显示 3–5 个证据。

---

## Phase 8：训练语料近邻检索

该模块是辅助解释，不是第二个训练模型。

### 8.1 目的

对高 evidence span 或句子上下文，检索训练语料中的真实例子。

例如：

```text
学校
```

检索结果：

```text
普通话：我们今天去学校。
普通话：学校已经开学。
上海话：无高相似结果或极少结果。
```

这样系统可以说：

> 该 span 推动了 Mandarin 判断，并且其相似用例主要来自普通话训练语料。

### 8.2 实现方式

优先方案：

- 复用主 Transformer 的 hidden states；
- 对训练句构建 embedding；
- 使用 FAISS 或 sklearn NearestNeighbors。

不需要训练新的神经网络。

### 8.3 重要限制

句子级 embedding 检索不等同于精确词典统计。

因此输出用：

```text
similar training examples
```

而不是：

```text
this word belongs to Mandarin
```

---

## Phase 9：解释结果格式

推荐最终文本输出：

```text
Target language: Shanghainese

Input:
阿拉今日去学校。

Sentence-level model scores:
- Shanghainese: 0.42
- Cantonese: 0.19
- Mandarin: 0.39

Assessment:
- Status: provisional mismatch wording

Evidence for Shanghainese:
- 阿拉
  Removing this span lowers the Shanghainese score by 0.24.

Evidence for Mandarin:
- 学校
  Removing this span lowers the Mandarin score by 0.22.

Evidence for Cantonese:
- No strong evidence found.

Interpretation:
These spans influenced the classifier's language decision.
They are not absolute token-level language labels.
```

---

## Phase 10：解释模块评估

不能只看例子“看起来合理”。

需要设计评估。

### 10.1 合成数据定位准确率

因为自动合成句知道插入片段在哪里，可以计算：

- top-1 span hit rate；
- top-k span hit rate；
- token-level overlap；
- phrase IoU；
- inserted-language attribution accuracy。

例如：

模型是否把被插入的“学校”排进 Mandarin evidence top 3。

### 10.2 Faithfulness

检查：

- 删除 top evidence span 后，对应语言分数是否显著下降；
- 删除随机 span 的影响是否更小；
- 删除 top-k evidence 后预测是否发生预期变化。

### 10.3 Stability

比较：

- mask vs deletion；
- 不同随机种子；
- 相近句子；
- 同义结构。

解释不应因极小无关变化完全翻转。

### 10.4 人工评估

让懂上海话、粤语、普通话的人评价：

- evidence 是否合理；
- 污染来源是否合理；
- 是否存在误导性解释；
- 结果是否对分析 LLM 输出、学习者文本或其他 human-written intended-variety text 有帮助。

---

## Phase 11：误差分析

建立错误类型表。

至少包括：

1. Shared expression
   - 三种语言都常见，无法归属。

2. Corpus imbalance
   - 某表达只因数据来源不均衡而看似属于某语言。

3. Topic leakage
   - 某语言语料集中讨论特定话题。

4. Orthographic variation
   - 不同写法被模型当成语言差异。

5. Named entities
   - 地名、人名或机构名影响判断。

6. Short sentence ambiguity
   - 输入太短，没有足够语言证据。

7. Context-dependent expression
   - 单独看词无法判断，必须看搭配。

8. Mask artifact
   - `[MASK]` 自身导致预测变化。

9. Tokenizer artifact
   - 特殊字被异常拆分。

10. Register variation
    - 口语、书面语、老派或新派用法被误判。

---

# 8. 关键概念澄清

## 8.1 “学校不像上海话”应如何准确表达

不要直接说：

> 学校不是上海话。

更准确的是：

> 移除“学校”后，模型对普通话的支持明显下降，同时对上海话的支持上升。因此，“学校”是该模型作出普通话判断的重要证据。

这是模型行为解释，不是绝对语言学归属。

---

## 8.2 “今日”可能是共享表达

如果“今日”在上海话和粤语语料中都大量出现，模型可能不会把它识别为强粤语证据。

这是正确现象，不是模型失败。

系统应允许：

```text
shared / insufficiently discriminative
```

而不是强行归类。

---

## 8.3 softmax 分数不是纯度比例

以下输出：

```text
Shanghainese: 0.42
Mandarin: 0.39
```

不代表：

```text
42% 的词是上海话
39% 的词是普通话
```

它只代表：

> 模型对整句三个类别的相对支持。

如果需要更可靠概率，应在开发集做 calibration：

- temperature scaling；
- expected calibration error；
- reliability diagram。

---

# 9. 推荐代码仓库结构

```text
ai-target-language-purity-system/
│
├── README.md
├── requirements.txt
├── configs/
│   ├── baseline.yaml
│   ├── transformer.yaml
│   └── attribution.yaml
│
├── data/
│   ├── raw/
│   ├── processed/
│   ├── splits/
│   └── challenge_sets/
│
├── src/
│   ├── data/
│   │   ├── clean.py
│   │   ├── validate.py
│   │   ├── split.py
│   │   └── synthesize_mixed.py
│   │
│   ├── baseline/
│   │   ├── train_ngram.py
│   │   └── evaluate_ngram.py
│   │
│   ├── model/
│   │   ├── train_classifier.py
│   │   ├── evaluate_classifier.py
│   │   ├── predict.py
│   │   └── calibrate.py
│   │
│   ├── explain/
│   │   ├── spans.py
│   │   ├── occlusion.py
│   │   ├── rank_evidence.py
│   │   └── stability.py
│   │
│   ├── retrieval/
│   │   ├── build_index.py
│   │   └── retrieve_examples.py
│   │
│   ├── evaluation/
│   │   ├── evaluate_pure.py
│   │   ├── evaluate_mixed.py
│   │   ├── evaluate_attribution.py
│   │   └── error_analysis.py
│   │
│   └── app/
│       └── demo.py
│
├── notebooks/
│   ├── 01_data_audit.ipynb
│   ├── 02_baseline.ipynb
│   ├── 03_transformer.ipynb
│   ├── 04_occlusion.ipynb
│   └── 05_error_analysis.ipynb
│
├── tests/
│   ├── test_data.py
│   ├── test_occlusion.py
│   └── test_output_schema.py
│
└── reports/
    ├── metrics/
    ├── figures/
    └── examples/
```

---

# 10. 推荐开发顺序

Codex 应按以下顺序指导开发，不要一开始就做完整 Demo。

## Milestone 0：Problem Definition

交付：

- 标签定义；
- 输入输出 schema；
- 项目边界；
- 成功指标。

## Milestone 1：Dataset Engineering

交付：

- 三类纯语料统一格式；
- 数据质量报告；
- 防泄漏划分；
- 数据统计。

## Milestone 2：Rule-based / N-gram Baseline

交付：

- character n-gram baseline；
- baseline metrics；
- 初步错误分析。

## Milestone 3：Transformer Sentence Detector

交付：

- 唯一 Transformer；
- 训练脚本；
- validation metrics；
- checkpoint。

## Milestone 4：Detector Evaluation

交付：

- 纯语料测试；
- 混合 challenge set；
- calibration；
- status threshold。

## Milestone 5：Explanation Module

交付：

- word/phrase span generator；
- mask and deletion occlusion；
- evidence ranking；
- example report。

## Milestone 6：Explanation Evaluation

交付：

- synthetic span hit rate；
- faithfulness；
- stability；
- human evaluation template。

## Milestone 7：Optional Retrieval

交付：

- training embedding index；
- nearest-neighbor examples；
- retrieval evidence display。

## Milestone 8：Optional Rewrite Pipeline

未来才考虑：

```text
Generate
→ Detect
→ Explain
→ Rewrite
→ Detect again
```

该阶段不是第一版核心。

也可以把未来版本的建议功能表述为：

```text
Detect evidence
→ Retrieve or infer possible target-variety alternatives
→ Generate cautious candidate replacement
→ Re-check with the Version 1 analyzer
```

该方向属于后续版本。第一版不提供自动纠错、教学反馈或地道改写承诺。

## Milestone 9：Demo

交付：

- 输入目标语言和句子；
- 展示语言分数；
- 展示 evidence；
- 展示训练语料相似例子；
- 展示解释限制。

---

# 11. 第一版成功标准

第一版至少满足：

1. 纯语言三分类达到合理 macro F1；
2. 对人工或合成混合句，竞争语言分数会合理上升；
3. 插入普通话片段后，Mandarin evidence 能定位该片段；
4. 插入粤语片段后，Cantonese evidence 能定位该片段；
5. 上海话主体词能被识别为 Shanghainese evidence；
6. shared expressions 不被强行归属；
7. 输出不声称 token 有绝对语言标签；
8. 所有解释都能追溯到模型分数变化；
9. 训练语料检索只作为辅助证据；
10. 项目只训练一个 Transformer。

---

# 12. Codex 不应擅自做的事情

请不要：

- 把项目改成 binary pure-vs-mixed classifier；
- 增加第二个 token classifier；
- 要求人工 token-level 标注；
- 把语言概率解释为词语比例；
- 直接生成“这个词属于粤语”的硬标签；
- 要求全部上海话语料配普通话翻译；
- 用词频表直接制造伪概率；
- 把 attention weight 当作唯一解释；
- 跳过 baseline 和数据审计；
- 在没有 challenge set 的情况下声称能检测污染；
- 一开始就做 rewrite；
- 为了功能丰富牺牲解释严谨性。

---

# 13. 给 Codex 的初始任务建议

可以从以下 prompt 开始：

```text
Read PROJECT_HANDOFF.md completely before proposing code.

Help me implement Milestone 1 only:
Dataset Engineering for a three-way Shanghainese/Cantonese/Mandarin
Transformer classifier.

Constraints:
- Training data consists primarily of target-language reference sentences.
- Do not add a Mixed label.
- Do not add a second model.
- The future explanation method will use occlusion and leave-one-out attribution.
- We need source-aware train/validation/test splits to reduce leakage.

Begin with a Milestone 1 design discussion only. Do not implement until the
project owner approves the exact scope.

First:
1. propose a minimal source-audit plan;
2. propose a source registry and data schema for discussion;
3. define the smallest useful Milestone 1 deliverable;
4. list the decisions that require approval before implementation.
```

完成 Milestone 1 后，再把下一阶段单独交给 Codex。

---

# 14. 最终项目定位

本项目的价值不是宣称：

> 某个词绝对属于某种语言。

而是建立一个可检验、可追溯的分析过程：

1. 模型从纯语言语料学习三种语言分布；
2. 当 intended-variety text 包含混合或竞争变体证据时，它在三类之间产生竞争性得分；
3. 遮蔽某词或短语，观察各语言分数变化；
4. 把导致分数下降的片段解释为该语言的 evidence；
5. 用训练语料中的真实近邻用例补充支持；
6. 对不确定和共享表达保持谨慎。

项目不只面向 LLM-generated text。更一般的研究定位是：

> Explainable target-variety consistency analysis for generated and human-produced text.

LLM 输出、学习者文本、方言母语者书写文本和人工挑战样例都可以成为分析对象，只要输入有一个 intended target variety。Version 1 的技术目标仍然保持克制：识别句级语言变体分数和模型 evidence，而不是提供自动纠错或语义改写。

从更抽象的角度看，本项目也可以被理解为：

> explaining local evidence when a text shows competition between learned linguistic distributions.

这种框架未来可能扩展到二语学习、区域变体、register / style consistency、作者风格变化等研究问题。但这些都属于远期研究窗口，不应扩大 V1 的实现范围。任何高风险或高 stakes 应用都需要单独验证、领域专家参与和严格伦理边界，不能直接从 V1 结果推出强结论。

核心表述：

> Rather than assigning absolute language labels to individual words,
> the system identifies which words or phrases provide evidence for each
> language prediction by observing how the classifier's outputs change
> when those spans are removed or masked.
