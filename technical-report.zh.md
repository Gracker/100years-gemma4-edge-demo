# 100 Years Gemma 4 Hackathon 端侧 AI 技术报告

## 项目定位

100 Years 是一个隐私优先的 iOS 健康寿命助手。它把 HealthKit、睡眠、运动、体检报告、体脂数据、生活方式问卷、基础疾病、长期病、家族病史、长期用药和用户备注汇总成个人健康画像，再用端侧 Gemma 4 E2B/E4B 生成解释、风险提示和可执行建议。

项目不提供医疗诊断或临床预后，输出属于健康模型估算、健康教育和风险识别。

## 核心结果

| 能力 | 结果 |
| --- | --- |
| 端侧健康简报 | Today 页面可以基于本地健康画像生成每日健康总结、风险解释和行动建议。 |
| 私密病史画像 | 用户可录入基础疾病、长期病、家族病史、长期用药和备注，并作为敏感分析上下文。 |
| 本地优先寿命估算 | 本地算法和本地 Gemma prompt 会使用 diseaseRisk、身体指标、生活方式和 HealthKit 信号。 |
| 本地优先报告导入 | 体检报告、PDF、体脂秤截图和运动截图走本地优先解析。 |
| 执行路径透明 | AI 输出附带 metadata-only receipt，只披露执行模式、模型、工具类别和隐私摘要。 |

## 功能支持矩阵

| 功能 | 支持模式 | 演示优先级 | 说明 |
| --- | --- | --- | --- |
| Today 每日健康简报 | 本地模型完全支持 | 必演示 | 本地 Gemma 4 agent 调用健康工具，生成短健康摘要和 metadata-only receipt。 |
| Gemma 4 工具调用 | 本地模型完全支持 | 必演示 | 模型按 JSON schema 调用 `getHealthSnapshot`、`getSleepAnalysis`、`computeLifespanImpact` 等只读工具。 |
| 单阶段寿命估算 | 本地模型完全支持 | 必演示 | 展示病史、HealthKit、报告和生活方式如何进入本地 prompt。 |
| 深度本地寿命分析 | 本地模型完全支持，受 E4B 和耗时限制 | 可选 | 4 个本地阶段加综合阶段，适合设备和时间稳定时展示。 |
| 病史隐私画像 | 本地模型完全支持 | 必演示 | 基础疾病、长期病、家族病史、用药和备注可本地录入并参与分析。 |
| 本地 OCR/PDF + Gemma 文本结构化 | 本地模型完全支持 | 建议演示 | 报告、PDF、截图先本地提取文本，再由本地 Gemma 做结构化。 |
| 原生视觉直读 | 已接入但待真机验收 | 边界说明 | 未完成真实 iPhone 验收前，不作为稳定生产能力宣传。 |
| 睡眠/运动 compact 估算 | 端云结合 | 可选 | balanced/allLocal 走 compact 本地任务；allCloud 走完整云端 prompt。 |
| compact 场景预测 | 端云结合 | 可选 | 本地模型可跑短推理版本；balanced 可在本地不可用时云端兜底。 |
| full 场景预测 | 纯云端 | 只作对比 | 长上下文深推理任务，不是 Edge AI 主演示。 |
| 云端多阶段寿命分析 | 纯云端 | 只作对比 | 4 个并行领域分析加 synthesis，适合质量对比，不是端侧主线。 |

## Gemma 4 函数调用伪流程

```swift
func generateDailyBrief(snapshot) async throws -> DailyBriefResult {
    let context = await AIService.captureRoutingContext()
    let plan = try AITaskRouter.plan(for: .dailyBrief, context: context)

    if plan.primary == .edge {
        let executor = AgentExecutor(
            driver: GemmaAgentDriver(),
            tools: HealthAgentToolRegistry.allTools
        )

        let run = try await executor.run(
            systemMessage: "summarize short-term health status",
            userMessage: snapshot.promptSummary(),
            descriptors: HealthAgentToolRegistry.descriptors
        )

        return DailyBriefResult(
            payload: try await renderFinalAsPayload(run.finalAnswer),
            receipt: AIExecutionReceipt(
                kind: .edge,
                routeTarget: .edge,
                usedHealthDataCategories: usedCategories(snapshot, run.trace),
                usedTools: usedTools(run.trace),
                limitations: [.noMedicalDiagnosis]
            ),
            trace: humanReadableTrace(run.trace)
        )
    }

    return try await generateViaDirectJSON(snapshot, context)
}
```

一轮工具调用被约束成 JSON envelope：

```json
{
  "tool_call": {
    "name": "getSleepAnalysis",
    "arguments": { "days": 14 }
  }
}
```

## 关键代码入口

源码仓库为私有仓库，已授权 GDG Reviewer 访问。以下链接需要仓库权限。

| 代码 | 作用 |
| --- | --- |
| [DailyBriefGenerator.generate](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/DailyBriefGenerator.swift#L24) | Today 简报入口：规划 route，选择 Gemma agent 或 direct JSON。 |
| [AgentExecutor.run](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/AgentExecutor.swift#L141) | 工具调用循环，限制 turn、tool call、总时长和重复调用。 |
| [GemmaAgentDriver.nextStep](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/GemmaAgentDriver.swift#L32) | 调用本地 Gemma runtime，并用 grammar 约束输出。 |
| [HealthAgentTools](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/HealthAgentTools.swift#L18) | 注册只读健康工具。 |
| [AITaskRouter.plan](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AITaskRouter.swift#L56) | 根据任务、模型安装、网络、用户模式和 native vision 能力选择 edge/cloud。 |
| [AIService+Routed](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIService+Routed.swift#L97) | 执行 route，并保存真实 provider/model/route metadata。 |
| [AIService+CompactEdge](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIService+CompactEdge.swift#L446) | 构建本地 compact 寿命估算 prompt。 |
| [LongevityAlgorithm.calculateDiseaseRiskScore](https://github.com/Gracker/100Years/blob/main/100Years/Services/LongevityAlgorithm.swift#L540) | 个人病史和家族病史进入 diseaseRisk。 |
| [MedicalConditionsInputView](https://github.com/Gracker/100Years/blob/main/100Years/Views/Medical/MedicalConditionsInputView.swift#L11) | 隐私病史入口。 |
| [AIExecutionReceipt](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIExecutionReceipt.swift#L59) | metadata-only 执行路径和隐私回执。 |

## 演示主线

1. 展示本地 Gemma 4 E2B/E4B 模型状态和 execution receipt。
2. 生成 Today 健康简报，突出 Gemma 4 tool calling。
3. 添加基础疾病、家族病史或长期用药，再运行 compact 寿命估算。
4. 展示结构化输出：寿命估算、健康分数、风险因素、建议和 receipt。
5. 导入报告/PDF/截图，解释本地 OCR/PDFKit + 本地 Gemma 文本结构化。
6. 解释 balanced / allLocal / allCloud，以及本地、端云结合、纯云端、待验收能力边界。

## 端侧 AI 边界

稳定演示主线是本地 Gemma 文本推理、工具调用、本地优先解析、病史上下文和 metadata-only receipt。

上线前仍必须完成真实 iPhone 的性能、内存、发热和导入样本验收。

不要把“Gemma 原生视觉直读图片”表述成稳定生产承诺。

真机验收前不宣传 Gemma 多模态直读已生产可用。

