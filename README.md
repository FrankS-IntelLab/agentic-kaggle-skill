<div align="center">

<img src="https://img.shields.io/badge/Agentic-AI%20Driven%20Kaggle-9cf?style=for-the-badge&logo=kaggle&logoColor=white" alt="Agentic Kaggle"/>

<br>

# 🤖 Agentic Data Science Competition
# 🤖 智能体驱动的数据科学竞赛

<p align="center">
  <strong>Turn AI Agents into Kaggle Teammates</strong><br>
  <strong>让 AI 智能体成为你的 Kaggle 队友</strong>
  <br/>
  <sub>Not just tools — autonomous collaborators that research, debug, iterate, and win.</sub><br>
  <sub>不仅仅是工具 —— 能够自主研究、调试、迭代并获胜的合作者。</sub>
</p>

<p align="center">
  <a href="#-quick-start--快速开始">Quick Start</a> •
  <a href="#-key-insights--核心洞察">Key Insights</a> •
  <a href="#-case-studies--案例研究">Case Studies</a> •
  <a href="#-installation--安装">Install</a>
</p>

<p align="center">
  <img src="https://img.shields.io/github/stars/FrankS-IntelLab/agentic-kaggle-skill?style=social" alt="GitHub stars"/>
  <img src="https://img.shields.io/github/forks/FrankS-IntelLab/agentic-kaggle-skill?style=social" alt="GitHub forks"/>
</p>

---

**Distilled from 2+ years of Kaggle competition experience**
**提炼自 2+ 年 Kaggle 竞赛实战经验**

Including: **RL Game AI** • **Audio Classification** • **LLM Reasoning** • Multiple debugging journeys
包括：**强化学习游戏 AI** • **音频分类** • **LLM 推理** • 多次调试实践

---

</div>

## 🎯 What This Skill Does / 这个 Skill 能做什么

Transform your Kaggle workflow from **manual iteration** to **autonomous agent-driven competition**:

将你的 Kaggle 工作流从**手动迭代**转变为**智能体驱动竞赛**：

| Before 之前 | After 之后 |
|--------|-------|
| Manual notebook analysis 手动分析 notebook | Agent pulls top solutions with dependencies 智能体拉取顶级方案及依赖 |
| Guess why submission failed 猜测提交失败原因 | Agent diagnoses 400 errors, zip format issues 智能体诊断错误 |
| Wait and refresh for scores 刷新等待分数 | Cronjob monitors kernel, auto-submits Cronjob 监控，自动提交 |
| Try random improvements 随机尝试改进 | Spec-driven development → delegate → verify 规范驱动开发 |

---

## ⚡ Quick Start / 快速开始

```bash
# Install the skill / 安装 skill
mkdir -p ~/.hermes/skills/data-science/agentic-kaggle/
curl -sL https://raw.githubusercontent.com/FrankS-IntelLab/agentic-kaggle-skill/main/SKILL.md \
  -o ~/.hermes/skills/data-science/agentic-kaggle/SKILL.md
```

Then in your AI agent / 然后在你的 AI 智能体中：
```
> Use the agentic kaggle skill to help me replicate this top notebook
> Why is my submission returning 400 error?
> Set up auto-monitoring for my kernel
```

---

## 💡 Key Insights / 核心洞察

### 1️⃣ Score Stabilization Pattern / 分数稳定模式

**Don't trust early scores! / 不要相信早期分数！**

| Time 时间 | Score 分数 | Reality 真相 |
|------|-------|---------|
| 0-2h | Peak 峰值 (800-1000) | ⚠️ **Inflated 虚高** — lucky matchups 幸运匹配 |
| 4h+ | Stable 稳定 (600-800) | ✅ **True score 真实分数** |

### 2️⃣ Competition Types Matter / 竞赛类型很重要

| Type 类型 | Submit 提交内容 | Detection 检测方法 |
|------|--------|-----------|
| **Answer 答案** | CSV predictions CSV 预测 | Most competitions 大多数竞赛 |
| **Model 模型** | LoRA/checkpoints LoRA/检查点 | Top notebooks train models 顶级 notebook 训练模型 |

### 3️⃣ Kernel Mode Trap / Kernel 模式陷阱

| Mode 模式 | Test Set 测试集 | Result 结果 |
|------|----------|--------|
| `kaggle kernels push` | ❌ Hidden 隐藏 | Invalid submission 无效提交 |
| "Save & Run All" | ✅ Mounted 挂载 | Valid submission 有效提交 |

### 4️⃣ Top Replication Workflow / 顶级方案复制流程

```bash
# Pull WITH metadata (-m is critical!) / 带 metadata 拉取（-m 很关键！）
kaggle kernels pull user/top-notebook -p ./solution/ -m

# Edit only id/title, KEEP all dependencies / 只修改 id/title，保留所有依赖
kaggle kernels push -p ./solution/
```

---

## 📊 Case Studies / 案例研究

### RL Strategy Game Competition / RL 策略游戏竞赛

| Lesson 教训 | Details 详情 |
|--------|---------|
| **Feature completeness 功能完整性** | Top agents: 3,000+ lines → LB 1200+ 顶级智能体：3000+ 行 |
| **Simplified agents 简化版** | ~120 lines, 4/12 features → LB 500-600 简化版：~120 行 |
| **Time budget 时间预算** | Strict turn limits — profile after each change 严格回合限制 |

👉 [Full Case Study / 完整案例](examples/rl-game-case-study.md)

### Audio Classification Competition / 音频分类竞赛

| Lesson 教训 | Details 详情 |
|--------|---------|
| **Hybrid ensemble 混合集成** | Temporal model + SED ensemble = Top scores 时序模型 + SED 集成 |
| **Prior limitations 先验局限** | Location priors don't help when all samples are similar 样本相似时先验无效 |
| **Silent failures 静默失败** | Log exceptions during feature extraction 记录异常 |

👉 [Full Case Study / 完整案例](examples/audio-classification-case-study.md)

---

## 🛠️ Troubleshooting Cheat Sheet / 故障排除速查表

| Problem 问题 | Solution 解决方案 |
|---------|----------|
| `400 Bad Request` | Try `.zip` format (only zip the CSV!) 尝试 .zip 格式 |
| `FileNotFoundError` | Check `/kaggle/input/competitions/<name>/` 检查路径 |
| Training IDs in submission 提交包含训练 ID | Use `sample_submission.csv` for fallback 使用 sample_submission.csv |
| Score dropped 分数下降 | Wait 4h for stabilization 等待 4 小时稳定 |
| GPU OOM | Use 4-bit quantization 使用 4-bit 量化 |
| `CUDA error` | FP16 → load_in_4bit=True |

---

## 📦 Installation / 安装

### Option 1: Direct Copy / 方式 1：直接复制

```bash
mkdir -p ~/.hermes/skills/data-science/agentic-kaggle/
cp SKILL.md ~/.hermes/skills/data-science/agentic-kaggle/
```

### Option 2: Claude Code CLI / 方式 2：Claude Code CLI

```bash
npx skills add FrankS-IntelLab/agentic-kaggle-skill
```

### Option 3: Manual Download / 方式 3：手动下载

Download `SKILL.md` and place in your skills directory.
下载 `SKILL.md` 并放入你的 skills 目录。

---

## 📁 Repository Structure / 仓库结构

```
agentic-kaggle-skill/
├── SKILL.md                              # Core skill definition / 核心 skill 定义
├── README.md                             # This file / 本文件
├── LICENSE                               # MIT
├── references/research/
│   ├── 01-competition-patterns.md        # Score stabilization, types
│   ├── 02-troubleshooting-guide.md       # Error diagnosis
│   └── 03-automation-patterns.md         # Cronjobs, delegation
└── examples/
    ├── rl-game-case-study.md             # RL game competition
    └── audio-classification-case-study.md # Audio classification
```

---

## 🧠 Mental Models / 心智模型

| Model 模型 | Description 描述 |
|-------|-------------|
| **Score Stabilization 分数稳定** | Early scores lie — wait 4h for truth 早期分数会骗人 |
| **Spec-Driven Development 规范驱动开发** | Document before coding, delegate with clarity 先文档后编码 |
| **Fail Fast, Learn Faster 快速失败，快速学习** | Systematic debugging beats random iteration 系统化调试 |
| **Agent as Teammate 智能体即队友** | Not just a tool — an autonomous collaborator 不仅是工具 |

---

## 🔗 Related Skills / 相关 Skills

| Skill | Purpose 用途 |
|-------|---------|
| `kaggle-auto-submit` | End-to-end automation with cronjob 端到端自动化 |
| `autonomous-iteration` | ANALYSIS → BUILD → EXPERIMENT → REVIEW 分析→构建→实验→审查 |
| `opencode` | Delegate coding to OpenCode CLI 委托编码 |
| `claude-code` | Delegate coding to Claude Code CLI 委托编码 |

---

## ⭐ Why Star This Repo? / 为什么 Star？

- ✅ **Battle-tested patterns** from real competitions 实战验证的模式
- ✅ **Bilingual documentation** (English + 中文) 双语文档
- ✅ **Practical troubleshooting** for common Kaggle issues 实用故障排除
- ✅ **Spec-driven workflow** templates included 规范驱动工作流模板
- ✅ **Case studies** with actual LB scores 带实际 LB 分数的案例研究

---

## 🤝 Contributing / 贡献

Found a new pattern? Solved a tricky error? 发现了新模式？解决了棘手错误？

1. Fork the repo / Fork 仓库
2. Add your insight to `references/research/` or `examples/` 添加你的洞察
3. Submit a PR / 提交 PR

---

## 📄 License / 许可证

MIT — Use freely, modify freely, learn freely.
MIT —— 自由使用，自由修改，自由学习。

---

<div align="center">

**Made with 🤖 by [Frank S (IntelLab)](https://github.com/FrankS-IntelLab)**

[![Kaggle](https://img.shields.io/badge/Kaggle-franksunp-blue?logo=kaggle)](https://www.kaggle.com/franksunp)
[![GitHub](https://img.shields.io/badge/GitHub-FrankS--IntelLab-black?logo=github)](https://github.com/FrankS-IntelLab)

<br>

*If this skill helped you win a competition, ⭐ star the repo!*
*如果这个 skill 帮助你赢得了竞赛，请 ⭐ star 这个仓库！*

</div>
