---
tags:
  - ai-tool
  - open-source
  - security
  - sandbox
  - status/evaluating
category: Agents
github: "https://github.com/nvidia/openshell"
stars: "3k+"
license: "Apache-2.0"
date_added: 2026-10-08
---

# 📦 OpenShell

> **一句话简介**：NVIDIA 开源的智能体安全外壳与隔离执行运行时，专为大规模 Agent 集群的受限环境调用与权限防御设计。

## 📌 基本信息

| 属性 | 内容 |
| :--- | :--- |
| **仓库地址** | [GitHub - nvidia/openshell](https://github.com/nvidia/openshell) |
| **技术栈** | C++, Python, Linux cgroups, eBPF |
| **许可证** | Apache-2.0 |
| **相关关联** | 关联专题: [[E2B]], [[Promptfoo]], [[NeMo-Guardrails]] |

---

## 🚀 核心特性与亮点
1. **细粒度系统调用过滤（Syscall Shield）**：基于 Linux eBPF 与 Seccomp，拦截 Agent 代码对敏感内核调用、提权操作及未授权套接字的访问。
2. **集群级网络隔离策略**：支持严格的 Domain 白名单出站流量控制，彻底防止数据外发（Data Exfiltration）与 SSRF 攻击。
3. **资源开销硬限制**：精细控制 CPU/内存/磁盘 I/O 阈值，防止 Agent 进入无限循环死锁消耗宿主资源。
4. **与同类工具对比**：
   - 相比于 E2B：OpenShell 更侧重于私有机房、多卡服务器集群上的轻量级内核防护，适合自建基础设施团队；
   - 相比于普通 Docker：OpenShell 具备 AI 原生的动态权限降级能力。

---

## 🛠️ 快速上手与配置

```bash
pip install openshell-agent
```

```python
from openshell import SecureRuntime, SecurityPolicy

policy = SecurityPolicy(
    allow_network=True,
    allowed_domains=["api.github.com"],
    max_memory_mb=1024,
    deny_syscalls=["ptrace", "reboot"]
)

with SecureRuntime(policy=policy) as runtime:
    result = runtime.execute_code(
        "import requests; print(requests.get('https://api.github.com').status_code)"
    )
    print(result.output)
```

---

## 💡 工程实战点评与适用场景
- **推荐使用场景**：企业内部私有化部署 Coding Agent、需要严格审计与等保合规的数据分析助手。
- **潜在不足 / 局限性**：对 Linux 内核版本有一定要求（需支持 eBPF 高级特性），在 Windows/macOS 开发机上通常需要借助 WSL2。
- **个人评估结论**：具备高安全要求的私有大模型平台强烈建议评估引入。
