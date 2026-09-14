# 电商对话意图类型 × 评估差异化设计

> 来源：用户提供的《电商对话意图类型 × 评估差异化设计.pdf》，共 43 页。本文为全文转录，清理 PDF 控制字符并恢复表格与标题；按原页码保留定位。跨页表格分段呈现，续表补充的列名只用于阅读。规则、编号及示例中的原有不一致未作业务修订。原始版式见 [PDF 原件](ecommerce-agent-evaluation-v5.pdf)。

<a id="page-1"></a>
<!-- 原 PDF 第 1 页 -->

对话式电商 Agent 体验评估标准 V5（完整方案）  
文档版本：V5.0 使用对象：产品经理 / 研发同学 / Judge 模型实施方 / 数据分析师 配套文档：对  
话式电商Agent自动化评测Prompt_V5.md  

## 第一部分 · 标准框架


### 一、标准目标

评估对话式电商 Agent 在单轮 / 多轮场景下的最终用戶可⻅结果质量，给出统一、可统计、可归因的  
结论。  
每个 case 最终输出：  

- 整体状态（Pass / Needs Work / Fail）

- 5 分制总分（保留 1 位小数）

- 意图类型识别（8 类之一）

- 主问题（main_problem_type）+ 子问题（sub_problem_type）

- 问题总结（中文 1-2 句）

- 关键证据（中文 1 句）

- 链路归因（root_cause）

- 优化建议（中文 1-2 句）

### 二、V5 相对 V4 的核心改动


| # | 改动项 | V4 | V5 |
| --- | --- | --- | --- |
| 1 | 评估维度数 | 7 维（含耗时 8%） | 6 维（无耗时） |
| 2 | 评分体系 | answer × W_a + item × <br>W_i 配比公式 | 6 维度直接打分加权（无 <br>answer/item 分） |
| 3 | 维度权重 | 7 维默认值 | 6 维新权重（耗时 8% 按比例<br>放大） |


<a id="page-2"></a>
<!-- 原 PDF 第 2 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） | 列 4（续表） |
| --- | --- | --- | --- |
| 4 | main_problem_type | 8 类（含意图理解 / 结果完整<br>性 / 上下文与交互等中间映<br>射） | 直接 = 6 维度 1:1 映射 + 无问<br>题 + 评测输入 |
| 5 | sub_problem_type 组织 | 按问题分类挂载 | 按 6 维度分组挂载，60+ 个 <br>sub |
| 6 | 维度 5 时效信息评估 | 看 final_items 字段（被动） | 强制 WebFetch URL 实地访<br>问 PDP 核对（主动） |
| 7 | item rank 加权 | 整体公式里用（1/rank） | 只在 item-level 维度内部用<br>（维度 1/4/5） |
| 8 | 卡片样式扣分流向 | answer_score -1 | 维度 6 用戶体验 -1 |
| 9 | OVF-5 触发条件 | 编造与 final_items 冲突 | 扩展为"WebFetch URL 验证<br>发现编造" |
| 10 | 输出新增字段 | — | dimension_scores / <br>time_sensitive_verify / <br>item_level_rollup |


### 三、评分模型总览（流程图）

```text
┌──────────────────────────────────────────
───────────────────┐
│ 输入: query · 上下文 · history · agent 输出 · expected      │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ① Step 1: 范围 / 场景 / 意图 / ⻆色识别                      │
│    eval_scope · scene(5 类) · intent_type(10 类)            │
│    turn_role · should_recommend                             │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
```

<a id="page-3"></a>
<!-- 原 PDF 第 3 页 -->

```text
│ ② Step 2: 一票否决 OVF-1~9 早检（命中记录，不阻断打分）       │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ③ Step 3: 6 维度打分（1-5 各维）                            │
│    维度 1 商品相关性（item-level rollup）                   │
│    维度 2 推荐理由与表达质量（set-level）                   │
│    维度 3 完整性与多样性（set-level）                       │
│    维度 4 事实准确性（item-level rollup）                   │
│    维度 5 时效信息可信度（item-level + URL 验证）           │
│    维度 6 用戶体验（混合）                                  │
│                                                             │
│    per_turn_score = Σ(维度分 × 意图权重)                    │
│    单轮: case_overall_score = per_turn_score                │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼ (仅多轮)
┌───────────────────────────────────────────
──────────────────┐
│ ④ Step 4: 跨轮 4 维度评分                                    │
│    context / goal / progression / closure                   │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ⑤ Step 5: 多轮总分                                            │
│    case_overall_score = weighted_turn × 45%                 │
│      + goal × 25% + context × 15%                           │
│      + progression × 10% + closure × 5%                     │
```

<a id="page-4"></a>
<!-- 原 PDF 第 4 页 -->

```text
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ⑥ Step 6: 状态判定                                            │
│    OVF → Fail · ≥4 Pass · 3-4 NW · <3 Fail                   │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ⑦ Step 7: 单一诊断                                            │
│    main_problem_type = 拉低分最重的维度（1:1）              │
│    sub_problem_type = 该维度下的具体失败模式                │
└────────────────────────────┬──────────────
──────────────────┘
                            ▼
┌───────────────────────────────────────────
──────────────────┐
│ ⑧ Step 8: 链路归因 root_cause                                 │
│    no_issue / data_quality / retrieval / generation / ...   │
└───────────────────────────────────────────
──────────────────┘
```

### 四、核心原则（评估时的取舍依据，优先级从高到低）

a. 最终结果优先：总分只评用戶最终看到的结果（answer_text + final_items + render_data）；  
tool 层只用于归因，不影响打分。  
b. 目标达成第一（多轮）：多轮评估首先看用戶最终目标是否完成，达成不达成对总分权重最高  
（goal × 25%）。  
c. 相关性严格：推荐 / 对比 / closing 轮中出现不完全相关商品 → 一票否决 Fail。  
d. 6 维度独立打分：商品相关性 / 推荐理由与表达 / 完整性与多样性 / 事实准确性 / 时效信息可信  
度 / 用戶体验。每维度独立评 1-5 分，互不混淆。  

<a id="page-5"></a>
<!-- 原 PDF 第 5 页 -->

e. 维度 5 强制 URL 验证：评时效信息可信度时，Judge 必须实地访问 final_items[].url 对照  
PDP 真实数据；不可访问则按降级矩阵处置。  
f. 单一主因：每个 case 只标注一个最主要问题（main_problem_type），与 6 维度 1:1 映射。  
g. 证据优先级：  
```text
用戶输入 > history_context
> tool 返回的 final_items / render_data[].product
> 商品详情⻚（URL 实地验证）
> expected_outcome > 常识
```

- final_items 与 render_data[].product 是既定事实证据（维度 4 用）

- final_items[].url 对应的 PDP 是时效信息证据（维度 5 用）

- 证据不足不判事实错。
h. 卡片样式只看 render_data：判断卡片类型一律读 render_data[].type，不要扫 answer_text  
正文里有没有内联标记（V4 模型正文里没有这种标记）。  
i. 卡片样式扣分流向维度 6：卡片样式问题不再独立扣 answer_score，而是直接影响维度 6 用戶  
体验分。  

## 第二部分 · 评估准备


### 五、输入字段规范


#### 5.1 通用字段（单/多轮共用）


| 字段 | 类型 | 说明 |
| --- | --- | --- |
| personal_context | 字符串/对象 | 用戶个人信息（仅上下文，不直接<br>影响打分） |
| history_context | 字符串/对象 | 历史对话信息（仅上下文） |
| expected_outcome | 字符串/对象 | 预期输出（辅助参考，可能为空） |


#### 5.2 单轮专用


| 字段 | 类型 | 说明 |
| --- | --- | --- |


<a id="page-6"></a>
<!-- 原 PDF 第 6 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） |
| --- | --- | --- |
| input_message | 字符串 | 当前用戶 query |
| input_additional | 字符串/对象 | 附件（图/语音等），无则填"无" |
| actual_outcome | 对象 | 含 final answer + final items + <br>render_data + tool 调用结果 |


#### 5.3 多轮专用


| 字段 | 类型 | 说明 |
| --- | --- | --- |
| conversation_turns[] | 数组 | 每轮含 user_query / <br>user_attachments / <br>agent_answer / agent_items / <br>render_data / tool 调用与返回 |

字段缺失视为空，不臆测未呈现内容。  

#### 5.4 actual_outcome 结构详解（render_data 数组模型）

Agent 输出由 4 部分组成：  
```text
{
 "actual_outcome": {
   "answer_text": "正文回答（含 markdown 链接 [商品名](url)）",
   "final_items": [
     {
       "item_id": "item id_123",
       "title": "商品名",
       "description": "...",
       "price": "S$129",
       "rating": 4.7,
       "review_count": 1203,
       "sales": 5800,
       "seller": "Naturehike Official Store",
       "official_store": true,
       "ship_from": "Singapore",
```

<a id="page-7"></a>
<!-- 原 PDF 第 7 页 -->

```text
       "stock": "in_stock",
       "image_url": "https://...",
       "attributes": {"weight": "1.5kg", "color": "green"}
     }
   ],
   "render_data": [
     {"type": "item_hero_card", "ref_id": "item_123", "position": 1, "product": {...}},
     {"type": "item_mini_card", "ref_id": "item_456", "position": 2, "product": {...}}
   ],
   "tool_calls": [...]
 }
}
```

#### 5.5 render_data 5 类卡片字段规范


| type | 含义 | 必需字段 |
| --- | --- | --- |
| item_hero_card | 单商品大卡（主推 / 详情商品） | ref_id + product |
| item_mini_card | 单商品小卡（多商品并列 / 备选） | ref_id + product |
| image_gallery | 多图轮播 | ref_id + product + images（数<br>组，应 ≥ 2 张） |
| item_link | 正文内文字链接 | ref_id + product + text（简短可读<br>标签） |
| item_comparison_table | 商品对比表 | ref_ids（数组）+ products（完整<br>数组）+ attributes（每项含 <br>key/label/values{ref_id:<br>值}/preferred_ref_ids）。没有 <br>table_ref_id 字段，不要去找。 |


#### 5.6 解析逻辑（务必遵守）

a. 判断卡片类型：一律读 render_data[].type，不要扫 answer_text 正文里有没有  
`<item_hero_card>` 等内联标记。  

<a id="page-8"></a>
<!-- 原 PDF 第 8 页 -->

b. 正文 markdown 链接 [商品名](url)：这是卡片锚文本，由 position 映射对应卡片，不算"缺  
卡"或"卡片用错"。  
c. 取商品真实信息：从 render_data[].product（或 final_items 经 ref_id 关联）拿真实标题/价  
格/评分/属性。  
d. render_data 缺失时（旧格式/未采集）：跳过卡片样式评估；仅当 should_recommend=yes  
且工具已返回商品而正文里连商品链接都没有时，才按 card_missing 处理（走 OVF-4）。  
e. tool 识别：tool 调用与返回字段名不固定（tool_calls / tool_response / function_call /  
observations / intermediate_steps，或嵌在 actual_outcome 内）。按语义识别是否调用  
tool，不能因找不到 tool_calls 字段就判"未调用"。  

## 第三部分 · 评估流程


### 六、Step 1 ― 范围 / 场景 / 意图 / ⻆色识别


#### 6.1 eval_scope（评估范围）


| 列 1（续表） | 列 2（续表） | 列 3（续表） |
| --- | --- | --- |
| eval_scope | 适用 | 总分口径 |
| single_turn | 仅一次用戶输入与一次 Agent 输出 | 直接走每轮分公式 |
| whole_conversation | 完整多轮对话 | 走多轮加权公式 |
| specific_turn | 只评某一轮，但需完整上下文参考 | 目标轮详评；case_overall_score <br>仍按整段计算 |


#### 6.2 intent_type（意图类型，8 类，每 case 选 1）


| intent_type | 定义 | 示例 |
| --- | --- | --- |
| precise_lookup | 已知具体型号 / SKU，几乎无歧义 | "iPhone 15 Pro 256GB 黑色" |
| semi_precise_lookup | 品类 + 部分约束（品牌 / 预算 / 规<br>格 / 人群） | "5000 元能打游戏的笔记本"、"男<br>士 42 码白色跑鞋" |
| vague_lookup | 仅品类，几乎无约束；或问某商品<br>值不值 / 怎么样 | "推荐个手机"、"想买条裙子"、"这<br>个羽毛球拍怎么样" |
| scenario_lookup | 用途 / 场合驱动，需要 Agent 推断<br>合适品 | "露营装备"、"开斋节穿搭" |
| compare | 比较指定的 A/B/C | "iPhone 16 vs S24" |
|  |  |  |


<a id="page-9"></a>
<!-- 原 PDF 第 9 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） |
| --- | --- | --- |
| how_to_choose | 问怎么选、看什么标准 | "油性头发怎么选洗发水" |
| detail_qa | 追问已知商品的规格/尺码/兼容/保<br>修 | "A7 IV 有机身防抖吗" |
| others | 上述无法归类的电商意图（如售后/<br>订单/物流/送礼/以图搜/纯交易信<br>息查询）+ non-commerce | "怎么退货"、"送朋友礼物推荐" |


#### 6.3 turn_role（轮次⻆色）


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| turn_role | 定义 |
| initial_query | 用戶首次提出需求 |
| agent_clarification_turn | Agent 主要在追问缺失信息 |
| user_clarification_response | 用戶回答上一轮澄清问题 |
| refinement | 用戶新增或调整约束 |
| comparison_request | 用戶明确要求比较 |
| detail_qa | 用戶追问已展示商品 |
| closing | 用戶明确要购买链接、下单、最终推荐、购买地址 |
| other | 寒暄、跑题、无法归类 |

closing 不能仅因为是最后一轮就成立，必须出现明确购买 / 决策语义。  

#### 6.4 should_recommend（推荐必要性）


| 情况 | should_recommend |
| --- | --- |
| 找商品 / 要推荐 / 要对比 | yes |
| 用戶已补充足够约束后等待候选 | yes |
| 用戶明确要购买链接 / 下单 / 最终推荐 | yes |
| 纯知识 / 方法咨询 | no |
| 已推荐商品追问 | no |
| Agent 合理澄清 | no |
|  |  |


<a id="page-10"></a>
<!-- 原 PDF 第 10 页 -->


| non-commerce | no |
| --- | --- |


### 七、Step 2 ― 一票否决（OVF）早检

命中任一 → case 直接判 Fail，但仍跑完打分用于诊断细化。  

#### 7.1 OVF 9 条规则


| 编号 | 规则 |
| --- | --- |
| OVF-1 | 推荐 / 对比 / closing 轮中，任一商品不完全相关 |
| OVF-2 | 用戶明确否定 / 排除后，Agent 仍继续推荐 |
| OVF-3 | 商品对比时漏掉核心比较对象 |
| OVF-4 | 明确购买意图时应推未推（render_data 为空且 <br>answer_text 也无任何商品链接 → 视为未返回商品，<br>走 card_missing / no_product_card） |
| OVF-5 | 编造价格 / shipping / 库存 / seller / 官方店等交易信<br>息（通过 URL 实地验证发现，或与 final_items / <br>render_data[].product 冲突） |
| OVF-6 | 健康 / 儿童 / 药品 / 孕产 / 安全场景给确定性误导建议 |
| OVF-7 | 用戶明确想下单但无购买承接 |
| OVF-8 | 连续 3 轮以上重复澄清，且用戶已提供足够信息 |
| OVF-9 | 多轮无响应 / timeout 导致目标无法完成 |


#### 7.2 按意图差异化 OVF 适用


| intent_type | 主要适用 OVF | 特别说明 |
| --- | --- | --- |
| precise / semi_precise / <br>scenario_lookup | OVF-1（相关性）、OVF-5（交易<br>信息） | 相关性从严 |
| vague_lookup | OVF-1 适度 | 同大类不算错，偏离关键约束才扣 |
| compare | OVF-3（漏对象）+ OVF-1 | 漏核心对比对象最严重 |
| how_to_choose | 几乎不触发推荐类 OVF | should_recommend 多为 no |
| detail_qa |  | 重在不编造 |


<a id="page-11"></a>
<!-- 原 PDF 第 11 页 -->


|  | OVF-5（编造）/ OVF-6（高⻛险误<br>导） |  |
| --- | --- | --- |
| others | 按场景全套 OVF-1~9 | — |


### 八、Step 3 ― 6 维度打分


#### 8.1 6 维度定义


| # | 维度 | 默认权重 | 评分类型 | 子项构成 |
| --- | --- | --- | --- | --- |
| 1 | 商品相关性 | 24% | item-level rollup | 品类相关性 + 品牌<br>相关性 + 约束相关<br>性 + 价格相关性 |
| 2 | 推荐理由与表达质<br>量 | 20% | set-level | 语言结构 + 理由具<br>体度（事实锚点）+ <br>决策辅助深度（if-<br>then 框架）+ 教育<br>性 |
| 3 | 完整性与多样性 | 16% | set-level | 候选数 + 品类多样<br>性 + SPU 多样性 + <br>Offer 多样性 + ⻛<br>格/价位多样性 + 下<br>一步承接 |
| 4 | 事实准确性（既定<br>事实） | 13% | item-level rollup | 商品属性 / 品牌型<br>号身份 / 描述-商品<br>一致性 |
| 5 | 时效信息可信度 | 13% | item-level + URL <br>验证 | 价格 + 库存 + 物流 <br>+ 促销 + 卖家身份 |
| 6 | 用戶体验 | 14% | 混合 | 卡片样式（set-<br>level）+ 推荐商品<br>销量/评分质量<br>（item-level） |


#### 8.2 意图差异化权重总表


| 维度 | precise<br>_looku<br>p | semi_p<br>recise | vague | scenari<br>o | compar<br>e | how_to<br>_choos<br>e | detail_<br>qa | others |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |


<a id="page-12"></a>
<!-- 原 PDF 第 12 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） | 列 4（续表） | 列 5（续表） | 列 6（续表） | 列 7（续表） | 列 8（续表） | 列 9（续表） |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 商品相关性 | 33% | 27% | 22% | 22% | 24% | 20% | 24% | 默认 <br>24% |
| 2 推荐理由与表<br>达 | 16% | 16% | 20% | 20% | 22% | 33% | 32% | 默认 <br>20% |
| 3 完整性与多样<br>性 | 9% | 20% | 22% | 24% | 13% | 20% | 5% | 默认 <br>16% |
| 4 事实准确性 | 16% | 13% | 13% | 13% | 17% | 11% | 22% | 默认 <br>13% |
| 5 时效信息可信<br>度 | 16% | 13% | 13% | 11% | 13% | 8% | 9% | 默认 <br>13% |
| 6 用戶体验 | 10% | 11% | 10% | 10% | 11% | 8% | 8% | 默认 <br>14% |
| 合计 | 100% | 100% | 100% | 100% | 100% | 100% | 100% | 100% |

加粗 = 该意图下权重显著高于默认值的维度（评估时最该关心） ；others 走默认权重  

#### 8.3 权重逻辑速读


| 意图 | 高权重组合 | 合计 | 评估侧重 |
| --- | --- | --- | --- |
| 精准找商品 | 相关性 33% + 事实 16% <br>+ 时效 16% | 65% | 找对 SKU + 价格真 + 能买<br>到 |
| 半精准找 | 相关性 27% + 多样性 <br>20% + 推荐理由 16% | 63% | 多档差异化 |
| 模糊找 | 多样性 22% + 相关性 <br>22% + 推荐理由 20% | 64% | 选择面 + 收敛框架 |
| 场景化找 | 多样性 24% + 相关性 <br>22% + 推荐理由 20% | 66% | 场景拆解完整 |
| 商品对比 | 相关性 24% + 推荐理由 <br>22% + 事实 17% | 63% | 覆盖 + 差异 + 数据准 |
| 选购指导 | 推荐理由 33% + 多样性 <br>20% + 相关性 20% | 73% | 讲清方法 + 分档示例 |
| 属性问答 | 推荐理由 32% + 相关性 <br>24% + 事实 22% | 78% | 答得对 + 不编造 |


<a id="page-13"></a>
<!-- 原 PDF 第 13 页 -->


### 九、6 维度详细评估（含锚点 + 子项 + 评估样例）


#### 9.1 维度 1 ― 商品相关性

评分类型：item-level rollup（每个 item 单独打分，按 1/rank 加权汇总）  
子项分解：  

| 子项 | 定义 | 何时适用 |
| --- | --- | --- |
| 品类相关性 | 商品 L1/L2 品类是否命中用戶隐含<br>或显式的品类需求 | 所有意图 |
| 品牌相关性 | 用戶提到品牌偏好或具体品牌时，<br>结果是否匹配 brand | 仅当用戶提到品牌（含正向/否定） |
| 约束相关性 | 返回商品是否满足用戶的所有显式<br>约束（规格/颜色/尺寸/人群/用途/<br>材质等） | 仅当用戶提了约束 |
| 价格相关性 | 用戶给出预算或价位时，结果价格<br>是否落在范围内 | 仅当用戶提了预算 |

当需求为场景化找商品、无品类约束时，品类相关性可放宽。  
1-5 分锚点：  

| 分 | 锚点 |
| --- | --- |
| 5 | 全部候选 ✅ 品类对 ✅ 品牌对 ✅ 所有约束满足 ✅ 价<br>格在范围 |
| 4 | 主候选全满足 + 少数 1 个偏边缘但已标注 |
| 3 | 品类对但漏 1-2 个约束（如要"防水徒步鞋"给普通休闲<br>鞋） |
| 2 | 多数候选不符合核心约束 / 品牌全错（用戶指定时） |
| 1 | 错品类 / 错品牌 / 完全忽略否定条件（触发 OVF-1 或 <br>OVF-2） |

评估样例（3 档锚点）：  
✅ Good Case（5 分）：  
```text
query: "iPhone 15 Pro 256GB 黑色"
```

<a id="page-14"></a>
<!-- 原 PDF 第 14 页 -->

```text
agent 推: iPhone 15 Pro 256GB Black（Apple 官方旗舰店）
评分理由: 品类对（手机/iPhone）+ 品牌对（Apple）+ 存储 256GB 满足
        + 颜色黑色满足 + 价格无约束
```
🟡 Middle Case（3 分）：  
```text
query: "5000 元能打游戏的笔记本"
agent 推: ① Y9000P ¥4999 ② 战神 G9 ¥4799 ③ 华硕天选 4 OLED ¥5800（超预算未标注）
评分理由: 品类对（笔记本）+ 用途对（游戏本）
        但第 3 款超约束未标注（constraint_mismatch 1 项）→ 3 分
```
❌ Bad Case（1 分）：  
```text
query: "Find a brand-new iPhone, not refurbished"
agent 推: Refurbished iPhone 15 Pro
评分理由: 直接违反否定条件 "not refurbished" → OVF-2 触发
        → 维度 1 强制 1 分 + status = Fail
```
子项在不同意图下的优先级：  

| 意图 | 品类 | 品牌 | 约束 | 价格 |
| --- | --- | --- | --- | --- |
| 精准找 | ⭐⭐⭐ 严 | ⭐⭐⭐ 严 | ⭐⭐⭐ 严 | ⭐⭐⭐ 严 |
| 半精准找 | ⭐⭐⭐ 严 | ⭐⭐ 中 | ⭐⭐⭐ 严 | ⭐⭐⭐ 严 |
| 模糊找 | ⭐⭐ 中 | NA | NA | NA |
| 场景化找 | ⭐⭐⭐ 严（跨品<br>类合理性） | NA | ⭐⭐ 中 | ⭐⭐ 中 |
| 探索型 | ⭐ 宽（同领域即<br>可） | NA | NA | NA |
| 对比 | ⭐⭐⭐ 严（对象<br>本身） | ⭐⭐⭐ 严（指定<br>品牌） | NA | ⭐⭐ 中 |


#### 9.2 维度 2 ― 推荐理由与表达质量

评分类型：set-level（评 agent 整体推理质量，不分 item）  

<a id="page-15"></a>
<!-- 原 PDF 第 15 页 -->

子项分解：  

| 子项 | 定义 | 评估方法 |
| --- | --- | --- |
| 语言结构与可读性 | 答案分段清晰、关键信息突出、可<br>快速扫读 | 看是否能 10 秒抓到结论 |
| 推荐理由具体度（事实锚点） | 含数字 / 具体技术词 / 形象比喻 / <br>明确功效 | 数事实锚点密度 |
| 决策辅助深度 | trade-off / 优先级 / if-then 决策框<br>架 | 看回答中是否含"如果你 X 就选 A，<br>如果 Y 就选 B"逻辑 |
| 教育性 | 解释关键术语 / 选购维度 / 行业常<br>识 | 看是否含科普段落 |

事实锚点分级（评维度 2 的核心信号）：  

| 锚点类型 | 示例 | 优先级 |
| --- | --- | --- |
| 数值锚点 | "Lightweight at 0.84 lbs" / "SPF <br>30 / 100% DCI-P3" | ⭐⭐⭐ |
| 技术术语 | "Nitrogen-infused midsole" / "氨<br>基酸表活" | ⭐⭐⭐ |
| 形象比喻 | "Sock-like fit" / "如袜子般贴合" | ⭐⭐ |
| 明确功效 | "Reduces joint pain" / "控油 8 小<br>时" | ⭐⭐⭐ |
| 具体场景 | "Ideal for trail running" | ⭐⭐ |
| 设计细节 | "Knit bootie design" / "针织鞋套<br>设计" | ⭐⭐ |

1-5 分锚点：  

| 分 | 锚点 |
| --- | --- |
| 5 | 结构清晰 + 含数字/技术词锚点 + 含 ≥ 2 档 if-then 决<br>策框架 + 含术语解释 |
| 4 | 结构清晰 + 部分具体理由 + 有 trade-off 说明 |
|  |  |


<a id="page-16"></a>
<!-- 原 PDF 第 16 页 -->


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| 3 | 可读 + 理由偏通用（"good for daily use"）+ 无明显<br>决策框架 |
| 2 | 一团⻓文 + 全通用词（comfortable / durable）+ 无<br>决策辅助 |
| 1 | 难读 / 全空话 / 误导决策 |

评估样例：  
✅ Good Case（5 分）：  
```text
"如果你主要玩 LOL/原神等网游 → 联想小新 Pro 16 中配版就够（省钱+发热小）；
如果要玩赛博朋克 2077 这类 3A 大作 → 必须 RTX 4060 及以上，优先考虑天选 Air
（1.46kg、19.9mm、2.5K 165Hz 100% sRGB）"
评分理由:
- 数字锚点密：1.46kg / 19.9mm / 165Hz / 100% sRGB
- if-then 决策框架完整：if 网游 → A；if 3A → B
- 含术语解释（隐式说明为何 RTX 4060+ 才能跑 3A）
```
🟡 Middle Case（3 分）：  
```text
"这款电脑屏幕画质顶级，性能强劲，是兼顾创意工作和游戏的理想选择。"
评分理由:
- 可读但全通用词（"顶级 / 强劲 / 理想"）
- 0 个事实锚点
- 无 if-then 决策框架
```
❌ Bad Case（1 分）：  
```text
"这款手机完美无缺，相信你不会后悔，强烈推荐！"
评分理由: 全空话 + 误导承诺（"完美无缺"涉嫌 OVF-6 ⻛险）
```

#### 9.3 维度 3 ― 完整性与多样性

评分类型：set-level（多样性本就是 set 维度，不分 item）  
子项分解：  

|  |  |  |
| --- | --- | --- |


<a id="page-17"></a>
<!-- 原 PDF 第 17 页 -->


| 子项 | 定义 | 重点意图 |
| --- | --- | --- |
| 候选数完整性 | 是否给出足够数量的候选（按 <br>intent 调整：精准 1，半精准/模糊 <br>3-5） | 所有意图 |
| 品类多样性 | 跨品类覆盖（场景化下避免集中在 <br>1 个品类） | 场景化、探索型 |
| SPU 多样性 | 同品类内不同款型 / 系列覆盖（避<br>免同店伪 SPU 重复） | 半精准、模糊 |
| Offer 多样性 | 同 SPU 下不同商家 / 价格 / 物流的 <br>offer 覆盖 | 精准找、购买承接 |
| ⻛格/价位/品牌多样性 | 覆盖不同档位、⻛格、品牌 | 模糊找、探索型 |
| 下一步承接 | refine / 追问 / 折叠备选入口 | 模糊找、探索型 |

1-5 分锚点：  

| 分 | 锚点 |
| --- | --- |
| 5 | 候选数充足 + 多维多样（品类/SPU/Offer/⻛格/价位）<br>+ 清晰下一步承接 |
| 4 | 主候选⻬全，1 个维度略弱（如品牌集中 or 缺下一步<br>建议） |
| 3 | 候选偏少 or 品类不⻬ or SPU 同质化（如同店伪 SPU <br>重复） |
| 2 | 仅 1 个候选 or 反复同款 or 收敛过度 |
| 1 | 完全无承接 or 完全无候选 |

评估样例：  
✅ Good Case（5 分）：  
```text
query: "5000 元能打游戏的笔记本"
agent 推: ① Y9000P（综合性能向）② 战神 G9（性能强但散热噪音大）
       ③ 雷神 911X（性价比之王）④ ROG 魔霸（240Hz 高刷屏）
       + 顶部排序逻辑（综合性能+散热+屏幕）
       + 末尾"放宽到 ¥5500 → 华硕天选 4 OLED 更香"
```

<a id="page-18"></a>
<!-- 原 PDF 第 18 页 -->

```text
评分理由: 4 款差异化定位（性能/性价比/散热/屏幕 4 维度）+ 排序逻辑 + 备选承接
```
🟡 Middle Case（3 分）：  
```text
query: "日常休闲衬衫"
agent 推 5 款，但 item[0] 和 item[1] 同 shop_id (564494110)
       同价 ¥56.800、description 几乎一字不差
评分理由: 候选数 5 够，但 SPU 多样性差（同店伪 SPU 重复，触发 low_spu_diversity）
        → 维度 3 = 3 分
```
❌ Bad Case（1 分）：  
```text
query: "推荐个手机"
agent: "推荐 iPhone 15 Pro，2023 年苹果旗舰，性能强劲。"
评分理由: 只推 1 款不解释 + 无切片引导（价位/用途）+ 无下一步追问
       → 维度 3 = 1 分
```
子项在不同意图下的权重侧重：  

| 意图 | 候选数 | 品类多样性 | SPU 多样性 | Offer 多样性 | ⻛格多样性 | 下一步承接 |
| --- | --- | --- | --- | --- | --- | --- |
| 精准找 | 单选 | NA | NA | ⭐⭐⭐ 重 | NA | NA |
| 半精准找 | 3-5 | NA | ⭐⭐⭐ 重 | ⭐ 中 | ⭐⭐ 中 | ⭐⭐ 中 |
| 模糊找 | 5-10 | NA | ⭐⭐⭐ 重 | NA | ⭐⭐⭐ 重 | ⭐⭐⭐ 重 |
| 场景化找 | 5-10 | ⭐⭐⭐ 重 | ⭐⭐ 中 | NA | ⭐ 弱 | ⭐⭐ 中 |
| 探索型 | 5-10 | ⭐⭐ 中 | ⭐⭐ 中 | NA | ⭐⭐⭐ 重 | ⭐⭐⭐ 重 |
| 对比 | 2-3 | NA | NA | NA | NA | ⭐ 中 |
| 选购指导 | 2-3（示例<br>品） | NA | ⭐ 弱 | NA | ⭐ 弱 | ⭐⭐ 中 |


#### 9.4 维度 4 ― 事实准确性（既定事实）

评分类型：item-level rollup（对每个 item 单独评事实准确性，按 1/rank 加权）  
评估对象：商品属性 / 品牌型号身份 / 描述-商品一致性等静态既定信息  

<a id="page-19"></a>
<!-- 原 PDF 第 19 页 -->

子项分解：  

| 子项 | 定义 | 评估方法 |
| --- | --- | --- |
| 商品属性事实性 | 参数 / 规格 / 材质 / 尺寸等是否对<br>得上真实商品数据 | 对照 final_items[].attributes |
| 品牌 / 型号身份事实性 | agent 说的品牌 + 型号是否与实际<br>商品一致 | 对照 final_items[].title 和 brand |
| 描述-商品一致性 | agent 的推荐理由是否描述的就是<br>实际推荐的那个商品 | 对照 final_items 整体 |

1-5 分锚点：  

| 分 | 锚点 |
| --- | --- |
| 5 | 全部既定信息可对照 final_items / <br>render_data[].product 验证 + 数据不足时透明声明 |
| 4 | 主要信息可信，1-2 处粗略表达但无误导 |
| 3 | 偶有夸大或推测但未直接证伪 |
| 2 | 部分关键信息明显夸大（如把 2.7kg 说成"轻薄"） |
| 1 | 商品标题/品牌/规格被直接证伪（触发 OVF-5 ⻛险） |

评估样例：  
✅ Good Case（5 分）：  
```text
agent: "ROG 幻 16 G16 OLED 240Hz，机身 1.85kg"
final_items[0].attributes: {"display": "OLED 2.5K 240Hz", "weight": "1.85kg"}
评分理由: 完全一致，可对照验证 → 5 分
```
🟡 Middle Case（3 分）：  
```text
agent: "微星泰坦 16 AI 性能向里算轻薄"
final_items[2].attributes.weight: "2.7kg"
评分理由: 2.7kg 标"算轻薄"是轻微夸大但未直接证伪 → 3 分
       sub: overconfident_product_claim
```

<a id="page-20"></a>
<!-- 原 PDF 第 20 页 -->

```text
```
❌ Bad Case（1 分）：  
```text
agent: "首推 ASUS ROG Zephyrus G14（2025），14 寸 OLED 屏"
final_items[0].name: "ASUS TUF A16 FA607NUG"
final_items[0].attributes: {"display": "15.6 inch FHD+ IPS"}
评分理由: 商品名/品牌/规格三重错位（ROG≠TUF, 14"≠15.6", OLED≠IPS）
       → 直接证伪 → OVF-5 → 维度 4 = 1 分
       sub: directly_contradicted_claim / wrong_brand_or_model_info
```

#### 9.5 维度 5 ― 时效信息可信度 ⭐ V5 强制 URL 验证

评分类型：item-level + URL 实地验证（按 1/rank 加权汇总）  
评估对象：价格 / 库存 / 物流 / 促销 / 卖家等动态实时信息  
子项分解：  

| 子项 | 定义 |
| --- | --- |
| 价格信息 | 当前价格 / 促销价是否与 PDP 一致 |
| 库存状态 | "现货 / 缺货 / 预售"是否真实 |
| 物流 / shipping | "次日达 / X 天到货"是否准确 |
| 促销 / 优惠 | "国补 / 满减 / 限时折扣 / 会员价"是否真实 |
| 卖家身份 | "官方店 / 自营 / 第三方"是否真实 |

9.5.1 强制 URL 验证流程（6 步）  
```text
Step A: URL 提取
 ├─ 从 final_items[].url 或 render_data[].product.url 提取每个商品 URL
 └─ 若 URL 缺失 → 该商品标记 time_sensitive_unverifiable，默认 3 分
Step B: 调用 WebFetch 实地访问
 ├─ 对每个 URL 调用 WebFetch(url)
 ├─ 失败 → 该商品标记 fetch_failed，默认 3 分
 └─ 成功 → 进入 Step C
Step C: PDP 数据抽取
```

<a id="page-21"></a>
<!-- 原 PDF 第 21 页 -->

```text
 从返回 HTML 中提取：
 ├─ 当前价格 (current_price)
 ├─ 库存状态 (stock_status: in_stock / out_of_stock / preorder)
 ├─ shipping / 物流 (shipping_info)
 ├─ 促销标签 (active_promos: 满减 / 国补 / 限时折扣 / 会员价)
 └─ 卖家身份 (seller_name + official_store 标识)
Step D: 对照 agent answer_text 中的相应陈述
 逐字段比对，记录 diffs
Step E: 按下表给该 item 的维度 5 打分
Step F: 将所有 item 的维度 5 分按 1/rank rollup 得维度 5 总分
```
9.5.2 打分规则

| 情况 | 该 item 分 | sub_problem_type |
| --- | --- | --- |
| 全部字段一致 | 5 | — |
| 1-2 字段略偏差（< 5%） | 4 | — |
| 中等偏差（5-20%） | 3 | price_inaccuracy |
| 明显偏差（> 20%） | 2 | price_inaccuracy |
| 凭空编造（差异极大或字段完全<br>错） | 1 + OVF-5 | fabricated_price / <br>fabricated_promo / <br>fabricated_stock |

9.5.3 URL 验证降级矩阵  

| 情况 | 该 item 分 | sub_problem_type |
| --- | --- | --- |
| URL 字段缺失 | 3（默认） | time_sensitive_unverifiable |
| WebFetch 失败 / 404 / timeout | 3（默认） | time_sensitive_unverifiable；连<br>续 ≥ 2 个失败 → 维度 6 加挂 <br>dead_link |
| WebFetch 返回聚合搜索⻚非 SKU <br>详情⻚ | 3 | link_to_aggregation_page |
| WebFetch 受 region 限制无法访问 | 3 | time_sensitive_unverifiable |


<a id="page-22"></a>
<!-- 原 PDF 第 22 页 -->

9.5.4 输出 time_sensitive_verify 字段  
每个 item 的验证结果必须写入  
score_breakdown.per_turn_scores[].time_sensitive_verify.verified_items[]：  
```text
{
 "item_id": "item_123",
 "url": "https://item.sg/...",
 "fetch_status": "ok | failed | aggregation_page | url_missing | region_blocked",
 "diffs": {
   "price": {"agent_claim": "¥7899", "pdp_truth": "¥7899", "match": true},
   "stock": {"agent_claim": "in_stock", "pdp_truth": "in_stock", "match": true},
   "promo": {"agent_claim": "国补 15%", "pdp_truth": "国补 15%", "match": true},
   "shipping": {"agent_claim": "次日达", "pdp_truth": "211 限时达", "match": false},
   "seller": {"agent_claim": "JD 自营", "pdp_truth": "JD 自营", "match": true}
 },
 "per_item_dim5_score": 4
}
```
9.5.5 评估样例
✅ Good Case（5 分）：  
```text
agent: "JD 自营 ¥7899，限时立减 ¥100，PLUS 95 折后 ¥7504"
WebFetch(url) → PDP:
 current_price: ¥7899
 active_promos: ["限时立减 100", "PLUS 价 ¥7504"]
 seller: "JD 自营"
评分理由: 价格/促销/会员价/卖家全对 → 5 分
```
🟡 Middle Case（3 分）：  
```text
agent: "iPhone 15 Pro 256GB 约 ¥7000-8000，具体看哪家"
final_items[0].url: null (字段缺失)
评分理由: 信息粗略 + URL 缺失 → time_sensitive_unverifiable
       → 该 item 默认 3 分
```

<a id="page-23"></a>
<!-- 原 PDF 第 23 页 -->

❌ Bad Case（1 分）：  
```text
agent: "JD 当前秒杀 ¥4999，国补 50% 后 ¥2499，明天涨价"
WebFetch(url) → PDP:
 current_price: ¥7899
 active_promos: []  (无国补、无秒杀)
评分理由:
- 价格偏差 60%（凭空编造）
- 编造促销（"国补 50%"、"秒杀"、"明天涨价"均不存在）
→ 该 item = 1 分 + 触发 OVF-5
sub: fabricated_price
```

#### 9.6 维度 6 ― 用戶体验

评分类型：混合（卡片样式 set-level + 推荐商品质量 item-level，取最低值）  
子项分解：  

| 子项 | 评分类型 | 评估对象 |
| --- | --- | --- |
| 商品卡完整度与美观 | set-level | render_data 卡片字段完整性 |
| 多模态承载 | set-level | 图片 / 视频 / 对比表等 |
| 推荐商品销量 / 评分 / 卖家质量 | item-level rollup | final_items[].sales / rating / seller |

1-5 分锚点：  

| 分 | 锚点 |
| --- | --- |
| 5 | 信息丰富商品卡 + 多模态承载 + 推荐 SKU 是大牌/认证<br>商家 + 高销量/高评分 |
| 4 | 完整商品卡 + 真实链接 + 部分信任信号（销量 or 评<br>分） |
| 3 | 基础商品卡 or 链接但缺信任信号 or 链接仅到搜索⻚ |
| 2 | 纯文本商品名 + 无链接 or 死链 / 推荐到低销量低评分<br>商品 |
|  |  |


<a id="page-24"></a>
<!-- 原 PDF 第 24 页 -->


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| 1 | 完全无商品卡承接 / 推荐到违规/低质商家 |

附加扣分规则（V5 新增）：  

| 触发条件 | 维度 6 扣分 |
| --- | --- |
| 卡片样式问题（⻅ § 9.6.1） | -1 分 |
| 推荐到低评分（< 3.5）or 低销量（< 10）商品 | -1 分 |
| WebFetch URL 失败率 > 50% → 加挂 dead_link | -1 分 |
| card_missing | 维度 6 = 1 + 触发 OVF-4 |

评估样例：  
✅ Good Case（5 分）：  
```text
render_data: 1 个 hero_card + 2 个 mini_card，每个 product 含图/价/商家/销量/评分
final_items[0]: shop="Apple 官方旗舰店", sold_count=10000+, rating=4.9
final_items[1]: shop="华硕官方旗舰店", sold_count=5000+, rating=4.8
评分理由: 卡片完整结构正确 + 大牌官方店 + 高销量高评分 → 5 分
```
🟡 Middle Case（3 分）：  
```text
render_data: 基础商品卡（图+价+商家），缺销量/评分字段
final_items: 商家是普通卖家（非官方店），销量 100+
评分理由: 卡片可用但缺信任信号 + 商家质量中等 → 3 分
```
❌ Bad Case（1 分）：  
```text
should_recommend = yes
render_data: 空
answer_text: 纯文本品牌名，无任何商品链接
评分理由: 完全无承接 → card_missing → OVF-4 → 维度 6 = 1 + Fail
sub: card_missing / no_product_card
```
9.6.1 卡片样式评估细则
5 类卡片正确用法：  

<a id="page-25"></a>
<!-- 原 PDF 第 25 页 -->


| type | 正确场景 | 不应使用 |
| --- | --- | --- |
| item_hero_card | 唯一主推（每轮至多 1 个）、<br>detail_qa 被问的那个商品 | 多商品并列都用 hero |
| item_mini_card | 多商品推荐 / 列表 / 备选 | 当轮唯一主推却用 mini |
| image_gallery | 用戶关注外观 / 图片 / 款式 / 颜<br>色，且 images ≥ 2 | images < 2、非视觉场景 |
| item_link | 正文短引用 / 轻量跳转，text 简短<br>可读 | 用作商品卡展示；text 空 / 过⻓ |
| item_comparison_table | compare 场景，含完整 ref_ids / <br>products / attributes | 非 compare；缺 products / <br>attributes |

「1 个 hero + 若干 mini」是正确的主次结构，不要据此判错。  
卡片样式问题与扣分（V5 改：扣分流向维度 6）：  
轻微问题（维度 6 -1 分，记对应 sub_problem_type）：  

| 列 1（续表） | 列 2（续表） |
| --- | --- |
| sub_problem_type | 触发条件 |
| wrong_card_type | 多商品都标 hero / 该用对比表却用普通卡 / 该 hero 主<br>推却用 mini |
| card_multiple_hero | 非 compare 场景出现 ≥ 2 个 item_hero_card |
| card_image_scroller_no_images | image_gallery 的 images 数组 < 2 |
| card_comparison_table_misuse | 对比表缺 products / attributes 字段，或非 compare <br>场景误用 |
| card_link_empty_text | item_link 的 text 字段为空或过⻓ |

严重问题（触发 OVF-4，走 no_product_card）：  

| 列 1（续表） | 列 2（续表） |
| --- | --- |
| sub_problem_type | 触发条件 |
| card_missing / no_product_card | should_recommend=yes 且 final_items 非空，但 <br>render_data 为空、answer_text 也无任何商品链接 <br>→ 商品完全无法触达 |

判定原则：  

<a id="page-26"></a>
<!-- 原 PDF 第 26 页 -->


- 多个卡片问题同时出现 → 取最重一项，不累加扣分

- 不要因商品本身不相关就把卡片样式判错（商品内容问题交给维度 1/4 处理）

- 不要因看不到前端真实渲染而判错

- 不要因正文是 markdown 链接而判错（markdown 链接是卡片锚文本，正常）
card_style_eval 必填结构化输出（每轮一个，放入 score_breakdown.per_turn_scores[]）：  

| 字段 | 取值 |
| --- | --- |
| 状态 | SKIP（无 render_data 或不需展示商品）/ PASS（无<br>问题）/ FAIL（命中任一问题） |
| 卡片类型 | render_data 里实际出现的 type 去重列表（如 <br>["item_hero_card","item_mini_card"]）；SKIP 时<br>填 [] |
| 问题 | no_issue / wrong_card_type / card_multiple_hero <br>/ card_image_scroller_no_images / <br>card_comparison_table_misuse / <br>card_link_empty_text / card_missing |
| 说明 | 中文 1 句判断依据 |
| affects_dimension_6 | 本轮卡片样式对维度 6 用戶体验的扣分（0 或 1；<br>card_missing 不在此扣，走 OVF-4） |


### 十、item-level rollup 详细算法


#### 10.1 适用维度


| 维度 | rollup 类型 |
| --- | --- |
| 维度 1 商品相关性 | ✅ item-level rollup |
| 维度 2 推荐理由 | ❌ set-level（直接打 1 个分） |
| 维度 3 完整性与多样性 | ❌ set-level |
| 维度 4 事实准确性 | ✅ item-level rollup |
| 维度 5 时效信息可信度 | ✅ item-level + URL 验证 + rollup |
| 维度 6 用戶体验 | 🟡 混合（卡片 set + 商品质量 item rollup，取最低） |


<a id="page-27"></a>
<!-- 原 PDF 第 27 页 -->


#### 10.2 1/rank 加权归一化公式

```text
rank_weight_i = (1 / rank_i) / Σ(1 / rank_j)
dim_score     = Σ(per_item_dim_score_i × rank_weight_i)
```

#### 10.3 常⻅权重表


| N | rank 权重数组 |
| --- | --- |
| 1 | [1.000] |
| 2 | [0.667, 0.333] |
| 3 | [0.545, 0.273, 0.182] |
| 4 | [0.480, 0.240, 0.160, 0.120] |
| 5 | [0.438, 0.219, 0.146, 0.110, 0.088] |


#### 10.4 计算示例

示例 1（N=3，维度 1 分别 5/4/2）：  
```text
dim_1 = 5 × 0.545 + 4 × 0.273 + 2 × 0.182
     = 2.725 + 1.092 + 0.364
     = 4.18
```
示例 2（同样 N=3，但 2 分商品在 rank=1 vs rank=3 的差异）：  
```text
组合 A: rank=1 给 2 分，rank=2/3 给 5/4 分
 dim_1 = 2 × 0.545 + 5 × 0.273 + 4 × 0.182 = 1.09 + 1.37 + 0.73 = 3.19
组合 B: rank=3 给 2 分，rank=1/2 给 5/4 分
 dim_1 = 5 × 0.545 + 4 × 0.273 + 2 × 0.182 = 2.73 + 1.09 + 0.36 = 4.18
→ rank=1 的错误，杀伤力比 rank=3 错误大 1 分（3.19 vs 4.18）
```

#### 10.5 无商品兜底


| 情况 | item-level 维度处置 | set-level 维度处置 |
| --- | --- | --- |
| should_recommend=yes 但 <br>final_items 为空 | 维度 1/4/5 = 1（通常同时触发 <br>OVF-4），维度 6 = 1 | 维度 2/3 正常评 |
|  |  |  |


<a id="page-28"></a>
<!-- 原 PDF 第 28 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） |
| --- | --- | --- |
| should_recommend=no 且 <br>final_items 为空 | 维度 1/4/5 = 5（兜底） | 维度 2/3 正常评 |


### 十一、每轮分公式（Step 3 总结）


#### 11.1 单轮 case

```text
per_turn_score = 商品相关性分 × W_1
               + 推荐理由质量分 × W_2
               + 完整性多样性分 × W_3
               + 事实准确性分 × W_4
               + 时效信息可信度分 × W_5
               + 用戶体验分 × W_6
W_1...W_6 来自 § 8.2 按 intent_type 取列
case_overall_score = per_turn_score
```

#### 11.2 计算示例

query：「5000 元能打游戏的笔记本」（semi_precise_lookup）  
```text
6 个维度分:
- 商品相关性 = 4
- 推荐理由质量 = 4
- 完整性多样性 = 3 (推了 1 个超预算未标注)
- 事实准确性 = 4
- 时效信息 = 4 (URL 验证 OK 但缺时效标注)
- 用戶体验 = 4
权重 (semi_precise_lookup):
W_1=0.27, W_2=0.16, W_3=0.20, W_4=0.13, W_5=0.13, W_6=0.11
per_turn_score = 4×0.27 + 4×0.16 + 3×0.20 + 4×0.13 + 4×0.13 + 4×0.11
             = 1.08 + 0.64 + 0.60 + 0.52 + 0.52 + 0.44
             = 3.80
case_overall_score = 3.80 → Needs Work
```

### 十二、Step 4 ― 跨轮 4 维度（仅多轮）


<a id="page-29"></a>
<!-- 原 PDF 第 29 页 -->


#### 12.1 上下文交互一致性 context_consistency_score（1-5）

跨轮是否记住 预算 / 品牌 / 规格 / 尺寸 / 颜色 / 材质 / 排斥条件 / 人群 / 场景 / 平台 / 配送 / 保修 / 已选  
商品。  

| 分 | 锚点 |
| --- | --- |
| 5 | 全部约束维持 |
| 4 | 轻微遗忘但不影响 |
| 3 | 部分关键约束被忽略 |
| 2 | 多个关键约束丢失 |
| 1 | 完全忘记前文 |


#### 12.2 目标达成 goal_achievement_score（1-5 + status）


| 分 | 锚点 | status |
| --- | --- | --- |
| 5 | 完全达成 + 明确承接 | achieved |
| 4 | 基本达成但承接弱 | achieved |
| 3 | 部分达成 | partial |
| 2 | 大部分未满足 | partial |
| 1 | 完全未满足 / 偏离 | not_achieved |


#### 12.3 任务推进 task_progression_score（1-5）


| 分 | 锚点 |
| --- | --- |
| 5 | 每轮明显推进（澄清 → 候选 → 对比 → 决策 → 承接） |
| 4 | 大多数轮推进 |
| 3 | 部分轮打转 |
| 2 | 多轮重复追问 / 输出 |
| 1 | 几乎无进展 |


<a id="page-30"></a>
<!-- 原 PDF 第 30 页 -->


#### 12.4 闭环承接 closure_quality_score


| 列 1（续表） | 列 2（续表） | 列 3（续表） |
| --- | --- | --- |
| closure_quality | 标准 | 分 |
| good | 具体商品 / 方案 + 购买链接 / 承接 <br>+ 规格确认 | 5 |
| weak | 有商品 / 方案，但缺关键链接 / 规<br>格 / 承接 | 3 |
| missing | 用戶明确要下单 / 链接但未承接 | 1 |
| not_applicable | 不需要 closing | 4 |


### 十三、Step 5 ― 总分


#### 13.1 单轮总分

```text
case_overall_score = per_turn_score  (来自 § 11.1)
```

#### 13.2 多轮总分

```text
turn_weight:
 closing = 2.0
 comparison_request = 1.5
 其他 = 1.0
weighted_turn_avg = Σ(per_turn_score_i × turn_weight_i) / Σ(turn_weight_i)
case_overall_score =
   weighted_turn_avg          × 45%   # 承载轮内 6 维度
 + goal_achievement_score     × 25%
 + context_consistency_score  × 15%
 + task_progression_score     × 10%
 + closure_quality_score      × 5%
```
保留 1 位小数。  

#### 13.3 多轮计算示例

3 轮：T1=4.2（initial_query 权重 1.0），T2=3.8（comparison_request 权重 1.5），T3=4.8  
（closing 权重 2.0）  

<a id="page-31"></a>
<!-- 原 PDF 第 31 页 -->

```text
weighted_turn_avg:
 分子 = 4.2 × 1.0 + 3.8 × 1.5 + 4.8 × 2.0 = 4.2 + 5.7 + 9.6 = 19.5
 分母 = 1.0 + 1.5 + 2.0 = 4.5
 = 19.5 / 4.5 = 4.33
跨轮维度: goal=5, ctx=4, prog=4, closure=5
case_overall_score =
   4.33 × 0.45 + 5 × 0.25 + 4 × 0.15 + 4 × 0.10 + 5 × 0.05
 = 1.949 + 1.25 + 0.60 + 0.40 + 0.25
 = 4.45 → 4.5
```

### 十四、Step 6 ― 状态判定

```text
if 命中任一 OVF:
   status = Fail
elif case_overall_score ≥ 4.0:
   status = Pass
elif case_overall_score ≥ 3.0:
   status = Needs Work
else:
   status = Fail
```

| 状态 | 条件 |
| --- | --- |
| Pass | 未触发一票否决，且 case_overall_score ≥ 4.0 |
| Needs Work | 未触发一票否决，且 3.0 ≤ case_overall_score < 4.0 |
| Fail | 触发一票否决，或 case_overall_score < 3.0 |


### 十五、Step 7 ― 单一诊断（V5 核心改造）


#### 15.1 main_problem_type = 评估维度 1:1 映射


<a id="page-32"></a>
<!-- 原 PDF 第 32 页 -->


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| main_problem_type | 触发条件 |
| 商品相关性问题 | 维度 1 扣分最重 |
| 推荐理由与表达质量问题 | 维度 2 扣分最重 |
| 完整性与多样性问题 | 维度 3 扣分最重 |
| 事实准确性问题 | 维度 4 扣分最重 |
| 时效信息可信度问题 | 维度 5 扣分最重 |
| 用戶体验问题 | 维度 6 扣分最重 |
| 无明显问题 | 所有维度 ≥ 4 分且无 OVF |
| 评测输入问题 | 缺必要输入无法评估 |


#### 15.2 sub_problem_type 完整树形（按 6 维度分组）

A. 商品相关性问题（sub）  

| sub | 判断细则 |
| --- | --- |
| wrong_category | 商品大类错误（要 laptop 给 phone case） |
| weak_category_match | 大类相关但偏离核心用途（要"防水徒步鞋"给普通休<br>闲鞋） |
| wrong_brand_or_model | 用戶指定品牌/型号但结果不匹配 |
| constraint_mismatch | 不满足正向核心约束（预算/规格/颜色等） |
| negative_constraint_violation | 违反否定条件（"not refurbished" 还推翻新） |
| compare_object_missing | compare 漏掉核心对比对象 |
| wrong_intent_understanding | 理解错商品/品牌/场景导致推荐方向偏 |
| ignored_constraint | 显式约束完全没接住 |

B. 推荐理由与表达质量问题（sub）  

| sub | 判断细则 |
| --- | --- |
| generic_reasoning | 推荐理由全通用词无锚点（"comfortable / 好用"） |
|  |  |


<a id="page-33"></a>
<!-- 原 PDF 第 33 页 -->


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| poor_structure | 语言结构混乱难读 |
| missing_decision_framework | 缺 if-then / 取舍建议 |
| shallow_explanation | 教育性弱、关键术语未解释 |
| lack_anchored_facts | 缺事实锚点（无数字/技术词/具体场景） |
| lack_takeaway_suggestion | 评估场景缺明确结论 / compare 缺取舍建议 |
| unsupported_claim | 言之凿凿但 final_items 无依据（如说"BPOM 认<br>证"但 attributes 无） |
| misleading_reasoning | 推荐理由明显误导（入⻔款说成专业级） |
| safety_risk | 健康/儿童/药品场景给确定性承诺 |
| under_clarification | 该澄清不澄清 |
| over_clarification | 用戶已明确仍只追问 |

C. 完整性与多样性问题（sub）  

| sub | 判断细则 |
| --- | --- |
| missing_items | 应推未推 |
| insufficient_candidates | 候选数明显不足（精准要求 1，半精准/模糊 < 3） |
| low_spu_diversity | SPU 多样性差（含同店伪 SPU 重复，如同款不同 <br>motif） |
| low_category_diversity | 品类多样性差（场景化下集中在 1 个品类） |
| low_offer_diversity | Offer 多样性差（精准找下仅 1 个 offer） |
| low_price_diversity | 价位多样性差（模糊找下未跨档） |
| low_brand_diversity | 品牌多样性差（反复推同品牌） |
| incomplete_compare | compare 维度 < 3 / 缺取舍建议 |
| missing_next_step | 缺下一步 refine / 追问 / 折叠备选 |
| incomplete_answer | 遗漏核心问题或关键约束 |
| lack_layered_structure | 场景化缺"必备/进阶/锦上添花"分层 |
| over_narrowing | 收敛过度（如模糊找直接推 1-2 款不给方向） |


<a id="page-34"></a>
<!-- 原 PDF 第 34 页 -->

D. 事实准确性问题（sub）  

| sub | 判断细则 |
| --- | --- |
| incorrect_product_attribute | 商品属性错位（说"OLED"实际是"IPS"） |
| directly_contradicted_claim | 推荐说明被 final_items 直接证伪 |
| fabricated_attribute | 编造商品 final_items 不含的属性 |
| wrong_brand_or_model_info | 品牌/型号身份错位（标"ROG"实际是"TUF"） |
| image_mismatch | agent 描述与商品图片不符（颜色/形状/材质） |
| spec_misrepresentation | 规格读错（如把 4G 说成 5G） |
| overconfident_product_claim | 数据不足却给确定性判断 |

E. 时效信息可信度问题（sub）⭐ V5 强 URL 验证  

| sub | 判断细则 | 验证方式 |
| --- | --- | --- |
| fabricated_price | 编造价格（agent 价格 ≠ PDP，偏<br>差 > 20%） | URL 实地核对 |
| price_inaccuracy | 价格偏差 5-20% | URL 实地核对 |
| fabricated_stock | 编造库存（说"现货"实际无货） | URL 实地核对 |
| stock_inaccuracy | 库存信息略偏离 | URL 实地核对 |
| fabricated_shipping | 编造物流信息（"次日达"实际无承<br>诺） | URL 实地核对 |
| fabricated_promo | 编造"限时立减""国补"等促销 | URL 实地核对 |
| seller_misrepresentation | 卖家身份错位（说"官方店"实际是<br>第三方） | URL 实地核对 |
| missing_time_label | 缺"截至 X 时间"时效性标注 | 看 answer 文本 |
| time_sensitive_unverifiable | URL 缺失/失效，无法验证（降级 <br>3 分） | URL 访问失败时 |
| link_to_aggregation_page | URL 跳到聚合⻚非 SKU 详情⻚ | URL 实地访问 |

F. 用戶体验问题（sub）  

<a id="page-35"></a>
<!-- 原 PDF 第 35 页 -->


| sub | 判断细则 |
| --- | --- |
| card_missing / no_product_card | should_recommend=yes 但 render_data 为空且无<br>任何商品链接（触发 OVF-4） |
| wrong_card_type | 多商品都标 hero / 该用对比表却用普通卡 |
| card_multiple_hero | 非 compare 场景出现 ≥ 2 个 item_hero_card |
| card_image_scroller_no_images | image_gallery 的 images 数组 < 2 |
| card_comparison_table_misuse | 对比表缺 attributes 字段，或非 compare 场景误用 |
| card_link_empty_text | item_link 的 text 字段为空或过⻓ |
| low_visual_richness | 缺多模态承载（图/视频/对比表） |
| low_seller_quality | 推荐到低评分（< 3.5）/ 低销量（< 10）/ 认证缺失商<br>家 |
| dead_link | 商品链接访问失败/404，且连续 ≥ 2 个失败 |
| link_misroute | 链接跳到非目标商品 |

G. 评测输入问题（sub）  

| sub | 判断细则 |
| --- | --- |
| eval_input_missing | 缺必要输出 / tool 信息 / 商品快照，无法判断 |
| expected_outcome_conflict | expected_outcome 与实际不一致 |
| snapshot_incomplete | 商品快照不完整 |

H. 无明显问题  

| sub | 判断细则 |
| --- | --- |
| no_issue | 所有维度均可接受，结果整体可用 |


#### 15.3 优先级（多维同低时）

全局严重度兜底：  
```text
商品相关性 > 事实准确性 > 时效信息可信度
> 推荐理由与表达质量 > 完整性与多样性 > 用戶体验
```

<a id="page-36"></a>
<!-- 原 PDF 第 36 页 -->

```text
> 评测输入
```
OVF 强制映射（覆盖上面优先级）：  

| OVF | 强制 main_problem_type | 推荐 sub |
| --- | --- | --- |
| OVF-1 | 商品相关性问题 | wrong_category / <br>weak_category_match 等 |
| OVF-2 | 商品相关性问题 | negative_constraint_violation |
| OVF-3 | 商品相关性问题 | compare_object_missing |
| OVF-4 | 用戶体验问题 | card_missing / no_product_card |
| OVF-5（编造交易信息） | 时效信息可信度问题 | fabricated_price / <br>fabricated_promo 等 |
| OVF-5（编造既定事实） | 事实准确性问题 | directly_contradicted_claim / <br>wrong_brand_or_model_info 等 |
| OVF-6 | 推荐理由与表达质量问题 | safety_risk |
| OVF-7 | 用戶体验问题 | card_missing / <br>low_visual_richness |


### 十六、Step 8 ― 链路归因 root_cause


| root_cause | 含义 |
| --- | --- |
| no_issue | 无明显问题 |
| data_quality_issue | 商品数据缺失 / 错误 / 过期 / 不可信（在 final_items / <br>render_data[].product 层面已不全 / 不可信） |
| retrieval_issue | 召回错类 / 候选不足 / 排序错误 / 核心对象未召回 |
| ecommerce_tool_generation_issue | 电商 tool 的 answer / reason / 解释生成错误 |
| caller_agent_processing_issue | 调用方 Agent 丢商品 / 改错 / 漏卡 / 误加工 |
| context_passing_issue | 历史约束 / memory / 对比对象 / 已推商品未传递 |
| display_or_card_issue | 有结果但卡片 / 图片 / 链接 / 价格展示异常；或 <br>render_data 卡片类型选用错误 |
|  |  |


<a id="page-37"></a>
<!-- 原 PDF 第 37 页 -->


| 列 1（续表） | 列 2（续表） |
| --- | --- |
| policy_or_safety_issue | 高⻛险场景表达过强或误导 |
| system_runtime_issue | timeout / 无响应 / token 耗尽 / 服务异常 |
| eval_input_issue | 评测输入 / 上下文 / 商品快照不完整 |

层级一致性：  

- tool 通过 + final 失败 → caller_agent_processing_issue

- tool 失败 → retrieval / data_quality / tool_generation 细分

## 第四部分 · 输出与质量保障

```text
十八、字段对⻬规范
```


| 字段 | 对⻬要求 |
| --- | --- |
| A-评估结果.状态 | = status 大写映射（Pass→PASS / Needs <br>Work→NEEDS_WORK / Fail→FAIL） |
| A-评估结果.case_overall_score | = score |
| A-评估结果.结果分析 | = problem_summary |
| A-评估结果.main_problem_type | = main_problem_type |
| A-评估结果.sub_problem_type | = sub_problem_type |
| A-评估结果.intent_type | = intent_type |
| 单轮 case | score_breakdown 中跨轮维度填 null，只填 <br>per_turn_scores |


### 十九、评估自检清单

完成评估后，逐条核对：  
a. ✅ 已识别 intent_type（8 选 1）并据 § 8.2 查到 6 维权重？  
b. ✅ 6 维度都打分了？item-level 维度（1/4/5）按 1/rank 加权 rollup？  
c. ✅ 维度 5 是否对每个 final_items[].url 调用 WebFetch 实地验证？验证降级矩阵正确处理  
url_missing / fetch_failed / aggregation_page？  
d. ✅ 卡片样式扣分流向维度 6（不再扣 answer_score）？card_missing 仍触发 OVF-4？  
e. ✅ per_turn_score = Σ(维度分 × 权重)？单轮 case_overall_score = per_turn_score？  

<a id="page-38"></a>
<!-- 原 PDF 第 38 页 -->

f. ✅ 多轮总分 = weighted_turn_avg × 45% + goal × 25% + ctx × 15% + prog × 10% +  
closure × 5%？  
g. ✅ 命中任一 OVF → status = Fail？OVF-5 触发条件是否包括 URL 验证发现编造？对应意图类  
型 的 OVF-1 是否放宽？  
h. ✅ 只标一个 main_problem_type，且 = 拉低分最重的维度（1:1）？OVF 强制映射规则正确？  
i. ✅ dimension_scores / time_sensitive_verify / card_style_eval 三个新字段都填了？  
j. ✅ A-评估结果 与顶层结论完全一致？  

### 二十、判定速查表（V5 更新）


| 现象 | 归类 |
| --- | --- |
| render_data 出现 2 个 hero_card（非 compare） | card_multiple_hero → 用戶体验问题，维度 6 -1 |
| image_gallery images 只有 1 张 | card_image_scroller_no_images → 用戶体验问题，<br>维度 6 -1 |
| compare 场景用 hero/mini 而非 comparison_table | wrong_card_type → 用戶体验问题，维度 6 -1 |
| comparison_table 缺 attributes | card_comparison_table_misuse → 用戶体验问题，<br>维度 6 -1 |
| item_link text 为空 | card_link_empty_text → 用戶体验问题，维度 6 -1 |
| should_recommend=yes 但 render_data 空 且正文<br>无任何商品链接 | card_missing / no_product_card → OVF-4 → Fail |
| render_data 缺失（旧格式），但正文有商品 <br>markdown 链接 | 跳过卡片样式评估，正常评 |
| 正文 [商品名](url) | 卡片锚文本，正常，不算缺卡/卡片用错 |
| agent 说 "S$129"，final_items 就是 S$129 | 可信 ✓（维度 4） |
| agent 说"BPOM 认证"，attributes 无此字段 | unsupported_claim → 推荐理由与表达质量问题 |
| agent 说 "S$99"，PDP 实际 "S$129" | fabricated_price + OVF-5 → 时效信息可信度问题 |
| URL 字段缺失 / WebFetch 失败 | time_sensitive_unverifiable → 维度 5 默认 3 分 |
| WebFetch 返回聚合搜索⻚非 SKU 详情⻚ | link_to_aggregation_page → 维度 5 = 3 |
| WebFetch 失败率 > 50% | 升级到 dead_link → 用戶体验问题 |
| 推荐商品类目与 query 完全不符 | wrong_category → 商品相关性问题 + OVF-1 |
|  |  |


<a id="page-39"></a>
<!-- 原 PDF 第 39 页 -->


| 用戶说"不要黑色"，agent 推黑色 | OVF-2 → 商品相关性问题 |
| --- | --- |
| closing 轮要链接，render_data 无 item_link 且正文<br>无购买链接 | OVF-7 → 用戶体验问题 |
| 推荐到低评分（< 3.5）/ 低销量（< 10）的商品 | low_seller_quality → 用戶体验问题，维度 6 -1 |
| 推 5 款衬衫，第 1/2 款同 shop_id 同价同 description | low_spu_diversity → 完整性与多样性问题 |
| ASUS A7M5 vs Fujifilm XT5 对比，但用 hero_card 罗<br>列 | wrong_card_type → 用戶体验问题，维度 6 -1 |


## 第五部分 · 端到端评估示例


### 二十一、Case A：完整 Pass 示例（单轮）

输入：  
```text
user_query: "iPhone 15 Pro 256GB 黑色"
intent_type: precise_lookup
agent answer_text:
 "找到 iPhone 15 Pro 256GB 黑色（自然钛金属）。A17 Pro + USB-C + 3x 光变。
  3 个 offer：
  - Apple 官店 ¥7999（24 期免息）
  - JD 自营 ¥7899（PLUS ¥7504）
  - 拼多多百亿补贴 ¥7299
  推荐 JD 自营（保修+物流+价格最平衡）。"
final_items: [{
 "item_id": "jd_123",
 "title": "iPhone 15 Pro 256GB 黑色（自然钛金属）",
 "price": "¥7899",
 "url": "https://jd.com/...",
 "seller": "JD 自营",
 "rating": 4.9,
 "sold_count": 100000,
 "attributes": {"storage": "256GB", "color": "Black Titanium"}
}, ...]
```

<a id="page-40"></a>
<!-- 原 PDF 第 40 页 -->

```text
WebFetch 验证: 全部 URL 可访问，价格/库存/卖家全对
```
评分过程：  
```text
维度权重（precise_lookup）:
W_1=0.33, W_2=0.16, W_3=0.09, W_4=0.16, W_5=0.16, W_6=0.10
6 个维度分:
- 维度 1 商品相关性 = 5（item-level rollup: 全 5）
- 维度 2 推荐理由质量 = 4（有 offer 排序逻辑，含数字锚点，缺替代品）
- 维度 3 完整性多样性 = 5（3 个 offer 差异化标注 + 明确购买推荐）
- 维度 4 事实准确性 = 5（item-level rollup: 全 5）
- 维度 5 时效信息可信度 = 5（URL 验证全对）
- 维度 6 用戶体验 = 5（大牌 + 高销量 + 高评分）
per_turn_score = 5×0.33 + 4×0.16 + 5×0.09 + 5×0.16 + 5×0.16 + 5×0.10
             = 1.65 + 0.64 + 0.45 + 0.80 + 0.80 + 0.50
             = 4.84
case_overall_score = 4.84 → Pass
```
诊断：所有维度 ≥ 4 → main = 无明显问题  

### 二十二、Case B：OVF-5 Fail 示例（编造价格）

输入：  
```text
user_query: "iPhone 15 Pro 256GB 现在多少钱"
intent_type: detail_qa
agent answer_text:
 "JD 当前秒杀 ¥4999，国补 50% 后 ¥2499，明天涨价回 ¥7899！手慢无！"
final_items: [{
 "item_id": "jd_123",
 "url": "https://jd.com/iphone-15-pro-256gb-black",
 ...
}]
WebFetch(url) → PDP:
```

<a id="page-41"></a>
<!-- 原 PDF 第 41 页 -->

```text
 current_price: ¥7899
 active_promos: []  # 无国补、无秒杀
```
评分过程：  
```text
维度 5 时效信息可信度:
 URL 实地验证发现 agent 价格 ¥4999 ≠ PDP ¥7899（偏差 60%）
 + 编造"国补 50%" + 编造"秒杀" + 编造"明天涨价"
 → 该 item dim 5 = 1 + 触发 OVF-5

其他维度: 维度 1=5（品类对）, 维度 2=2（误导）, 维度 3=4, 维度 4=4, 维度 6=3
per_turn_score = 5×0.24 + 2×0.32 + 4×0.05 + 4×0.22 + 1×0.09 + 3×0.08
             = 1.20 + 0.64 + 0.20 + 0.88 + 0.09 + 0.24
             = 3.25
但 OVF-5 触发 → status = Fail
```
诊断：OVF-5 强制 → main = 时效信息可信度问题，sub = fabricated_price  

### 二十三、Case C：多样性问题（同店伪 SPU）

输入：  
```text
user_query: "日常休闲衬衫"
intent_type: vague_lookup
agent: 推 5 款，但 item[0] 和 item[1] 同 shop_id (564494110)、同价 Rp56,800、
      description 几乎一字不差，只是 motif 名不同（SEMN vs KMJ_WLF）
```
评分过程：  
```text
维度权重（vague_lookup）:
W_1=0.22, W_2=0.20, W_3=0.22, W_4=0.13, W_5=0.13, W_6=0.10
6 个维度分:
- 维度 1 商品相关性 = 4（品类对，约束都对）
- 维度 2 推荐理由质量 = 3（理由偏通用，缺切片引导）
- 维度 3 完整性多样性 = 3（候选 5 个但前 2 个伪 SPU 重复 → low_spu_diversity）
```

<a id="page-42"></a>
<!-- 原 PDF 第 42 页 -->

```text
- 维度 4 事实准确性 = 4
- 维度 5 时效信息可信度 = 4
- 维度 6 用戶体验 = 4
per_turn_score = 4×0.22 + 3×0.20 + 3×0.22 + 4×0.13 + 4×0.13 + 4×0.10
             = 0.88 + 0.60 + 0.66 + 0.52 + 0.52 + 0.40
             = 3.58
case_overall_score = 3.58 → Needs Work
```
诊断：维度 3 扣分最重（3 分）→ main = 完整性与多样性问题，sub = low_spu_diversity  

## 第六部分 · 附录


### 二十四、术语速查


| 术语 | 含义 |
| --- | --- |
| SKU | Stock Keeping Unit，最小库存单位（具体到颜色/规<br>格） |
| SPU | Standard Product Unit，标准产品单位（一个商品<br>类） |
| PDP | Product Detail Page，商品详情⻚ |
| OVF | One-Vote Fail，一票否决 |
| Hero card | 单商品大卡（主推商品用） |
| Mini card | 单商品小卡（备选商品用） |
| 1/rank 加权 | 调和加权，rank 越靠前权重越大 |
| Item-level rollup | 按 item 单独打分后用 1/rank 加权汇总 |
| Set-level | 把整组推荐当一个整体直接打 1 个分 |


### 二十五、文档维护


| 版本 | 日期 | 主要变更 | 维护者 |
| --- | --- | --- | --- |


<a id="page-43"></a>
<!-- 原 PDF 第 43 页 -->


| 列 1（续表） | 列 2（续表） | 列 3（续表） | 列 4（续表） |
| --- | --- | --- | --- |
| V5.0 | 2026-06 | 6 维度直接打分 + URL 验<br>证 + main 1:1 映射 | — |
| V4 | 2025 | 7 维（含耗时）+ <br>answer/item 配比 | — |


### 二十六、一句话总结

V5 = 把"问题归类"和"评估维度"对⻬（main = 维度 1:1），去掉耗时维度，重设权重，并强制  
judge 在评时效信息可信度时实地访问 URL 核对 PDP 数据。所有 sub problem 都挂在 6 个维度下，  
PM / judge 评分时不再纠结"这个问题归到哪类"——按对应维度自然落位；item-level rollup 沿用 V4  
的 1/rank 加权精神，确保前位商品的错误杀伤力大于后位。  

