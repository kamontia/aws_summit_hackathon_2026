# Component Methods: 夫婦間API Gateway

## 1. 方針

この文書では、詳細な業務ロジックではなく、Application Design段階の高レベルなメソッド、入力、出力を定義する。  
実装に進む場合、Functional Designで詳細なルールと例外条件を定義する。

## 2. 共通データ型

| 型 | 内容 |
|---|---|
| `HouseholdId` | デモ家庭を識別するID |
| `ActorId` | 入力者または受け取り側を識別するID |
| `PlanType` | `Free`, `Standard`, `Pro` |
| `MessageIntent` | 入力文、家庭内タスク、感情、依頼内容を正規化した構造 |
| `RiskAssessment` | 地雷ワード、リスクスコア、リスクラベル、避けるべき表現 |
| `SafetyDecision` | 許可、警告、再生成、停止の判断 |
| `RewriteCandidate` | 言い換え文、理由、元意図との対応 |
| `FamilyMeetingResult` | AI家族会議の議題、発話、合意案、ラスト指標 |
| `VitalScenario` | Pro将来拡張用の疑似バイタル情報 |
| `TimingRecommendation` | 伝えるタイミングの推奨 |

## 3. C-01 Mobile Demo Web UI

| Method | Input | Output | 目的 |
|---|---|---|---|
| `showLoginScreen()` | none | `LoginView` | デモ開始画面を表示する |
| `submitDemoLogin(accountCode)` | `string` | `HouseholdContext` | デモ家庭へログインする |
| `submitMessageDraft(input)` | `MessageDraftInput` | `MessageSessionView` | 不満・依頼を送信する |
| `showRiskResult(sessionId)` | `SessionId` | `RiskResultView` | 地雷ワードと炎上リスクを表示する |
| `showRewriteCandidate(sessionId)` | `SessionId` | `RewriteView` | 言い換え候補を表示する |
| `startFamilyMeeting(sessionId)` | `SessionId` | `FamilyMeetingView` | AI家族会議画面へ進む |
| `showFinalSatire(meetingId)` | `MeetingId` | `FinalSatireView` | ブラックユーモアのラスト画面を表示する |

## 4. C-02 Demo Auth and Household Context

| Method | Input | Output | 目的 |
|---|---|---|---|
| `authenticateDemoAccount(accountCode)` | `string` | `DemoSession` | デモアカウントを認証扱いにする |
| `resolveHouseholdContext(demoSession)` | `DemoSession` | `HouseholdContext` | 家庭ID、入力者、相手、プランを解決する |
| `loadInitialHouseholdState(householdId)` | `HouseholdId` | `HouseholdProfile` | デモ家庭の初期状態を取得する |

## 5. C-03 API Gateway Facade

| Method | Input | Output | 目的 |
|---|---|---|---|
| `POST /demo-login` | `DemoLoginRequest` | `HouseholdContextResponse` | デモログインを受ける |
| `POST /message-sessions` | `MessageDraftRequest` | `MessageSessionResponse` | 入力からセッションを開始する |
| `POST /message-sessions/{id}/rewrite` | `RewriteRequest` | `RewriteResponse` | リスク判定と言い換えを実行する |
| `POST /message-sessions/{id}/meeting` | `FamilyMeetingRequest` | `FamilyMeetingResponse` | AI家族会議を開始する |
| `GET /presentation-summary` | `PresentationSummaryRequest` | `PresentationSummary` | 審査員向け説明を取得する |

## 6. C-04 Plan Flow Controller

| Method | Input | Output | 目的 |
|---|---|---|---|
| `selectPlanFlow(planType)` | `PlanType` | `PlanFlowDefinition` | プランごとの処理フローを選ぶ |
| `requiresManualInstructions(planType)` | `PlanType` | `boolean` | Freeで手動指示が必要か判定する |
| `resolveGenerationMode(planType)` | `PlanType` | `GenerationMode` | 自動調整の度合いを決める |
| `buildPlanComparison(sessionId)` | `SessionId` | `PlanComparison` | Free / Standard / Pro の差分表示を作る |

PBT候補:

- `selectPlanFlow` は、任意の有効な `PlanType` に対して必須ステップを含むことを不変条件として扱える
- `requiresManualInstructions` は、Freeのみ `true` になることを不変条件として扱える

## 7. C-05 Message Intake Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `validateMessageDraft(input)` | `MessageDraftInput` | `ValidationResult` | 空入力や不足項目を検出する |
| `normalizeMessageIntent(input, householdProfile)` | `MessageDraftInput`, `HouseholdProfile` | `MessageIntent` | 入力を構造化する |
| `extractHouseholdTask(messageText)` | `string` | `HouseholdTaskHint` | 洗い物、保育園準備などの議題候補を抽出する |
| `applyFreeManualInstructions(intent, instructions)` | `MessageIntent`, `ManualInstructions` | `MessageIntent` | Freeの手動指示を反映する |

PBT候補:

- `normalizeMessageIntent` は、空でない入力から空の `originalText` を返してはいけない
- JSON保存・復元の往復対象として `MessageIntent` を扱える

## 8. C-06 Risk and Landmine Analyzer

| Method | Input | Output | 目的 |
|---|---|---|---|
| `detectLandmines(messageIntent, dictionary)` | `MessageIntent`, `LandmineDictionary` | `LandmineDetection` | 地雷ワードを検出する |
| `calculateRiskScore(detection, householdProfile)` | `LandmineDetection`, `HouseholdProfile` | `RiskScore` | 家庭内炎上リスクを計算する |
| `classifyRiskLevel(score)` | `RiskScore` | `RiskLevel` | Low, Medium, Highなどへ分類する |
| `buildRewriteConstraints(assessment)` | `RiskAssessment` | `RewriteConstraints` | 言い換え生成に渡す制約を作る |

PBT候補:

- `calculateRiskScore` は常に0から100の範囲に収まる
- `classifyRiskLevel` は同じスコアに対して同じ分類を返す
- 地雷ワードを追加しても、既存の検出結果が消えない単調性を候補にできる

## 9. C-07 Safety Control Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `evaluateAppRules(content, context)` | `string`, `SafetyContext` | `AppRuleDecision` | アプリ側禁止表現を判定する |
| `applyBedrockGuardrails(content, context)` | `string`, `SafetyContext` | `GuardrailDecision` | Bedrock Guardrailsで安全制御する |
| `mergeSafetyDecisions(appDecision, guardrailDecision)` | `AppRuleDecision`, `GuardrailDecision` | `SafetyDecision` | 警告、再生成、停止を決める |
| `buildSafetySummary(decision)` | `SafetyDecision` | `SafetySummary` | 審査員向け説明に使う安全サマリーを作る |

## 10. C-08 Rewrite Generation Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `buildRewritePrompt(intent, assessment, flow)` | `MessageIntent`, `RiskAssessment`, `PlanFlowDefinition` | `PromptSpec` | Bedrockに渡すプロンプト仕様を作る |
| `generateRewrite(promptSpec)` | `PromptSpec` | `RewriteCandidate` | 言い換え候補を生成する |
| `explainRewrite(candidate, assessment)` | `RewriteCandidate`, `RiskAssessment` | `RewriteExplanation` | どの地雷表現をどう避けたか説明する |
| `regenerateWithSafetyFeedback(candidate, decision)` | `RewriteCandidate`, `SafetyDecision` | `RewriteCandidate` | 安全制御結果を反映して再生成する |

## 11. C-09 AI Family Meeting Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `createMeetingAgenda(intent, candidate)` | `MessageIntent`, `RewriteCandidate` | `MeetingAgenda` | 家庭内議題を整理する |
| `buildAgentRoles(householdProfile)` | `HouseholdProfile` | `AgentRoleSet` | 妻AI、夫AI、調停AIの役割を定義する |
| `generateMeetingTurns(agenda, roles)` | `MeetingAgenda`, `AgentRoleSet` | `MeetingTurn[]` | AI同士の発話ターンを生成する |
| `summarizeAgreement(turns)` | `MeetingTurn[]` | `AgreementSummary` | 合意案をまとめる |
| `buildFinalSatireMetrics(result)` | `FamilyMeetingResult` | `SatireMetrics` | ラスト画面指標を作る |

## 12. C-10 Household Data Store

| Method | Input | Output | 目的 |
|---|---|---|---|
| `getHouseholdProfile(householdId)` | `HouseholdId` | `HouseholdProfile` | 家庭プロフィールを取得する |
| `putMessageSession(session)` | `MessageSession` | `WriteResult` | 入力セッションを保存する |
| `putRewriteCandidate(candidate)` | `RewriteCandidate` | `WriteResult` | 言い換え結果を保存する |
| `putFamilyMeetingResult(result)` | `FamilyMeetingResult` | `WriteResult` | AI家族会議結果を保存する |
| `getPresentationSummary(householdId)` | `HouseholdId` | `PresentationSummaryData` | 審査員向けサマリーの元データを取得する |

PBT候補:

- `HouseholdProfile`, `MessageSession`, `RewriteCandidate`, `FamilyMeetingResult` はJSON保存・復元の往復性を検証できる

## 13. C-11 Pro Vital Timing Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `normalizeVitalScenario(input)` | `VitalScenarioInput` | `VitalScenario` | 疑似バイタル情報を正規化する |
| `classifyRecipientState(scenario)` | `VitalScenario` | `RecipientState` | ストレス、睡眠、活動量から状態を分類する |
| `recommendTiming(state, messageIntent)` | `RecipientState`, `MessageIntent` | `TimingRecommendation` | 伝えるタイミングを推奨する |
| `buildFutureConsentNotes()` | none | `ConsentNotes` | 将来連携時の同意・監査論点を作る |

PBT候補:

- `normalizeVitalScenario` は心拍、睡眠、ストレスなどを定義済み範囲へ収める
- `recommendTiming` は必ず許可されたタイミング分類を返す

## 14. C-12 Presentation Component

| Method | Input | Output | 目的 |
|---|---|---|---|
| `buildArchitectureSummary()` | none | `ArchitectureSummary` | AWSサービス対応を説明する |
| `buildBusinessPlanSummary()` | none | `BusinessPlanSummary` | Free / Standard / Proの課金理由を説明する |
| `buildSatireSummary(metrics)` | `SatireMetrics` | `SatireSummary` | 人をダメにするテーマを説明する |
| `buildSafetyNarrative(summary)` | `SafetySummary` | `SafetyNarrative` | ブラックユーモアへの安全配慮を説明する |

## 15. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- メソッドは擬似シグネチャとしてMarkdown表に記載した
- 実装コードや実際の型定義は作成していない
