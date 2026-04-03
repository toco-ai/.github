<h1 align="center">Toco AI</h1>

<p align="center">
  <strong>TocoAI: Stop Vibe Coding. 建模驱动的 AI Coding 工具。</strong>
  <strong>先建模，后写代码。告别“开盲盒”式的 AI 编程。</strong>
</p>

<p align="center">
  <b><a href="https://github.com/toco-ai/toco-ai/blob/main/README.md">English</a> | <a href="https://github.com/toco-ai/toco-ai/blob/main/README.ja-JP.md">日本語</a> | <a href="https://github.com/toco-ai/toco-ai/blob/main/README.zh-CN.md">中文 (Simplified)</a></b>
</p>

<p align="center">
  <a href="https://tocoai.dev/en/docs/installation"><img src="https://img.shields.io/badge/IntelliJ_IDEA-2024.03+-blue?style=for-the-badge" alt="IntelliJ IDEA" /></a>
  <a href="https://tocoai.dev/en/docs/installation-vscode"><img src="https://img.shields.io/badge/VS_Code-1.82.0+-blue?style=for-the-badge" alt="VS Code" /></a>
  <a href="https://discord.gg/NubsdbF3MK"><img src="https://img.shields.io/badge/Discord-Join_Community-5865F2?style=for-the-badge" alt="Discord" /></a>
  <a href="https://x.com/TocoAI"><img src="https://img.shields.io/badge/X-@TocoAI-000000?style=for-the-badge" alt="X" /></a>
</p>

<p align="center">
  <img src="assets/hero_tocoai.gif" alt="TocoAI: More Reliable AI Architect -> More Controllable Backend Code -> Easier Backend Changes" width="800" />
</p>

*<p align="center">👆 <b>更可靠的AI架构师</b> ➔ <b>更可控的后端代码</b> ➔ <b>更轻松的后端变更</b></p>*

<h2 align="center">TocoAI ：基于 DSL-Spec 的 backend harness</h2>

> [!TIP]
> "形式化符号是排除各种胡说八道的极其有效的工具。自然语言 的'自然性'，不过是我们能轻松地用它说出那些荒谬性并不显而 易见的话。" — Edsger Dijkstra, EWD667, 1978 

TocoAI是服务端研发的Harness Engineering方案。 
在这，Spec不应是一次性生成代码的文本，而是控制系统的结构层。 
我们用 DSL-Spec 约束大模型生成，并通过建模引擎约束结构性代码（约占80%）。 

<p align="center">
  <img src="assets/tocoai-arch.png" alt="TocoAI Archtecture" width="800" />
</p>

### 我们的Harness Engineering思路 
服务端是我们用来存储数据、还原流程的场景，需要长时间维护，更像是去建造一座大厦。我们希望 用建筑思维，而不是3D打印模式来构建我们的宏伟工程。 

* 我们需要精准的设计图纸，于是我们定义了一套类DDD, 符合CQRS的DSL-SPEC 
* 当AI辅助下，我们有了图纸设计，我们更希望有一致的、易于维护的代码结构，于是我们开发了一 个建模引擎，可以稳态的还原出和设计一致的结构性代码，这部分常在一个复杂项目中占到8成 
* 后在一个结构稳固、水电接口齐全、图纸齐全的大楼里，我们用AI完成室内装修。那些难以DSL 化的，只能通过自然语言和代码表达的if/else部分 

#### 1. DSL-SPEC​
需求和架构设计不再散落在对话、文档或脑子里,也不是纯文本无法校验的随意表达,而是统一表达为结构化的 DSL Spec,作为整个系统的 Single Source of Truth。​

DSL 覆盖后端系统的完整设计层级:领域模型(Entity / Relation)、聚合对象(BO)、数据传输层(DTO / VO)、查询方案(ReadPlan / WO)、写入方案(WritePlan)、服务接口(API / RPC / Service)。​

DSL 人类可读(提供UI展示界面),机器可解析。它不是配置文件——**它是可执行的架构意图**。​

**DSL：**
```JSON
{
    "dto": {
        "uuid": null,
        "name": "user_detail_dto",
        "description": "User details including affiliated company and user settings list",
        "fromEntity": "user",
        "expandList": [
            {
                "foreignKeyInThisEntity": "company_id",
                "dtoFieldName": "company",
                "dto": {
                    "uuid": null,
                    "name": "company_base_dto",
                    "fromEntity": "company",
                    "description": "Basic company information"
                }
            }
        ],
        "reverseExpandList": [
            {
                "foreignKeyInOtherEntity": "user_id",
                "dtoFieldName": "user_setting_list",
                "dto": {
                    "uuid": null,
                    "name": "user_setting_base_dto",
                    "fromEntity": "user_setting",
                    "description": "Basic user setting information"
                }
            }
        ]
    }
}
```

|Generated File|Purpose|
|---|---|
|UserDetailDto.java(~60 Lines)|DTO structure definition with nested Company and UserSetting list|
|UserDetailDtoManager.java(~25 Lines)|Data access interface: getById, getByIdList, getByCompanyId|
|UserDetailDtoManagerImpl.java(~125 Lines)|Implementation with batch query and N+1 prevention logic，Extension point for custom data access logic|
|UserDetailDtoConverter.java(~80 Lines)|Entity → DTO field mapping and conversion|
|UserDetailDtoService.java(~70 Lines)|Service layer with company forward-assembly and user_setting reverse-injection|
|UserDetailDtoDataAssembler.java(~130 Lines)|Assembler, auto-handles nested relational data fetching，postProcessData extension point for custom post-processing|

一种DSL简单示例(组装返回数据)
​
→更多 DSL 详细介绍​（TODO）

#### 2. 建模引擎​
引擎覆盖:分层骨架(persist / manager / service / entrance)、接口契约、数据模型、CQRS 命令查询拆分、跨层转换器(DtoConverter / VoConverter)、数据拼装器(DataAssembler)、聚合写链路(BoService)。
开发者只需关注剩余 **20% 的业务逻辑**,Review 成本骤降一个数量级。​

*→ 引擎机制详细介绍​（TODO）*

#### 3. 标准的Human-in-Loop 
AI从来不是神，在软件工程的世界里，我们需要在合适的地方作为他们的指导者，既不能过度干预， 但也绝不让他无约束的随意发挥。对于长时间影响的领域划分、数据结构定义、接口描述、读写方案 （事务约束、操作性能）我们提供标准的AI修改和手动修改相结合的手段。让AI始终是人的助理，而不 是主人。 

### 对比、优势、案例 

#### 对比 
Cursor、Claude Code 是优秀的工具，我们不是它们的替代品。 

||Cursor / Claude Code|TocoAI|
|---|---|---|
|定位|对话式编程助手|后端工程 Harness|
|代码来源|LLM 按 prompt 生成|DSL → 引擎确定性生成（80%）+ LLM 实现业务逻辑（20%）|
|架构一致性|随迭代漂移|DSL 锁定，不随时间和人员变化|
|变更方式|重新描述，人肉追踪影响范围|改 DSL，引擎自动级联同步|
|Review 成本|全量代码|只审 20% 业务逻辑|
|知识沉淀|对话用完即丢|DSL 即文档，可传承|
|适用场景|原型、碎片化编码|复杂后端系统、长期多人协作|

>一句话： 写一个能跑的系统，用 Cursor。建一个能活三年的系统，用 TocoAI。 

#### 核心优势 
**结构稳定，不随时间漂移** DSL 是系统唯一的真相来源。无论迭代多少次、换多少人，生成的代码结构 始终和设计保持一致。 

**变更成本极低** 改一个字段、调整一个关系，引擎自动级联更新所有受影响的结构性代码。99% 的变更 无需额外 Review。 

**新人上手快** DSL 是可视化的架构文档。新成员不需要啃代码猜意图，看 DSL 一天内可以理解项目全 

**团队协作有标准** 代码风格由引擎统一，不依赖个人 prompt 技巧和 Code Review 约定。团队越大，优势越明显。 

#### 真实案例 

**大型医院 HIS 系统:**

**背景**：新一代全院管理系统，120+ 核心模块，200+ 业务流程，多团队并行开发。 

**难点**：医疗业务零容错，多人跨时间协作极易产生逻辑漏洞和技术债。 

**结果**：架构规范统一执行，生成代码准确率显著提升，整体项目AI代码采纳率近97%。交接时新成员 可快速上手项目全局。 

*→ 案例详细介绍（TODO）*

**金融保证金支付系统重构：**

**背景**：重构遗留保证金支付系统，原技术栈为 SQLServer 2008 + 大量存储过程 + 多个三方中间件。 

**难点**：业务流程散落，数据和业务关联关系不明，性能瓶颈明显。 

**结果**：通过 DSL 重新建模，业务关联关系显式化，端到端流程可追溯，系统可在 TocoAI 框架下持续 迭代。 

### 适应场景及发展方向 

#### DSL 描述的局限性 
当前只适合描述关系型数据库驱动的后端系统。我们会持续深耕这个领域。 
但 Harness Engineering 的思路本身是通用的——在嵌入式、前端组件、基础设施配置等场景同样适 用，只是需要针对那些领域重新定义 DSL 语言。期待其他团队在各自领域延伸这个思路。 

#### DSL 的完备性 
我们实现了一套可运行的基础 DSL，但它不是终点。表达力的边界只有在广泛使用和讨论中才会被发现。 
**我们愿意开源，和社区一起共建新的 DSL 表达能力。** 

#### 引擎的约束性 
引擎的深层含义是"设计时固化"。优秀的 Harness 能力和约束性是同一枚硬币的两面。 
我们的解法：一是**开源共建**，开放引擎规则层让社区扩展；二是**基于 LLM 维护引擎规则**，降低规则修改周期，让引擎的演化速度跟上真实需求。 

#### 老项目的兼容 
可以在老项目中开发新模块，但存量遗留代码的整体接管不在我们的覆盖范围内。 
**重构是 AI 时代的日常工作**。AI 大幅降低了重构成本，正确的路径是用新模块承接新需求，逐步把老逻 辑迁移进 DSL 管辖的范围，而不是等待一个"大重构"时机。 

没有完美的技术，只有适应的场景，一部分是在我们深耕的赛道，我们会矢志不渝坚持下去；一部分 是希望和有同样理解的朋友一起共创的世界。 
毕竟AI改变研发才一年，但软件开发本身已经有七、八十年的历史。 

Getting Started-> 
