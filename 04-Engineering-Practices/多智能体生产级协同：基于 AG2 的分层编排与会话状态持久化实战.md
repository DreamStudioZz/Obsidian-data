---
tags:
  - ai-practice
  - engineering
  - architecture
domain: Agent设计
difficulty: 进阶
date_added: 2026-10-08
---

# 💡 多智能体生产级协同：基于 AG2 的分层编排与会话状态持久化实战

> **核心摘要**：AutoGen 在进化为独立的 AG2 生产框架后，全面剥离了早期的研究原型冗余逻辑。本文详解如何基于 AG2 构建企业级分层多智能体（Hierarchical Multi-Agent）编排架构，结合自定义发言人选择策略（Speaker Selection）、外部会话状态持久化序列化、Token 预算熔断拦截与人在回路（HITL）审批断点，解决复杂长周期任务中的死循环、状态丢失与调用失控难题。

---

## 🎯 业务/技术背景与痛点

在企业落地多智能体协同系统（如复杂的研报生成、端到端敏捷开发流、金融风险合规审查）时，简单的双人对话（Two-Agent Chat）或粗放的群聊（Broadcast GroupChat）在生产环境中经常遇到严峻瓶颈：

1. **群聊发言混乱与意图死循环（Chatter Loops）**：
   - 在传统的轮询或自由竞争发言模式下，多个 Agent 容易陷入礼貌客套（如反复出现“谢谢”、“您说得对”）、相互踢皮球，或者在缺乏全局领导角色的情况下偏离初始目标，消耗成千上万无效 Token；
2. **长周期任务状态丢失（State Amnesia）**：
   - 生产环境的多智能体任务往往耗时数分钟甚至数小时，遇到服务重启、网络波动或需要异步等待人类外部审批时，内存中的上下文状态瞬间丢失，无法无缝恢复执行；
3. **缺乏刚性开销管控与故障熔断机制**：
   - 多智能体交叉调用会带来指数级上升的 Token 与 API 消耗，缺乏全局预算监控（Max Rounds / Max Cost）容易造成账单黑洞；
4. **工具调用与人类干预割裂**：
   - 某些关键动作（如删除生产资源、执行转账、合并主干代码）必须引入强制人机协同审批，传统回调函数难以在异步分布式环境中挂起并安全唤醒。

---

## 🏗️ 架构设计与解决方案

针对上述痛点，采用 **AG2 分层指挥总线（Hierarchical Orchestration）+ 状态快照持久化网关** 架构：

```mermaid
flowchart TD
    User([用户发起复杂工程/研报任务]) --> Gateway[任务入口网关 & 预算配额校验]
    Gateway --> Leader[Team Leader / 架构师 Agent]

    subgraph StateStore [状态快照与持久化中心]
        StateDB[(Redis / PostgreSQL 状态持久化)]
        SnapshotEngine[上下文序列化与恢复引擎]
        StateDB <--> SnapshotEngine
    end

    subgraph AG2Cluster [AG2 分层多智能体协同集群]
        direction TB
        Leader -->|分解子任务 / 派发指令| Router{自定义发言人调度器}
        
        Router -->|文献/数据检索| Researcher[Research Agent (数据检索分析)]
        Router -->|业务代码/方案实现| Coder[Engineer Agent (方案实现与代码构建)]
        Router -->|合规校验/质量门禁| Reviewer[Reviewer Agent (安全审查与逻辑评估)]
        
        Researcher -->|提交调研初稿| Router
        Coder -->|提交实现版本| Router
        Reviewer -->|提交审计报告| Router
        
        Router -->|汇总各子成果| Leader
    end

    Leader --> StateSync[每轮次会话增量序列化]
    StateSync --> SnapshotEngine
    
    Leader --> QualityCheck{任务是否达到验收准则?}
    QualityCheck -->|未达标 / 需修复| Router
    QualityCheck -->|需要关键决策| HITL[Human-in-the-Loop 人机审批断点]
    
    HITL -->|人工通过| FinalDelivery([最终交付物输出])
    HITL -->|打回修改| Leader
```

### 核心解法分步剖析
1. **分层指挥与动态委派**：建立单一决策核心（Leader），工作 Agent（Researcher, Coder, Reviewer）只与 Leader 汇报或在受控子群中交互，避免全员无序广播；
2. **状态快照机制**：在每个关键交互轮次通过 AG2 提供的消息序列化接口提取 Conversation History 与 Agent Scratchpad，持久化存储于 Redis/数据库；
3. **Token 预算与深度熔断**：配置 `max_consecutive_auto_reply` 和全局 Token 限额，超出立即降级并触发预警；
4. **异步审批断点**：通过挂起指令将任务置为 `WAITING_FOR_APPROVAL`，释放执行线程，待外部 Webhook 确认后直接注入上下文恢复执行。

---

## 💻 关键代码 / 配置 / Prompt 模板

以下是在生产环境中基于 **AG2** 实现分层协作与状态持久化保存与恢复的完整工程示例：

```python
"""
production_ag2_workflow.py
基于 AG2 (Formerly AutoGen) 的分层智能体编排与会话状态持久化实战
"""

from typing import List, Dict, Any, Optional
import json
import logging
from autogen import ConversableAgent, GroupChat, GroupChatManager

logging.basicConfig(level=logging.INFO, format="%(asctime)s [%(levelname)s] %(message)s")
logger = logging.getLogger("AG2-Production")

# 1. 统一大模型配置
llm_config = {
    "config_list": [
        {
            "model": "gpt-4o",
            "api_key": "sk-mock-key-replace-with-env",
            "temperature": 0.2,
            "cache_seed": 42,
        }
    ],
    "timeout": 120,
}

# 2. 角色定义与 System Prompt 精准固化
leader_system_prompt = """
你是分层多智能体团队的总指挥（Team Leader）。
你的核心职责：
1. 深入分析用户的复杂目标，将其拆解为确定性的执行步骤；
2. 调度 Researcher 进行情报与数据搜集；
3. 调度 Coder 进行系统实现；
4. 调度 Reviewer 进行代码审核与安全检验；
5. 当且仅当所有交付物通过 Reviewer 的严格审核后，输出最终整合结论并明确包含 'TERMINATE'。
严禁与成员进行无关闲聊，严格控制讨论轮次。
"""

coder_system_prompt = """
你是资深系统研发专家（Coder Agent）。
职责：根据 Leader 明确的规范与 Researcher 提供的素材编写生产级、健壮的高性能代码。
当收到 Reviewer 的修改意见时，迅速针对性修复并输出完整 Diff 或修改后的代码。
"""

reviewer_system_prompt = """
你是代码安全与合规审计师（Reviewer Agent）。
职责：检查 Coder 产出的代码是否存在逻辑缺陷、内存/连接泄漏、并发竞争或安全性漏洞（SQL 注入、反序列化风险等）。
如果存在缺陷，指出精确修改意见；如已达到生产上线标准，输出 'REVIEW_PASSED'。
"""

# 3. 初始化 ConversableAgent 实例
leader = ConversableAgent(
    name="TeamLeader",
    system_message=leader_system_prompt,
    llm_config=llm_config,
    human_input_mode="NEVER",
)

coder = ConversableAgent(
    name="Coder",
    system_message=coder_system_prompt,
    llm_config=llm_config,
    human_input_mode="NEVER",
)

reviewer = ConversableAgent(
    name="Reviewer",
    system_message=reviewer_system_prompt,
    llm_config=llm_config,
    human_input_mode="NEVER",
)


# 4. 自定义发言人选择逻辑（避免群聊争抢与无序发言）
def custom_speaker_selection(last_speaker: ConversableAgent, groupchat: GroupChat):
    messages = groupchat.messages
    if not messages:
        return leader

    last_content = messages[-1]["content"]

    # 如果由 Leader 刚发言，根据指示分配执行角色
    if last_speaker == leader:
        if "请 Coder" in last_content or "Coder" in last_content:
            return coder
        if "请 Reviewer" in last_content or "Reviewer" in last_content:
            return reviewer
        return coder

    # Coder 发言完毕后，强制转交 Reviewer 进行审核
    if last_speaker == coder:
        return reviewer

    # Reviewer 审核完毕后，交回 Leader 做最终定夺或进一步规划
    if last_speaker == reviewer:
        return leader

    return leader


# 5. 构建群聊中枢与管理器
agents_pool = [leader, coder, reviewer]
groupchat = GroupChat(
    agents=agents_pool,
    messages=[],
    max_round=12,  # 刚性最大轮次防死循环
    speaker_selection_method=custom_speaker_selection,
)

manager = GroupChatManager(
    groupchat=groupchat,
    llm_config=llm_config,
    name="HierarchicalManager",
)


# 6. 会话状态快照序列化与恢复工具类
class SessionStateGateway:
    @staticmethod
    def save_state(chat_manager: GroupChatManager, filepath: str = "ag2_session_state.json"):
        """将群聊上下文及 Agent 状态导出为持久化 JSON"""
        state_data = {
            "messages": chat_manager.groupchat.messages,
            "agent_names": [a.name for a in chat_manager.groupchat.agents],
        }
        with open(filepath, "w", encoding="utf-8") as f:
            json.dump(state_data, f, ensure_ascii=False, indent=2)
        logger.info(f"会话状态已持久化至: {filepath}")

    @staticmethod
    def restore_state(chat_manager: GroupChatManager, filepath: str = "ag2_session_state.json"):
        """从持久化快照恢复群聊上下文"""
        with open(filepath, "r", encoding="utf-8") as f:
            state_data = json.load(f)
        chat_manager.groupchat.messages = state_data.get("messages", [])
        logger.info(f"成功从 {filepath} 恢复 {len(chat_manager.groupchat.messages)} 条历史消息")


# 7. 任务调度执行函数
def run_production_task(task_prompt: str):
    logger.info("启动 AG2 分层任务流水线...")
    try:
        # 发起协作
        chat_result = leader.initiate_chat(
            manager,
            message=task_prompt,
            summary_method="last_msg",
        )
        # 持久化会话快照
        SessionStateGateway.save_state(manager)
        return chat_result
    except Exception as e:
        logger.error(f"任务执行出现异常: {str(e)}，正在保留现场状态...")
        SessionStateGateway.save_state(manager, "ag2_error_snapshot.json")
        raise e


if __name__ == "__main__":
    task = "请使用 Python 实现一个高可靠线程安全的限流器（Token Bucket），要求包含单元测试。"
    # 模拟启动运行
    # result = run_production_task(task)
    logger.info("AG2 分层多智能体编排模版就绪。")
```

---

## ⚠️ 生产环境踩坑与避坑指南

1. **群聊上下文（Context Window）急剧膨胀导致截断**：
   - **痛点**：多智能体每轮发言都作为完整历史推送到所有 Agent 的输入端，5 轮以上即可能耗尽 32k/64k Token；
   - **避坑方案**：在 `GroupChatManager` 中配置 `select_speaker_prompt` 压缩策略，或者利用中间 Agent 定期对历史消息调用 `summarize_messages` 生成阶段摘要，清空前序细节日志。
2. **自定义发言人死锁（Ping-Pong Lock）**：
   - **痛点**：`custom_speaker_selection` 如果规则书写不严谨（如 Coder 返回不含审查触发词，而 Reviewer 又在等待 Coder），会导致 Agent 之间互相等待或死锁停滞；
   - **避坑方案**：必须在选择器中设置兜底兜转逻辑（Fallback to Leader），且配置全局 `max_round` 超时与无进展（Progress Stagnation）熔断器。
3. **异步并发环境下的共享代理污染**：
   - **痛点**：在多租户 Web 服务中，直接复用同一个全局 `ConversableAgent` 单例会导致不同用户的对话历史相互串音与污染；
   - **避坑方案**：坚持“按会话工厂创建（Factory per Request）”模式，或严格在每个 Request 开始前调用 `agent.reset()` 清空对话内部状态。

---

## 🔗 关联项目与引用
- 相关工具: [[AG2]], [[CrewAI]], [[LangGraph]], [[OpenHands]]
- 官方项目: [ag2ai/ag2 (GitHub)](https://github.com/ag2ai/ag2)
- 进阶阅读: Multi-Agent Conversation Frameworks in Enterprise Production
