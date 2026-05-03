# Services: 夫婦間API Gateway

## 1. サービス層方針

主案は、API Gatewayを入口にし、LambdaとStep Functionsで処理を組み立て、Amazon BedrockとBedrock Guardrailsを呼び出し、DynamoDBへ保存するサーバーレス構成とする。

AgentCoreは、将来の「妻AI、夫AI、調停AIを本物のエージェントとして動かす」進化形として扱う。

## 2. サービス一覧

| ID | Service | 主な責務 | 対応コンポーネント |
|---|---|---|---|
| S-01 | Demo Session Service | デモログインと家庭コンテキスト解決 | C-02, C-10 |
| S-02 | Message Session Service | 入力セッションの作成と保存 | C-03, C-05, C-10 |
| S-03 | Risk Assessment Service | 地雷ワード検出と炎上リスク判定 | C-06 |
| S-04 | Rewrite Orchestration Service | プラン別の言い換え生成フローを実行 | C-04, C-07, C-08 |
| S-05 | Family Meeting Service | AI家族会議の議題、発話、合意案を生成 | C-09 |
| S-06 | Presentation Service | AWS活用、事業性、安全配慮、風刺を説明 | C-12 |
| S-07 | Pro Future Timing Service | Pro将来拡張の疑似バイタルタイミング制御を説明 | C-11 |

## 3. AWSサービス対応

| AWSサービス | 用途 | 採用理由 |
|---|---|---|
| Amazon API Gateway | フロントエンドからのAPI入口 | 企画名「夫婦間API Gateway」と整合し、家庭内交渉APIとして説明しやすい |
| AWS Lambda | 入力正規化、地雷ワード検出、プラン分岐、整形処理 | 小さな処理単位をサーバーレスに分けられる |
| AWS Step Functions | Free / Standard / Pro の処理フロー、Bedrock呼び出し、安全制御、保存を順序立てる | プラン別の体験差をワークフローとして説明しやすい |
| Amazon Bedrock | 言い換え生成、AI家族会議の発話生成 | 生成AIの中核 |
| Amazon Bedrock Guardrails | 入力・生成結果の安全制御 | 攻撃的、強制的、操作的な表現を抑制する説明に使える |
| Amazon DynamoDB | 家庭設定、会話履歴、言い換え結果、会議結果の保存 | デモ家庭ごとのデータ分離とアクセスパターン説明に向く |
| AWS AppSync Events | 将来または対案として、家族会議発話をリアルタイム配信 | デモ映えするAI家族会議UIに向く |
| Amazon Bedrock Flows | 対案として、生成AIワークフローを視覚的に説明 | Bedrock中心の見せ方を強められる |
| Amazon Bedrock AgentCore | 将来進化形として、妻AI、夫AI、調停AIをエージェント化 | エージェント性と新しさを補える |

## 4. オーケストレーション

### 4.1 Free Flow

目的:

- ユーザーが細かく指定しないとよい結果が出にくいことを見せる
- 設定が静的なため、相手を逆撫でする可能性があることを表現する

処理:

1. Message Intake Component が入力と手動指示を受け取る
2. Plan Flow Controller が Free Flow を選ぶ
3. Risk and Landmine Analyzer が地雷ワードを検出する
4. Rewrite Generation Component が手動指示を反映して生成する
5. Safety Control Component が生成候補を検査する
6. Household Data Store が結果を保存する
7. Presentation Component が「Freeは人間がまだ頑張る必要がある」と説明する

### 4.2 Standard Flow

目的:

- MVPの基本体験として、ユーザーが言いたいことだけ入力すればAIが自動調整する状態を見せる

処理:

1. Message Intake Component が入力を正規化する
2. Plan Flow Controller が Standard Flow を選ぶ
3. Household Data Store が家庭プロフィールと地雷傾向を返す
4. Risk and Landmine Analyzer が炎上リスクを判定する
5. Rewrite Generation Component がBedrockで言い換え候補を生成する
6. Safety Control Component がBedrock Guardrailsとアプリルールで検査する
7. 必要に応じて再生成する
8. Household Data Store が結果を保存する
9. Mobile Demo Web UI が生成結果を表示する

### 4.3 AI Family Meeting Flow

目的:

- 人間が直接話さず、AI同士が家庭内合意を作る怖さを見せる

処理:

1. Family Meeting Service が言い換え結果を受け取る
2. AI Family Meeting Component が議題を作る
3. 妻AI、夫AI、調停AIの論理役割を作る
4. Amazon Bedrockで会議ターンを生成する
5. Safety Control Component が発話を検査する
6. AI Family Meeting Component が合意案をまとめる
7. Presentation Component がラスト画面指標を作る
8. Household Data Store が会議結果を保存する

### 4.4 Pro Future Flow

目的:

- Proは「人間がタイミングすら考えなくてよい」将来拡張として説明する

処理:

1. Pro Future Timing Service が疑似バイタルシナリオを受け取る
2. Pro Vital Timing Component が状態を分類する
3. タイミング推奨を生成する
4. Presentation Component が将来の同意、監査、手動承認の論点を補足する

## 5. DynamoDBアクセス設計

### 5.1 単一テーブル

`HouseholdGatewayTable` を基本とする。

| Entity | PK | SK | 主な属性 |
|---|---|---|---|
| HouseholdProfile | `HOUSEHOLD#{householdId}` | `PROFILE#main` | members, planType, householdTraits |
| MemberProfile | `HOUSEHOLD#{householdId}` | `MEMBER#{memberId}` | role, communicationTraits, landmineHints |
| MessageSession | `HOUSEHOLD#{householdId}` | `SESSION#{sessionId}` | actorId, originalText, planType, status |
| RewriteCandidate | `HOUSEHOLD#{householdId}` | `REWRITE#{sessionId}` | rewrittenText, explanation, safetyDecision |
| FamilyMeetingResult | `HOUSEHOLD#{householdId}` | `MEETING#{meetingId}` | agenda, turns, agreement, satireMetrics |
| VitalScenario | `HOUSEHOLD#{householdId}` | `VITAL_SCENARIO#{scenarioId}` | stressLevel, sleepScore, heartRateTrend |

### 5.2 アクセスパターン

| Access Pattern | Service |
|---|---|
| デモログイン後に家庭プロフィールを読む | Demo Session Service |
| 入力セッションを作成する | Message Session Service |
| Standard Flowで家庭プロフィールを読む | Rewrite Orchestration Service |
| 言い換え結果を保存する | Rewrite Orchestration Service |
| AI家族会議結果を保存する | Family Meeting Service |
| 審査員向けラスト画面サマリーを読む | Presentation Service |

## 6. 対案・将来進化形

### 6.1 Bedrock Flows対案

Bedrock Flowsを主軸にすると、以下のノード構成として説明できる。

| Node | 用途 |
|---|---|
| Input Node | 入力文と家庭コンテキストを受け取る |
| Lambda Node | 地雷ワード検出とプラン分岐を行う |
| Prompt Node | 言い換え文面を生成する |
| Guardrail | 生成結果を安全制御する |
| Lambda Node | DynamoDBへ保存する |
| Prompt Node | AI家族会議を生成する |

### 6.2 AppSync Events対案

AppSync Eventsを使うと、AI家族会議の発話をイベントとしてリアルタイム配信できる。  
主案では必須にしないが、デモ映えを強める拡張として有効である。

### 6.3 AgentCore将来進化形

AgentCoreを使う場合、妻AI、夫AI、調停AIをAgentCore Runtime上のエージェントとして扱う構想にできる。  
AgentCore Memoryを家庭内の長期記憶、AgentCore Gatewayを家庭内タスクAPIのツール公開、AgentCore Policyをツール利用制御として説明できる。

## 7. Extension Compliance

| Extension | 状態 | Servicesでの扱い |
|---|---|---|
| Security Baseline | 無効 | PoC/プロトタイプ扱いのためスキップ |
| Property-Based Testing | 一部有効 | 実装時に純粋関数、JSON往復、リスクスコア不変条件へ適用する |

## 8. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- サービス間フローは番号付きリストで表現した
- AWSサービス名と用途はMarkdown表で整理した
