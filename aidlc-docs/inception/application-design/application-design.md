# Application Design: 夫婦間API Gateway

## 1. サマリー

夫婦間API Gateway は、家庭内の言いにくいことをAIが翻訳し、最終的にはAI同士が家族会議を行う、AWS Summit ハッカソン向けの風刺的サービスである。

Application Design では、応募・ピッチ資料に使える論理設計として、以下を確定した。

| 項目 | 決定 |
|---|---|
| 主案 | 基本サーバーレス構成 |
| 入口 | Amazon API Gateway |
| 処理制御 | AWS Step Functions + AWS Lambda |
| 生成AI | Amazon Bedrock |
| 安全制御 | Bedrock Guardrails + アプリケーションルール |
| データ保存 | Amazon DynamoDB |
| 将来進化形 | Amazon Bedrock AgentCore |
| AI家族会議 | 妻AI、夫AI、調停AIを論理コンポーネントとして扱う |
| プラン差分 | Free / Standard / Pro ごとに処理フローを分ける |
| Proバイタル | 将来拡張コンポーネント |

## 2. 成果物

| 成果物 | 内容 |
|---|---|
| `components.md` | コンポーネント、責務、DynamoDB設計、PBT受け皿 |
| `component-methods.md` | 各コンポーネントの高レベルメソッドと入出力 |
| `services.md` | サービス層、AWS対応、プラン別オーケストレーション |
| `component-dependency.md` | コンポーネント依存、AWS依存、データフロー |
| `application-design.md` | 統合サマリー |

## 3. 主要コンポーネント

| Component | 役割 |
|---|---|
| Mobile Demo Web UI | モバイルファーストの体験画面 |
| Demo Auth and Household Context | デモ家庭の解決 |
| API Gateway Facade | 家庭内交渉APIの入口 |
| Plan Flow Controller | Free / Standard / Proの処理差分 |
| Message Intake Component | 入力正規化 |
| Risk and Landmine Analyzer | 地雷ワードと炎上リスク判定 |
| Safety Control Component | アプリルールとGuardrailsによる安全制御 |
| Rewrite Generation Component | Bedrockによる言い換え生成 |
| AI Family Meeting Component | 妻AI、夫AI、調停AIの合意形成 |
| Household Data Store | DynamoDBによる家庭データ保存 |
| Pro Vital Timing Component | Pro将来拡張のタイミング制御 |
| Presentation Component | 審査員向け説明材料の生成 |

## 4. 主なユーザー体験

### 4.1 Standard中心のMVP体験

1. ユーザーがデモ家庭にログインする
2. 家庭内の不満や依頼をそのまま入力する
3. システムが地雷ワードと家庭内炎上リスクを判定する
4. Bedrockが相手に伝わりやすい文面へ言い換える
5. Bedrock Guardrailsとアプリルールが危険表現を抑制する
6. ユーザーが生成文を確認する

### 4.2 AI家族会議体験

1. 言い換え結果をもとに、家庭内議題を作る
2. 妻AI、夫AI、調停AIの論理役割を作る
3. AI同士の会議ターンを生成する
4. 合意案をまとめる
5. 「家庭内摩擦: 低下」「直接会話: 0件」「関係性の自律性: 低下」を表示する

### 4.3 プラン差分

| プラン | 体験 | 人をダメにする度合い |
|---|---|---|
| Free | 細かい指示が必要。静的設定なので外すと逆撫でする可能性がある | 低 |
| Standard | 言いたいことだけ入力すればAIが基本自動調整する | 中 |
| Pro | 将来拡張として、疑似バイタル情報から伝えるタイミングまでAIが決める | 高 |

## 5. DynamoDB設計サマリー

単一テーブル `HouseholdGatewayTable` を候補とする。

| Entity | 代表キー | 用途 |
|---|---|---|
| HouseholdProfile | `HOUSEHOLD#{householdId}` + `PROFILE#main` | 家庭プロフィール |
| MemberProfile | `HOUSEHOLD#{householdId}` + `MEMBER#{memberId}` | 配偶者プロフィール |
| MessageSession | `HOUSEHOLD#{householdId}` + `SESSION#{sessionId}` | 入力セッション |
| RewriteCandidate | `HOUSEHOLD#{householdId}` + `REWRITE#{sessionId}` | 言い換え結果 |
| FamilyMeetingResult | `HOUSEHOLD#{householdId}` + `MEETING#{meetingId}` | AI家族会議結果 |
| VitalScenario | `HOUSEHOLD#{householdId}` + `VITAL_SCENARIO#{scenarioId}` | Pro将来拡張の疑似バイタル |

## 6. 対案・将来進化形

| 構成 | 位置付け |
|---|---|
| Bedrock Flows | 生成AIワークフローを視覚的に説明したい場合の対案 |
| AppSync Events | AI家族会議をリアルタイム配信したい場合の拡張 |
| AgentCore | 妻AI、夫AI、調停AIを本格的なエージェントとして扱う将来進化形 |

## 7. 安全制御

Safety Control Component は、アプリケーションルールと Bedrock Guardrails をまとめて扱う。

対象:

- 入力文
- 言い換え候補
- AI家族会議の発話
- ラスト画面の表現

制御:

- 警告
- 再生成
- 停止
- 審査員向け安全説明

## 8. PBT一部適用サマリー

| PBTルール | 対象候補 |
|---|---|
| PBT-02 | HouseholdProfile, MessageSession, FamilyMeetingResult, VitalScenario のJSON保存・復元 |
| PBT-03 | RiskScoreの範囲、RiskLevel分類、PlanFlowDefinitionの必須ステップ |
| PBT-07 | MessageIntent, HouseholdProfile, VitalScenario のドメインジェネレータ |
| PBT-08 | 後続実装時のseed記録とshrinking |
| PBT-09 | 実装言語決定後のPBTフレームワーク選択 |

Application Design時点では実装コードがないため、PBTのブロッキング不適合はない。

## 9. 公式ドキュメント参照

- Amazon Bedrock AgentCore overview: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- Amazon Bedrock Flows: https://docs.aws.amazon.com/bedrock/latest/userguide/flows.html
- AWS AppSync Events concepts: https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-concepts.html
- Amazon Bedrock Guardrails: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-how.html
- Step Functions with Lambda: https://docs.aws.amazon.com/lambda/latest/dg/with-step-functions.html

## 10. Extension Compliance

| Extension | 状態 | Application Designでの扱い |
|---|---|---|
| Security Baseline | 無効 | PoC/プロトタイプ扱いのためスキップ |
| Property-Based Testing | 一部有効 | PBT-02, PBT-03, PBT-07, PBT-08, PBT-09を後続工程へ渡す |

## 11. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- 複雑な図の代替として、表と番号付きデータフローを使用した
- Markdown表とコード風識別子の構文を確認した
