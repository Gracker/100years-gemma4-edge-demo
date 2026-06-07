# 100 Years - Gemma 4 Edge AI 演示

[English version](README.md)

![100 Years app screens](assets/today.png)

这是 100 Years 参加 Gemma 4 Hackathon 的公开落地页。

## 提交链接

| 项目 | 链接 / 状态 |
| --- | --- |
| Online demo URL | `https://github.com/Gracker/100years-gemma4-edge-demo` |
| Demo video URL | 待上传 |
| 技术报告 | [英文](technical-report.md) / [中文](technical-report.zh.md) |
| 源码仓库 | [Gracker/100Years](https://github.com/Gracker/100Years)，私有仓库，已授权 GDG Reviewer 访问 |
| 赛道 | C - Edge AI |

这个公开仓库刻意保持很小，只包含落地页、公开截图、技术报告公开副本和提交说明。这里不包含私有 iOS 源码、模型文件、API key、健康数据或用户数据。

## 为什么这是 Online Demo URL

100 Years 是一个 Edge AI iOS App，真实演示运行在物理 iPhone 上，本地加载 Gemma 4 E2B/E4B runtime。因此它不是一个可以在浏览器里直接试用的云端 Web App，也没有托管的在线推理服务。

这个页面是给评委打开的公开入口：它集中放置 demo 视频、技术报告、私有源码仓库指向、功能支持矩阵和演示主线。

主源码仓库保持私有，是因为其中包含未发布的 iOS App 源码和健康数据相关实现细节。GitHub 不提供可靠的“私有仓库单文件公开”模式，所以这里单独创建一个公开仓库作为提交安全的落地页。

## 项目定位

100 Years 是一个隐私优先的 iOS 健康寿命助手。它把 HealthKit、睡眠、运动、身体指标、生活方式问卷、体检报告、基础疾病、长期病、家族病史、长期用药和用户备注汇总成长期健康画像。Gemma 4 E2B/E4B 会在端侧支持的任务中，把这些上下文转化为解释、风险提示和可执行建议。

App 不提供疾病诊断或临床预后。它的定位是健康教育、风险识别和个人长期健康上下文保存。

## 功能支持矩阵

| 功能 | 支持模式 | 演示优先级 | 演示重点 |
| --- | --- | --- | --- |
| Today 每日健康简报 | 完全本地 | 必演示 | Gemma 4 agent 调用只读健康工具，然后返回短健康摘要和 metadata-only receipt。 |
| Gemma 4 工具调用 | 完全本地 | 必演示 | 展示 health snapshot、sleep analysis、lifespan impact 等 JSON tool call。 |
| 单阶段寿命估算 | 完全本地 | 必演示 | 展示病史、HealthKit、报告和生活方式如何进入本地 prompt。 |
| 病史隐私画像 | 完全本地 | 必演示 | 基础疾病、长期病、家族病史、长期用药和备注保留在本地画像中。 |
| 本地 OCR/PDF + Gemma 文本结构化 | 本地优先 fallback | 建议演示 | Apple Vision/PDFKit 本地提取，再由本地 Gemma 文本结构化。 |
| 睡眠 / 运动 compact 调整估算 | 端云结合 | 可选 | 先走本地 compact 估算；只有显式选择深度分析时才走云端。 |
| full 场景预测和 full 生活方式分析 | 纯云端 | 只作对比 | 长上下文深推理不是 Edge AI 主演示。 |
| Gemma 原生视觉 | 已接入但待验收 | 边界说明 | 未完成真实 iPhone 验收前，不宣传为生产可用能力。 |

## 以模型为中心的演示主线

1. 展示本地模型状态：打开本地模型或结果 receipt 页面，展示 Gemma 4 E2B/E4B、执行路径和本地/云端标识。
2. 展示 Gemma 4 工具调用：生成 Today 简报，解释 `DailyBriefGenerator -> AgentExecutor -> GemmaAgentDriver -> HealthAgentTools`。
3. 展示私密病史：添加基础疾病、家族病史或长期用药，说明它如何改变本地 prompt 上下文和 diseaseRisk 计算。
4. 展示结构化本地输出：运行 compact 寿命估算，展示健康分数、风险因素、建议和 metadata-only receipt。
5. 展示本地优先报告解析：导入报告/PDF/截图，解释本地 OCR/PDFKit + 本地 Gemma 文本结构化。
6. 展示端云边界：解释 `balanced`、`allLocal`、`allCloud`，并说明哪些任务是本地、端云结合、纯云端、待真机验收。

## 核心技术证据

源码仓库为私有仓库。以下链接需要仓库访问权限；GDG Reviewer 已经获得访问权限。

| 代码 | 作用 |
| --- | --- |
| [DailyBriefGenerator.generate](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/DailyBriefGenerator.swift#L24) | Today 简报入口：规划 route，选择 Gemma agent 或 direct JSON 路径。 |
| [AgentExecutor.run](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/AgentExecutor.swift#L141) | 工具调用循环，限制 turn、tool call、wall-clock 和重复调用。 |
| [GemmaAgentDriver.nextStep](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/GemmaAgentDriver.swift#L32) | 调用本地 Gemma runtime，并用 grammar 约束输出。 |
| [HealthAgentTools](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/HealthAgentTools.swift#L18) | 注册只读健康工具。 |
| [AITaskRouter.plan](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AITaskRouter.swift#L56) | 根据任务、模型安装状态、网络状态、用户模式和 native-vision 能力选择 edge 或 cloud。 |
| [AIService+CompactEdge](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIService+CompactEdge.swift#L446) | 构建 compact 本地寿命估算 prompt。 |
| [LongevityAlgorithm.calculateDiseaseRiskScore](https://github.com/Gracker/100Years/blob/main/100Years/Services/LongevityAlgorithm.swift#L540) | 将个人病史和家族病史纳入 diseaseRisk。 |
| [AIExecutionReceipt](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIExecutionReceipt.swift#L59) | metadata-only 执行路径和隐私回执。 |

## 隐私和边界说明

- 基础疾病、家族病史、长期用药和备注属于敏感个人数据。
- 本地可执行任务会把这些上下文保留在设备本地画像中。
- 执行回执只披露 route、model、数据类别和工具 ID，不披露原始疾病名称、备注、报告文本或 HealthKit 数值。
- 输出是健康模型估算和健康教育，不是诊断、治疗或临床预后。
- 稳定 Edge AI 演示主线是本地 Gemma 文本推理、工具调用、本地优先解析、病史上下文和 metadata-only receipt。
- Gemma 原生视觉仍然处于真实设备验收门控状态。

## 截图

| Today 简报 | 本地模型设置 | 寿命估算详情 |
| --- | --- | --- |
| ![Today health brief](assets/today.png) | ![Local model settings](assets/local-model.png) | ![Prediction detail](assets/prediction.png) |

