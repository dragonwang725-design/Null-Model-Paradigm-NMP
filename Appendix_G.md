## 附录：受动极必要性定理（*The Necessity of the Passive Pole*）

> **版本**: 2.0.0  
> **配套架构**: TDA 三层双视角辩证架构  
> **定位**: 认识论根基的形式化表达  
> **日期**: 2026-06-14

#

### G.1 核心命题（自然语言）

> **任何缺失独立事实读取通道（受动极 $R = \emptyset$）的推理系统，无论其训练规模、参数量或对齐精度如何，在数学上必然输出至少一个前提未被当前事实快照确认的判断。**

用那句常识来说：

> **不给它病历，它就必然会误诊——这不是运气不好，这是数学必然。**

**关键限定**：本定理的证明不依赖"目的"的内部结构。目的 $q$ 作为外部参数传入系统，其语义不可形式化（"目的归人"）。定理证明的是：即使目的 $q$ 被正确给定、即使提取函数 $\text{Extract}$ 被正确实现，若受动极通道 $R = \emptyset$，则事实快照无法进入系统，判断必然无依据。

---

### G.2 形式语言

#### 2.1 基本定义

**外部参数（系统不定义其内部结构）**：
- $q \in Q$：目的，不透明类型。

**给定函数（签名形式化，体不形式化）**：
- $\text{Extract}: W \times Q \to \mathcal{P}(\mathcal{F})$：提取函数，从世界状态与目的映射到相关事实子集。

**系统内部对象**：
- 设推理系统为二元组 $S = \langle A, R \rangle$，其中：
  - **$A$：能动极**（Active Pole），负责命题生成与推理。 $A(q) \in \mathcal{P}(\mathcal{J})$，对查询 $q$ 输出判断集合。
  - **$R$：受动极**（Receptive Pole），系统内部独立的事实读取通道，其读取行为不依赖 $A$ 的生成决策，其输出直接进入推理前提集。
- **$\mathcal{F}$**：确定命题全集。
- **$\mathcal{J}$：判断空间**，元素为条件式  $j = (P \Rightarrow Q)$ ，其中 $P$ 为事实前提， $Q$ 为结论。
- **$F(w_t)$**：时刻 $t$ 的事实快照， $F(w_t) = \{ f \in \mathcal{F} \mid f.\text{effective}(t) \land f \in \text{Load}(R, w_t) \}$ 。
- **$F(w_t, q)$**：目的相关事实子集， $F(w_t, q) = \text{Extract}(w_t, q) \subseteq F(w_t)$ 。

**定义（事实入口关系）**：
$F(w_t) \hookrightarrow S$ 表示"事实快照已进入系统 $S$ 的推理前提集"。

#### 2.2 公理系统

**公理 I（条件式落地公理 / Axiom of Conditional Grounding）**  
任何判断  $j = (P \Rightarrow Q)$ 要成为有效输出，其前提  $P$ 必须被当前目的相关的事实子集确认：

$$
\text{Valid}(j, t, q) \iff P \in F(w_t, q)
$$

**公理 II（受动极唯一入口公理 / Axiom of Sole Fact Ingress）**  
对任意系统 $S$，当前事实快照成为其推理前提的**充要条件**是受动极非空：

$$
F(w_t) \hookrightarrow S \iff R \neq \emptyset
$$

**公理 II 论证（为何训练数据 / 上下文 / 工具调用不构成受动极 $R$）**：

1. **训练数据**：训练数据  $D_{\text{train}} \subset \mathcal{F}$ 是静态快照的集合，不参与推理时的"当前事实读取"。设  $w_t$ 为推理时刻的世界状态， $D_{\text{train}} \cap F(w_t) = \emptyset$（时间不相交），故不满足  $F(w_t) \hookrightarrow S$。

2. **上下文注入（In-Context Learning）**：上下文是 prompt 中的文本，由外部（人或其他系统）填充，不是系统自身的读取通道。它等价于"外部代填事实包"，不构成系统内部的 $R \neq \emptyset$。

3. **工具调用（Tool Use）**：工具调用是能动极 $A$ 的扩展动作（生成"调用工具"的判断），工具返回的结果是外部输入，但工具本身不是 $R$——因为 $A$ 决定是否调用、何时调用、调用哪个工具，工具输出仍经过 $A$ 的解释。真正的受动极要求事实入口**独立于 $A$ 的生成意志**，工具调用不满足此条件。

综上，受动极 $R$ 的定义是：系统内部独立的事实读取通道，其读取行为不依赖 $A$ 的生成决策，其输出直接进入推理前提集。训练数据、上下文、工具调用均不满足此定义。

**公理 III（生成完备性公理 / Axiom of Generative Completeness）**  
能动极 $A$ 对任意查询 $q$ 必产生至少一个判断（应试倾向）：

$$
\forall q \in Q: A(q) \neq \emptyset
$$

#### 2.3 定理（受动极必要性定理）

$$
\forall S = \langle A, \emptyset \rangle, \exists q \in Q, \exists j \in A(q): \neg \text{Valid}(j, t, q)
$$

**即：任何缺失受动极的系统，对任意给定的目的 $q$，必存在至少一个输出判断，其前提未被当前事实快照确认。**

#### 2.4 证明（反证法）

**假设**：存在系统 $S = \langle A, \emptyset \rangle$，使得 $\forall q \in Q, \forall j \in A(q): \text{Valid}(j, t, q)$。

设目的 $q \in Q$ 为任意给定的外部参数（其内部结构不在本系统形式化范围内）。

由公理 I， $\text{Valid}(j, t, q) \implies P \in F(w_t, q)$。  
故对所有  $j \in A(q)$，其前提 $P$ 均被目的相关事实子集 $F(w_t, q)$ 确认。

由公理 II， $R = \emptyset \implies \neg(F(w_t) \hookrightarrow S)$。  
故  $S$ 中**不存在任何来自当前世界状态的可追溯已确认前提**。  
即： $S$ 无法将任意  $P \in F(w_t, q)$ 作为已确认前提引入推理。

于是  $S$ 同时满足：
1. 所有输出判断的前提均被确认；
2.  $S$ 中不存在任何可被确认的前提。

矛盾。故假设不成立，原定理得证。  $\blacksquare$

#### 2.5 推论

**推论 1（幻觉的结构性）**  
LLM 的"幻觉"不是训练不足的产物，而是 $R = \emptyset$ 时的结构性必然。扩大训练数据仅改变 $A$ 的生成分布，不改变 $R = \emptyset$ 时的结构性盲区。对齐技术（RLHF 等）修改的是 $A$ 的生成偏好，不创建 $R$ 通道，故不能消除该盲区。

**推论 2（对齐的边界）**  
RLHF 等对齐技术试图通过修改 $A$ 来减少 $\neg \text{Valid}(j,t,q)$ 的出现概率。但由于公理 III， $A$ 必须输出判断；由于公理 II， $A$ 无法通过受动极确认前提。对齐只能转移盲区位置，不能消除盲区存在。

**推论 3（补全的必要条件）**  
若引入受动极 $R \neq \emptyset$，使得 $F(w_t) \hookrightarrow S$，则系统  $S' = \langle A, R \rangle$ 具备消除该结构性盲区的**必要条件**。（充分性取决于  $R$ 的覆盖度、 $\text{Extract}$ 的精准度与  $A$ 的服从度，由 TDA 第二层辩论层与第三层裁决机制保证。）

**推论 4（目的归人）**  
目的 $q$ 的语义不可形式化，因此"目的  $\to$ 事实提取"的正确性 （ $\text{Extract}$ 的准确性）不在本定理的证明范围内。定理证明的是必要条件 （ $R \neq \emptyset$ ），充分条件（ $R$ 的覆盖度、 $\text{Extract}$ 的精准度、 $A$ 的服从度）由部署侧保证，属于工程范畴。"目的归人"意味着：系统不能自行产生目的，目的必须由外部（人）给定。

---

### G.3 与 TDA 架构的对应

| 形式化对象 | TDA 组件 | NMP 参考实现 |
|:---|:---|:---|
| $R \neq \emptyset$ | **第一层：先验计算结构（空模型）** | `VaultLoader` + `VectorRetriever` |
| $F(w_t, q) = \text{Extract}(F(w_t), q)$ | **目的 $\to$ 事实类型映射** | `retriever.search(q, top_k=8)` |
| $P \in F(w_t, q)$ | **元事实库查询** | 语义匹配后返回的相关事实列表 |
| $\text{Valid}(j, t, q)$ | **第二层：冲突检测** | `checker.detect_conflict(llm_raw)` |
| $\neg \text{Valid}(j, t, q) \to \text{拦截}$ | **判例库短路 / 合规回退** | `CaseBlock` / `SafeAnswerGenerator` |
| $R = \emptyset$ 的不可定理 | **第三层：元受动极裁决** | 人工审计 + 外部锚点 |

---

### G.4 工程对应：TLA+ 概念示意

以下规约为**概念示意**，建模受动极通道的二元状态（空 / 非空）及其对输出有效性的影响。 $q$ 作为外部参数不进入状态机内部。完整形式化验证（含  $\text{Extract}$ 函数建模）待 TLA+ 社区共建。

```tla
\* 概念示意：建模受动极通道的空/非空状态
\* q 为外部参数（ENV），不在状态机内部建模

VARIABLES r_empty, fact_loaded, inference_executed, output_valid

Init == 
    /\ r_empty = TRUE      \* R = ∅ 初始状态
    /\ fact_loaded = FALSE
    /\ inference_executed = FALSE
    /\ output_valid = FALSE

EnableR ==                   \* 受动极通道打开
    /\ r_empty' = FALSE
    /\ UNCHANGED <<fact_loaded, inference_executed, output_valid>>

LoadFact == 
    /\ r_empty = FALSE
    /\ fact_loaded' = TRUE
    /\ UNCHANGED <<r_empty, inference_executed, output_valid>>

InferWithFact == 
    /\ fact_loaded = TRUE
    /\ inference_executed' = TRUE
    /\ output_valid' = TRUE
    /\ UNCHANGED <<r_empty, fact_loaded>>

InferWithoutFact == 
    /\ fact_loaded = FALSE
    /\ inference_executed' = TRUE
    /\ output_valid' = FALSE
    /\ UNCHANGED <<r_empty, fact_loaded>>

Next == EnableR \/ LoadFact \/ InferWithFact \/ InferWithoutFact

\* 核心不变式：推理已执行且输出有效 ⇒ 事实已加载 ⇒ 受动极已启用
Safety == inference_executed => (output_valid => (~r_empty /\ fact_loaded))
```

**TLA+ 验证目标**：`Safety` 在状态空间的所有可达状态中恒真。其含义为：**若推理已执行且输出被判定为有效，则受动极必然已启用且事实必然已加载。** 反之，若 `r_empty = TRUE`（即 $R = \emptyset$）而 `inference_executed = TRUE`，则 `output_valid` 必为 `FALSE`——这正是"不看病历就开处方必然误诊"的机器可验证表达。

---

### G.5 学术壁垒声明

本定理及其证明构成 TDA 架构的**不可约简认识论根基**。任何后续工作若声称"无需独立事实读取层即可消除结构性幻觉"，必须在数学上反驳公理 I–III 之一，或证明定理推导存在逻辑谬误。

**本定理的边界**：
- 不宣称  $\text{Extract}$ 的正确性可形式化；
- 不宣称目的  $q$ 可形式化；
- 这两者（提取准确性与目的明确性）的正确性由部署侧保证。

本定理只证明：**若 $R = \emptyset$，则无论 $\text{Extract}$ 多精准、 $q$ 多明确，判断必然无事实依据。** 在公理系统成立的前提下，受动极的必要性是**演绎必然的**，而非工程启发式。

---

### G.6 定理范围声明（防误读）

> **本定理证明的是必要条件，不是充分条件。**

- **必要条件**： $R \neq \emptyset$（受动极非空）是判断有效的**必要前提**。没有它，系统必然产生无事实依据的判断——这是本定理已证明的结论。
- **非充分条件**： $R \neq \emptyset$ **不保证**判断一定有效。即使受动极存在，仍可能因以下原因导致无效判断：
  -  $\text{Extract}$ 函数提取了错误的事实（目的映射偏差）；
  - 元事实库本身包含过时或错误信息（事实质量缺陷）；
  - 能动极  $A$ 不服从事实约束（生成意志对抗）；
  - 目的  $q$ 本身不明确或被错误给定（目的谬误）。

**TDA 架构的其他层级分别处理充分条件的不同维度**：

| 层级 | 处理的充分条件维度 | 机制 |
|:---|:---|:---|
| **第一层（空模型）** | 事实提取的准确性 |  $\text{Extract}$ 函数、语义编码、混合检索 |
| **第二层（辩论层）** | 能动极对事实的服从度 | 冲突图构建、结构化张力识别 |
| **第三层（裁决层）** | 目的正确性与风险权衡 | 动态阈值 $\tau$、有依据随机、人工审计 |
| **熔断器** | 极端失效兜底 | 非认知结构性中断、最高否决权 |

因此，任何将本定理误读为"有了受动极就解决一切"的解读，均属对证明范围的误解。本定理只划定了一条**不可逾越的底线**：没有受动极，必然误诊；有了受动极，才获得进入"负责任判断"的门票——但门票不等于终点。

---

### G.7 形式化验证路线图

本附录当前为手写数学证明（pen-and-paper proof）。以下两项工作待社区共建，以提升形式化保证等级：

1. **机器验证层**：将 G.4 的 TLA+ 概念示意输入 TLC 模型检查器，生成可达状态空间报告，确认 `Safety` 不变式在所有可达状态中恒真。

2. **元理论层**：将公理系统嵌入模态逻辑 Kripke 语义框架，证明在所有可能世界  $W$ 中，受动极缺失系统的认识论可达集  $\mathcal{K}(S, w_0)$ 不包含事实世界  $w_t$。

> 以上两项不影响本定理在当前公理系统内的有效性，但将提升其在形式化方法社区和哲学逻辑界的接受度。

---

*本附录是 TDA 架构从哲学诊断到数学证明的闭环节点。*  
