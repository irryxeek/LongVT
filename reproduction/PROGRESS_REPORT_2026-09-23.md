# LongVT 复现任务进度报告

汇报日期：2026 年 9 月 23 日

## 一、总体进展

已完成 LongVT-RFT 在原任务 VideoSIAH-Eval 上的全量推理与近似语义评分，以及 QVHighlights 上的完整 training-free validation 评测。QVHighlights LoRA 已完成数据与训练流程搭建，并产出小样本训练 checkpoint；正式训练及最终配对评测尚未完成。

## 二、已完成工作与结果

### 1. 原任务 VideoSIAH 性能核验

已核验模型、数据版本及公开推理协议，完成全部 **652 条问题、244 个来源视频**的 Tool/Global 推理和评分。保存了原始轨迹、工具执行记录、逐题评分、成本账本及断点恢复机制。

| 评测范围 | Tool 语义分数 | Global 语义分数 | Tool−Global（95% 置信区间） |
|---|---:|---:|---:|
| 完整集：652 条 | 45.48 | 54.22 | −8.74 [−11.87, −5.59] |
| 共同有效集：649 条 | 45.69 | 54.39 | −8.71 [−11.93, −5.53] |

分数按 0–100 表示，置信区间采用按来源视频分组的配对 bootstrap。完整集保留两组共同的两条解码失败及 Tool 额外一条无答案，按预先声明规则计零。

本次 Tool 比论文报告的 **42.0 高 3.48 分**。但托管 judge 权重版本及历史服务配置无法完全对齐，因此结论为“**采用官方推荐 judge 的近似性能核验**”，不能宣称严格复现。当前内部对照中 Global 明显高于 Tool，工具负增益的原因仍需诊断。

评分使用官方推荐的 Qwen3-235B-A22B judge 家族，完成 **1,224 个去重 API 请求**（含 sanity），无 API/解析失败或输出截断。累计使用约 **66.56 万输入、2,182 输出 tokens**，估算费用 **$0.194**，低于授权的 $1 总预算。人工盲审尚未完成，人工/judge 一致性尚未验证；上述区间不包含 judge 系统偏差。

### 2. QVHighlights training-free 评测

Global 64 和 Tool 两组均完成全部 **1,550 条 validation**，并完成官方评分核对、逐题配对、评分并列、工具证据覆盖和成本分析。

| VeryGood 指标 | Global 64 | Tool | 差值 |
|---|---:|---:|---:|
| mAP | 13.90 | 14.60 | +0.70 |
| Hit@1 | 20.65 | 22.71 | +2.06 |

Tool 在本次固定实验中有正向差异，但该报告未建立统计显著性。工具调用较稀疏，且存在大量评分并列、局部证据覆盖不足等问题。相关结果已冻结，属于跨任务推理适配，不是原论文训练流程的完整复现。

### 3. QVHighlights LoRA 实验准备与试运行

已完成数据划分、标签转换、多模态输入与 loss mask 检查、精确 LoRA 模块选择及训练/评测脚本。train_fit **6,491 条**、dev **727 条**，所需 **8,619 个媒体片段**全部通过检查，无缺失或已检查的划分交叉。

已发现并修正早期 FSDP/PEFT 保存空 adapter 的问题。后续 smoke_v4 完成 **20 个 optimizer steps**，耗时约 **19 分钟**；最终 adapter 包含 **392 个有限且非零的张量、约 4,037 万参数**。这些产物证明已有真实小样本训练执行，尚不能证明任务性能改善；完整保存加载/生成门槛仍需汇总核实，pilot、正式训练和最终同协议 A/B 评测尚无完成证据。部分早期状态报告已过时，已在 AGENTS.md 标注。

## 三、下一步工作

1. 基于已有全量结果，复核评分缓存、配对关系及统计，分析 VideoSIAH Tool 负增益的逐题来源。
2. 审计工具轨迹与官方协议；仅对明确实现错误修正和受限复测，保留原始结果。
3. 核实 LoRA smoke 的完整技术门槛，再决定是否恢复 Global-only pilot；本轮不直接启动正式训练。

下一阶段执行 prompt 已保存，可交由后续模型继续执行。目前不应概括为“全面符合论文预期”：原任务绝对分数接近论文报告，但协议存在限制，且本地 Tool/Global 对照未体现工具优势。

## 四、主要交付位置

- 原任务完整报告：[VideoSIAH FINAL_REPORT](videosiah_verification/FINAL_REPORT.md)
- QVHighlights 完整评测：[FULL_VALIDATION_REPORT](qvhighlights/FULL_VALIDATION_REPORT.md)
- LoRA 训练产物：[smoke_v4](qvhighlights/lora_sft/runs/smoke_v4/)
- 下一阶段方案：[执行 Prompt](NEXT_REPRODUCTION_PROMPT_LUNA_MAX.md)

本报告依据已保存产物汇总，不表示当前存在正在运行的训练或推理任务。
