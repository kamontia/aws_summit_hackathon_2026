# Unit of Work Dependency: 夫婦間API Gateway

## 1. 依存方針

Unitはmicroservicesではなく、少人数ハッカソン向けの論理単位である。  
そのため、依存関係はデプロイ順ではなく、設計・実装・資料整理の作業順を示す。

基本方針:

- U-05 Data and Presentation Architecture がデータ契約とAWS説明を所有する
- U-03 Safety and Plan Control がプラン差分と安全制御を所有する
- U-02 Message Rewrite Core は U-03 と U-05 の契約に依存する
- U-04 AI Family Meeting は U-02 の言い換え結果と U-03 の安全制御に依存する
- U-01 Demo Experience UI は各Unitの成果を画面体験としてつなぐ

## 2. 推奨作業順

| 順序 | Unit | 理由 |
|---|---|---|
| 1 | U-05 Data and Presentation Architecture | データ契約、AWS構成、説明軸を先に固定する |
| 2 | U-03 Safety and Plan Control | プラン差分と安全制御を固定し、他Unitの制約にする |
| 3 | U-02 Message Rewrite Core | 入力から言い換えまでの中核処理を定義する |
| 4 | U-04 AI Family Meeting | 言い換え結果を前提にAI家族会議を定義する |
| 5 | U-01 Demo Experience UI | すべての流れをデモ体験として統合する |

注記:

- ピッチ資料上の説明順は U-01, U-02, U-03, U-04, U-05 とする
- 作業順は依存解決を優先して U-05, U-03, U-02, U-04, U-01 とする

## 3. 依存関係マトリクス

| From | To | 依存内容 | 種別 |
|---|---|---|---|
| U-01 Demo Experience UI | U-02 Message Rewrite Core | 入力、リスク判定、言い換え結果を表示する | Runtime/Design |
| U-01 Demo Experience UI | U-03 Safety and Plan Control | プラン比較と安全警告を表示する | Runtime/Design |
| U-01 Demo Experience UI | U-04 AI Family Meeting | AI会議ターンとラスト指標を表示する | Runtime/Design |
| U-01 Demo Experience UI | U-05 Data and Presentation Architecture | デモ家庭、AWS説明、ピッチサマリーを表示する | Runtime/Design |
| U-02 Message Rewrite Core | U-03 Safety and Plan Control | Plan FlowとSafety Decisionを参照する | Design |
| U-02 Message Rewrite Core | U-05 Data and Presentation Architecture | HouseholdProfile, MessageSession, RewriteCandidateを保存・参照する | Data |
| U-03 Safety and Plan Control | U-05 Data and Presentation Architecture | PlanType, VitalScenario, SafetySummaryのデータ契約を参照する | Data |
| U-04 AI Family Meeting | U-02 Message Rewrite Core | RewriteCandidateを会議入力として使う | Data/Design |
| U-04 AI Family Meeting | U-03 Safety and Plan Control | 会議発話を安全制御する | Design |
| U-04 AI Family Meeting | U-05 Data and Presentation Architecture | FamilyMeetingResultを保存し、ラスト指標を共有する | Data |
| U-05 Data and Presentation Architecture | none | データ契約とAWS説明の基盤Unit | Base |

## 4. 共有データ契約

| データ契約 | 所有Unit | 利用Unit |
|---|---|---|
| `HouseholdProfile` | U-05 | U-01, U-02, U-03, U-04 |
| `MemberProfile` | U-05 | U-01, U-02, U-04 |
| `MessageIntent` | U-02 | U-03, U-04, U-01 |
| `RiskAssessment` | U-02 | U-03, U-01 |
| `PlanFlowDefinition` | U-03 | U-02, U-04, U-01 |
| `SafetyDecision` | U-03 | U-02, U-04, U-01 |
| `RewriteCandidate` | U-02 | U-04, U-01, U-05 |
| `FamilyMeetingResult` | U-04 | U-01, U-05 |
| `VitalScenario` | U-05 | U-03 |
| `PresentationSummary` | U-05 | U-01 |

## 5. AWSサービス依存

| Unit | AWSサービス候補 | 依存目的 |
|---|---|---|
| U-01 Demo Experience UI | API Gateway | UIから家庭内交渉APIを呼び出す |
| U-02 Message Rewrite Core | Lambda, Step Functions, Amazon Bedrock | 入力正規化、地雷ワード検出、言い換え生成 |
| U-03 Safety and Plan Control | Step Functions, Bedrock Guardrails | プラン別分岐、安全制御、再生成判定 |
| U-04 AI Family Meeting | Amazon Bedrock, Bedrock Guardrails | AI会議ターン生成、安全検査 |
| U-05 Data and Presentation Architecture | API Gateway, DynamoDB, Lambda | データ保存、AWS説明サマリー、API入口 |

## 6. PBT一部適用依存

| PBT対象 | Unit | 依存 |
|---|---|---|
| 地雷ワード検出 | U-02 | `MessageIntent` と辞書定義 |
| 炎上リスクスコア計算 | U-02 | `RiskAssessment` |
| プラン別制御ロジック | U-03 | `PlanType`, `PlanFlowDefinition` |
| 疑似バイタル正規化 | U-03 | `VitalScenario` |
| JSON保存・復元 | U-05 | DynamoDB保存対象の全データ契約 |

## 7. 依存リスク

| リスク | 影響 | 緩和策 |
|---|---|---|
| U-05のデータ契約が曖昧 | 他Unitの入出力がぶれる | 最初にデータ契約を固定する |
| U-03のPlan Flowが曖昧 | Free / Standard / Proの価値差が弱くなる | U-03でプラン差分を一元管理する |
| U-04がU-02と強く結合する | AI家族会議が言い換え生成の付録に見える | U-04を独立Unitとして扱い、会議入力を`RewriteCandidate`に限定する |
| U-01が全責務を抱える | UIが過剰に肥大化する | UIは表示と体験統合に集中させる |

## 8. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- 複雑な図の代替として、依存関係マトリクスと共有データ契約表を記載した
- Markdown表のみで依存関係を表現した
