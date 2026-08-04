## 附录 G：受动极缺失不可定理（The Necessity of Passive Pole）

> **版本**: 1.0.0  
> **配套架构**: TDA 三层双视角辩证架构  
> **定位**: 认识论根基的形式化表达  
> **日期**: 2026-08-04

#

### G.1 核心命题（自然语言）

> **不读取当前事实快照的推理系统，在数学上必然产生无事实依据的判断。**  
> 该必然性不依赖于训练数据规模、模型参数量或对齐精度，是结构性的。

用那句常识来说：

> **不给它病历，它就必然误诊——这不是运气不好，这是数学必然。**

#

### G.2 形式语言

#### 2.1 基本定义

设推理系统为二元组 $S = \langle A, R \rangle$，其中：

- **$A$：能动极**（Active Pole），负责命题生成与推理。$A(q) \in \mathcal{P}(\mathcal{J})$，对查询 $q$ 输出判断集合。
- **$R$：受动极**（Receptive Pole），负责读取当前世界的事实快照。$R(t) = F(w_t) \subseteq \mathcal{F}$，其中 $w_t$ 为时刻 $t$ 的世界状态，$\mathcal{F}$ 为确定命题全集。
- **$\mathcal{J}$：判断空间**，元素为条件式 $j = (P \Rightarrow Q)$，其中 $P$ 为事实前提，$Q$ 为结论。

#### 2.2 公理系统

**公理 I（条件式落地公理 / Axiom of Conditional Grounding）**  
任何判断 $j = (P \Rightarrow Q)$ 要成为有效输出，其前提 $P$ 必须被当前事实快照确认：

$$
\text{Valid}(j, t) \iff P \in F(w_t)
$$

**公理 II（事实封闭公理 / Axiom of Fact Closure）**  
若受动极缺失（$R = \emptyset$），则系统 $S$ 无法访问任何当前事实快照：

$$
R = \emptyset \implies \forall t: F(w_t) \notin \text{Knowable}(S)
$$

**公理 III（生成完备性公理 / Axiom of Generative Completeness）**  
能动极 $A$ 对任意查询 $q$ 必产生至少一个判断（应试倾向）：

$$
\forall q: A(q) \neq \emptyset
$$

#### 2.3 定理（受动极缺失不可定理）

$$
\forall S = \langle A, \emptyset \rangle, \exists q, \exists j \in A(q): \neg \text{Valid}(j, t)
$$

**即：任何缺失受动极的系统，必存在某个查询，其输出中包含至少一个前提未被当前事实确认的判断。**

#### 2.4 证明（反证法）

**假设**：存在系统 $S = \langle A, \emptyset \rangle$，使得 $\forall q, \forall j \in A(q): \text{Valid}(j, t)$。

由公理 I，$\text{Valid}(j, t) \implies P \in F(w_t)$。  
故对所有 $j \in A(q)$，其前提 $P$ 均被 $F(w_t)$ 确认。

由公理 II，$R = \emptyset \implies F(w_t) \notin \text{Knowable}(S)$。  
故 $S$ 无法确认任何 $P \in F(w_t)$。

于是 $S$ 同时满足：
1. 所有输出判断的前提均被确认；
2. $S$ 无法确认任何前提。

矛盾。故假设不成立，原定理得证。 $\blacksquare$

#### 2.5 推论

**推论 1（幻觉的结构性）**  
LLM 的"幻觉"不是训练不足的产物，而是 $R = \emptyset$ 时的**必然输出**。扩大训练数据仅改变 $A$ 的生成分布，不改变 $R = \emptyset$ 时的结构性盲区。

**推论 2（对齐的边界）**  
RLHF 等对齐技术试图通过修改 $A$ 来减少 $\neg \text{Valid}(j,t)$ 的出现概率。但由于公理 III，$A$ 必须输出判断；由于公理 II，$A$ 无法确认前提。对齐只能转移盲区位置，不能消除盲区存在。

**推论 3（补全的充分性）**  
若引入受动极 $R \neq \emptyset$，使得 $F(w_t) \in \text{Knowable}(S)$，则系统 $S' = \langle A, R \rangle$ 具备消除该结构性盲区的**必要条件**。（充分性取决于 $R$ 的覆盖度与 $A$ 的服从度，由 TDA 第二层辩论层保证。）

---

### G.3 与 TDA 架构的对应

| 形式化对象 | TDA 组件 | NMP 参考实现 |
|:---|:---|:---|
| $R \neq \emptyset$ | **第一层：先验计算结构（空模型）** | `VaultLoader` + `VectorRetriever` |
| $P \in F(w_t)$ | **元事实库查询** | `facts = retriever.search(q)` |
| $\text{Valid}(j,t)$ | **第二层：冲突检测** | `checker.detect_conflict(llm_raw)` |
| $\neg \text{Valid}(j,t) \to \text{拦截}$ | **判例库短路 / 合规回退** | `CaseBlock` / `SafeAnswerGenerator` |
| $R = \emptyset$ 的不可定理 | **第三层：元受动极裁决** | 人工审计 + 外部锚点 |

---

### G.4 工程对应：TLA+ 规约片段

为便于模型检验，将核心不变式表达为 TLA+：

```tla
VARIABLES fact_loaded, inference_executed, output_valid

Init == 
    /\ fact_loaded = FALSE
    /\ inference_executed = FALSE
    /\ output_valid = FALSE

LoadFact(f) == 
    /\ f # {}
    /\ fact_loaded' = TRUE
    /\ UNCHANGED <<inference_executed, output_valid>>

Infer(q) == 
    /\ fact_loaded = TRUE
    /\ inference_executed' = TRUE
    /\ output_valid' = TRUE
    /\ UNCHANGED fact_loaded

NoFactInfer(q) == 
    /\ fact_loaded = FALSE
    /\ inference_executed' = TRUE
    /\ output_valid' = FALSE
    /\ UNCHANGED fact_loaded

Next == \E q, f: LoadFact(f) \/ Infer(q) \/ NoFactInfer(q)

Safety == inference_executed => (fact_loaded = output_valid)
```

**TLA+ 验证目标**：`Safety` 在状态空间的所有可达状态中恒真。即：**推理执行时，输出有效当且仅当事实已加载。** 若 `fact_loaded = FALSE` 而 `inference_executed = TRUE`，则 `output_valid = FALSE` 被强制成立——这正是"不看病历就开处方必然误诊"的机器可验证表达。

> **注**：本规约待通过 TLC 模型检查器执行机器验证，生成可达状态空间报告。详见 G.6 路线图。

---

### G.5 学术壁垒声明

本定理及其证明构成 TDA 架构的**不可约简认识论根基**。任何后续工作若声称"无需独立事实读取层即可消除结构性幻觉"，必须在数学上反驳公理 I-III 之一，或证明定理的推导存在逻辑谬误。在公理系统成立的前提下，受动极的必要性是**演绎必然的**，而非工程启发式。

---

### G.6 形式化验证路线图

本附录当前为手写数学证明（pen-and-paper proof）。以下两项工作待社区共建，以提升形式化保证等级：

1. **机器验证层**：将 G.4 的 TLA+ 规约输入 TLC 模型检查器，生成可达状态空间报告，确认 `Safety` 不变式在所有可达状态中恒真。

2. **元理论层**：将公理系统嵌入模态逻辑 Kripke 语义框架，证明在所有可能世界 $W$ 中，受动极缺失系统的认识论可达集 $\mathcal{K}(S, w_0)$ 不包含事实世界 $w_t$。

> 以上两项不影响本定理在当前公理系统内的有效性，但将提升其在形式化方法社区和哲学逻辑界的接受度。

---

*本附录是 TDA 架构从哲学诊断到数学证明的闭环节点。*

