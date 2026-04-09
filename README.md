# 🦌 DeerFlow 2.0 源码学习笔记（精简版）

> 基于 ByteDance 开源 Super Agent Harness 的学习总结

## 📌 项目简介

DeerFlow 2.0 是字节跳动开源的 **Super Agent Harness（超级智能体执行底座）**，从早期 Deep Research 工具全面升级而来。它不再只是“生成内容”，而是能够 **规划、执行并完成长周期复杂任务** 的 AI 系统 :contentReference[oaicite:0]{index=0}  

核心目标：

- 让 Agent 从“会聊天” → “能干活”
- 支持复杂、多步骤、长时间任务执行

---

## 🧠 核心架构理解

DeerFlow 2.0 本质是一个 **Agent 运行时系统（Runtime）**，核心由以下模块组成：

- **Lead Agent + Sub-Agents**
  - 主 Agent 负责规划
  - 动态生成子 Agent 并行执行任务 :contentReference[oaicite:1]{index=1}  

- **Agent Harness**
  - 提供任务规划、工具调用、记忆管理等能力
  - 相当于 AI 的“身体” :contentReference[oaicite:2]{index=2}  

- **Memory + Context**
  - 支持长上下文与跨任务记忆
  - 自动摘要避免上下文丢失 :contentReference[oaicite:3]{index=3}  

- **Sandbox + File System**
  - 提供安全执行环境（代码、文件操作等）
  - 支持长时间运行任务 :contentReference[oaicite:4]{index=4}  

- **Skills & Tools**
  - 可扩展能力体系（搜索、代码、API等）

---

## ⚡ 核心特点

- 🧩 多 Agent 协同 + 并行调度  
- 🧠 长时任务记忆与上下文管理  
- 🛠️ 工具化执行（不仅是文本生成）  
- 🔒 沙箱执行保障安全  
- 🔌 Skills 可扩展能力体系  

👉 本质：一个“带操作系统能力”的 AI Agent

---

## 📈 总结

DeerFlow 2.0 的关键突破在于：

> 将 Agent 从 Prompt 封装，升级为 **可编排的执行系统**

它代表了 AI Agent 的一个重要趋势：  
**从对话工具 → 自动化生产力系统** :contentReference[oaicite:5]{index=5}  

---

## 🔗 参考

- https://github.com/bytedance/deer-flow  
- https://github.com/coolclaws/deerflow-book  
