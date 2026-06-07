# 100 Years Gemma 4 Hackathon Edge AI Technical Report

## Product Positioning

100 Years is a privacy-first iOS longevity assistant. It combines HealthKit, sleep, activity, body metrics, lifestyle answers, medical reports, personal medical conditions, chronic diseases, family history, long-term medications, and private notes into a longitudinal health profile. Gemma 4 E2B/E4B then turns that profile into explanations, risk context, and practical next actions on device whenever the selected task supports local execution.

The app does not diagnose disease or provide clinical prognosis. Its role is health education, risk awareness, and personal context preservation.

## Core Results

| Capability | Result |
| --- | --- |
| On-device health brief | The Today screen can generate daily health summaries, risk explanations, and action suggestions from the local health profile. |
| Private condition profile | Users can record personal conditions, chronic diseases, family history, long-term medications, and private notes as sensitive analysis context. |
| Local-first longevity estimate | Local algorithms and local Gemma prompts use disease risk, body metrics, lifestyle factors, and HealthKit signals. |
| Local-first report import | Medical reports, PDFs, body-composition screenshots, and workout screenshots use local-first parsing. |
| Transparent execution receipt | AI output includes metadata-only receipts that disclose execution mode, model identity, tool categories, and privacy categories without exposing raw medical history. |

## Feature Support Matrix

| Feature | Support mode | Demo priority | Notes |
| --- | --- | --- | --- |
| Today daily health brief | Fully local model supported | Must show | Local Gemma 4 agent calls read-only health tools, then returns a short health brief and metadata-only receipt. |
| Gemma 4 tool calling | Fully local model supported | Must show | The model calls read-only tools such as `getHealthSnapshot`, `getSleepAnalysis`, and `computeLifespanImpact` through JSON schemas. |
| Single-stage longevity estimate | Fully local model supported | Must show | Shows medical history, HealthKit, reports, and lifestyle context entering a local prompt. |
| Deep local longevity analysis | Fully local model supported, E4B/time limited | Optional | Four local stages plus synthesis; useful if the demo device and timing are stable. |
| Private medical-history profile | Fully local model supported | Must show | Personal conditions, chronic diseases, family history, medications, and notes are entered locally and used by local analysis. |
| Local OCR/PDF plus Gemma text structuring | Fully local model supported | Recommended | Reports, PDFs, and screenshots are extracted locally first, then structured by local Gemma text inference. |
| Native vision for body/workout/report images | Wired but pending real-device validation | Boundary only unless verified | Do not present as stable production capability before real iPhone validation. |
| Compact sleep/exercise adjustment | Edge-cloud combined | Optional | balanced/allLocal use compact local tasks; allCloud uses the full cloud prompt. |
| Compact scenario prediction | Edge-cloud combined | Optional | Local model can run the short reasoning version; balanced mode can use cloud fallback when local is unavailable. |
| Full scenario prediction | Cloud only | Comparison only | Long-context deep reasoning does not fit the E2B/E4B compact edge path. |
| Cloud multi-stage longevity analysis | Cloud only | Comparison only | Four parallel domain analyses plus synthesis; useful for quality comparison, not the primary edge story. |

## Gemma 4 Function Calling Flow

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

One Gemma 4 tool-call turn is constrained to a JSON envelope:

```json
{
  "tool_call": {
    "name": "getSleepAnalysis",
    "arguments": { "days": 14 }
  }
}
```

## Key Code Entry Points

The source repository is private. GDG Reviewer access has been granted. These links require repository access.

| Code | Role |
| --- | --- |
| [DailyBriefGenerator.generate](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/DailyBriefGenerator.swift#L24) | Today brief entry point: route planning, Gemma agent, or direct JSON path. |
| [AgentExecutor.run](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/AgentExecutor.swift#L141) | Tool-calling loop with turn, tool-call, wall-clock, and repeat-call limits. |
| [GemmaAgentDriver.nextStep](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/GemmaAgentDriver.swift#L32) | Calls the local Gemma runtime and constrains output with grammar. |
| [HealthAgentTools](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/Agent/HealthAgentTools.swift#L18) | Registers read-only health tools. |
| [AITaskRouter.plan](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AITaskRouter.swift#L56) | Chooses edge or cloud from task type, model install state, network state, user mode, and native-vision capability. |
| [AIService+Routed](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIService+Routed.swift#L97) | Executes the route and preserves provider/model/route metadata. |
| [AIService+CompactEdge](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIService+CompactEdge.swift#L446) | Builds compact local longevity prompts. |
| [LongevityAlgorithm.calculateDiseaseRiskScore](https://github.com/Gracker/100Years/blob/main/100Years/Services/LongevityAlgorithm.swift#L540) | Uses personal and family medical history in diseaseRisk. |
| [MedicalConditionsInputView](https://github.com/Gracker/100Years/blob/main/100Years/Views/Medical/MedicalConditionsInputView.swift#L11) | Privacy-aware medical-history entry. |
| [AIExecutionReceipt](https://github.com/Gracker/100Years/blob/main/100Years/Services/AIService/AIExecutionReceipt.swift#L59) | Metadata-only route and privacy receipt. |

## Demo Story

1. Show local Gemma 4 E2B/E4B model state and execution receipt.
2. Generate the Today health brief and highlight Gemma 4 tool calling.
3. Add a private condition, family-history item, or medication, then run compact longevity analysis.
4. Show structured output: lifespan estimate, health scores, risk factors, recommendations, and receipt.
5. Import a report/PDF/screenshot and explain local OCR/PDFKit plus local Gemma text structuring.
6. Explain balanced / allLocal / allCloud and the local, edge-cloud combined, cloud-only, and validation-gated task boundaries.

## Edge AI Boundary

The stable demo centers on local Gemma text inference, tool calling, local-first parsing, medical-history context, and metadata-only receipts.

Do not claim Gemma native vision is production-ready until it is verified on real iPhone hardware with representative screenshots and PDFs.

The current submitted demo can show the native-vision app path only after real-device validation. Until then, imported screenshots and PDFs should be described as local-first with a stable Apple Vision/PDFKit plus local Gemma text fallback.

Do not claim Gemma multimodal vision unless finished and verified.

