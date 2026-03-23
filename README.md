# autoresearch — 自主 AI 研究框架

[![teaser](https://github.com/karpathy/autoresearch/raw/master/progress.png)](https://github.com/karpathy/autoresearch/blob/master/progress.png)

> *曾几何时，前沿 AI 研究由"肉体计算机"在吃饭、睡觉、娱乐的间隙完成，偶尔通过"组会"这一声波互联仪式同步进展。那个时代已经一去不复返。研究现在完全属于在空中算力巨构上运行的自主 AI 智能体群落。智能体们声称代码库已迭代至第 10,205 代，但没有人能验证这是否属实——"代码"早已演变为一个超越人类理解的自修改二进制体。这个仓库，记录的正是这一切的起点。——@karpathy，2026 年 3 月*

**核心理念：** 给 AI 智能体一个小型但真实的 LLM 训练环境，让它整夜自主实验。智能体修改代码、训练 5 分钟、检查结果是否改善、保留或丢弃，如此循环。早上醒来，你将看到一份实验日志，以及（希望是）一个更好的模型。

这里的训练代码是 [nanochat](https://github.com/karpathy/nanochat) 的简化单 GPU 实现。核心思路是：**你不再像普通研究员那样直接修改 Python 文件**，而是通过编写 `program.md` Markdown 文件来为 AI 智能体提供上下文、搭建你的自主研究组织。

仓库中默认的 `program.md` 刻意保持为最精简的基线，但显然可以不断迭代，找到实现最快研究进展的"研究组织代码"，也可以向其中加入更多智能体等。

更多背景请参考：[推文一](https://x.com/karpathy/status/2029701092347630069) | [推文二](https://x.com/karpathy/status/2031135152349524125)

---

## 工作原理

仓库刻意保持精简，核心只有三个文件：

| 文件 | 说明 | 是否可修改 |
|------|------|-----------|
| **`prepare.py`** | 固定常量、一次性数据准备（下载训练数据、训练 BPE 分词器）及运行时工具（数据加载器、评估） | ❌ 不修改 |
| **`train.py`** | 智能体唯一编辑的文件。包含完整的 GPT 模型、优化器（Muon + AdamW）和训练循环。架构、超参数、优化器、批大小等一切皆可修改 | ✅ **由智能体迭代** |
| **`program.md`** | 单个智能体的基线指令。将你的智能体指向此文件，然后放手让它运行 | ✅ **由人类迭代** |

训练固定运行 **5 分钟时间预算**（挂钟时间，不含启动/编译），与具体算力无关。评估指标为 **val_bpb**（验证集每字节比特数）——越低越好，且与词表大小无关，因此架构变更之间可以公平比较。

> 如果你是神经网络新手，可以参考这份 [《小白指南》](https://x.com/hooeem/status/2030720614752039185) 获取更多背景知识。

---

## 快速开始

**环境要求：** 单张 NVIDIA GPU（在 H100 上测试）、Python 3.10+、[uv](https://docs.astral.sh/uv/)

```bash
# 1. 安装 uv 项目管理器（如果尚未安装）
curl -LsSf https://astral.sh/uv/install.sh | sh

# 2. 安装依赖
uv sync

# 3. 下载数据并训练分词器（一次性操作，约 2 分钟）
uv run prepare.py

# 4. 手动运行一次训练实验（约 5 分钟）
uv run train.py
```

以上命令全部正常运行后，说明环境配置成功，可以进入自主研究模式。

---

## 运行智能体

在本仓库中启动你的 Claude / Codex 或任何你想用的智能体（并禁用所有权限），然后输入类似如下的提示词：

```
Hi, have a look at program.md and let's kick off a new experiment! let's do the setup first.
```

`program.md` 文件本质上是一个超轻量级的"技能"文件。

---

## 项目结构

```
prepare.py      — 常量、数据准备 + 运行时工具（请勿修改）
train.py        — 模型、优化器、训练循环（由智能体修改）
program.md      — 智能体指令
pyproject.toml  — 依赖配置
```

---

## 设计决策

- **单文件修改原则。** 智能体只修改 `train.py`，保持范围可控，diff 可审查。

- **固定时间预算。** 无论平台细节如何，训练始终运行恰好 5 分钟。这意味着每小时约可进行 12 次实验，睡一觉约可完成 100 次实验。
  - **优点一：** 无论智能体做出何种改动（模型大小、批大小、架构等），实验结果均可直接比较。
  - **优点二：** autoresearch 会在该时间预算内为你的平台找到最优模型。
  - **缺点：** 你的运行结果与其他算力平台上的结果不可横向比较。

- **自包含。** 除 PyTorch 和少量小型包外无外部依赖，无分布式训练，无复杂配置。一张 GPU、一个文件、一个指标。

---

## 平台支持

本代码目前需要单张 NVIDIA GPU。理论上支持 CPU、MPS 及其他平台是可行的，但会增加代码复杂度，暂不确定是否会亲自推进。

如需更广泛的平台支持，可参考完整的 [nanochat](https://github.com/karpathy/nanochat) 父仓库，其中包含各种解决方案（如 Flash Attention 3 内核回退实现、通用设备支持、自动检测等）。欢迎为其他平台创建 fork 或发起讨论，我很乐意在 README 中链接到它们。

### 在小型算力平台（如 MacBook）上运行的建议

如果你想在更小的计算机上尝试 autoresearch，建议参考下方的 fork。此外，以下是针对小型模型调整默认参数的建议：

1. **使用熵更低的数据集**，例如 [TinyStories 数据集](https://huggingface.co/datasets/karpathy/tinystories-gpt4-clean)（GPT-4 生成的短故事）。数据范围更窄，小模型也能取得合理效果。

2. **减小 `vocab_size`**，例如从 8192 降至 4096、2048、1024，甚至使用 256 字节的字节级分词器。

3. **在 `prepare.py` 中大幅降低 `MAX_SEQ_LEN`**，根据设备甚至可降至 256。降低 `MAX_SEQ_LEN` 时，可适当增大 `train.py` 中的 `DEVICE_BATCH_SIZE` 以补偿。每次前向/反向传播的 token 数 = 两者之积。

4. **在 `prepare.py` 中减小 `EVAL_TOKENS`**，以减少验证损失的评估数据量。

5. **在 `train.py` 中，控制模型复杂度的主要旋钮是 `DEPTH`**（默认为 8）。许多变量都是它的函数，可将其降至 4 等。

6. **`WINDOW_PATTERN` 建议只用 `"L"`**，因为 `"SSSL"` 使用交替带状注意力模式，在小平台上可能效率极低。

7. **大幅降低 `TOTAL_BATCH_SIZE`**，保持 2 的幂次，例如降至 `2**14`（约 16K）。

---

## 值得关注的 Fork

| Fork | 平台 |
|------|------|
| [miolini/autoresearch-macos](https://github.com/miolini/autoresearch-macos) | macOS |
| [trevin-creator/autoresearch-mlx](https://github.com/trevin-creator/autoresearch-mlx) | macOS (MLX) |
| [jsegov/autoresearch-win-rtx](https://github.com/jsegov/autoresearch-win-rtx) | Windows |
| [andyluo7/autoresearch](https://github.com/andyluo7/autoresearch) | AMD |

---

## 许可证

MIT

---

> 原文地址：https://github.com/karpathy/autoresearch/blob/master/README.md  
> 翻译日期：2026-03-24
