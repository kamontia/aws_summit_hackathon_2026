# Component Dependency: 夫婦間API Gateway

## 1. 依存関係方針

主案は、Mobile Demo Web UI が API Gateway Facade を呼び出し、API Gateway Facade が Step Functions または Lambda 経由で各コンポーネントを呼び出す構成とする。

依存方向は、UIからサービス層、サービス層からドメインコンポーネント、ドメインコンポーネントからデータストアと生成AIサービスへ流す。  
生成AIと安全制御は直接UIに依存しない。

## 2. 依存関係マトリクス

| From | To | 依存内容 |
|---|---|---|
| C-01 Mobile Demo Web UI | C-03 API Gateway Facade | 画面操作からAPIを呼び出す |
| C-03 API Gateway Facade | C-02 Demo Auth and Household Context | デモログイン時に家庭コンテキストを解決する |
| C-03 API Gateway Facade | C-04 Plan Flow Controller | プラン別フローを開始する |
| C-04 Plan Flow Controller | C-05 Message Intake Component | プランに応じた入力処理を選ぶ |
| C-05 Message Intake Component | C-10 Household Data Store | 家庭設定を読み、入力セッションを保存する |
| C-04 Plan Flow Controller | C-06 Risk and Landmine Analyzer | Free / Standard / Proに応じてリスク判定を呼ぶ |
| C-06 Risk and Landmine Analyzer | C-08 Rewrite Generation Component | 生成プロンプト用の制約を渡す |
| C-08 Rewrite Generation Component | C-07 Safety Control Component | 生成候補を安全検査へ渡す |
| C-07 Safety Control Component | C-08 Rewrite Generation Component | 再生成条件を返す |
| C-08 Rewrite Generation Component | C-10 Household Data Store | 言い換え結果を保存する |
| C-08 Rewrite Generation Component | C-09 AI Family Meeting Component | 家族会議の入力として言い換え結果を渡す |
| C-09 AI Family Meeting Component | C-07 Safety Control Component | 会議発話を安全検査する |
| C-09 AI Family Meeting Component | C-10 Household Data Store | 会議結果を保存する |
| C-11 Pro Vital Timing Component | C-04 Plan Flow Controller | Pro将来フローの説明条件を返す |
| C-12 Presentation Component | C-10 Household Data Store | 審査員向けサマリーデータを読む |
| C-12 Presentation Component | C-07 Safety Control Component | 安全配慮の説明を受け取る |

## 3. 外部AWSサービス依存

| Component | AWS Service | 依存目的 |
|---|---|---|
| C-03 API Gateway Facade | Amazon API Gateway | HTTP API入口 |
| C-04 Plan Flow Controller | AWS Step Functions | プラン別フローの制御 |
| C-05 Message Intake Component | AWS Lambda | 入力検証と正規化 |
| C-06 Risk and Landmine Analyzer | AWS Lambda | ルールベース判定 |
| C-07 Safety Control Component | Amazon Bedrock Guardrails | 入力・生成結果の安全制御 |
| C-08 Rewrite Generation Component | Amazon Bedrock | 言い換え生成 |
| C-09 AI Family Meeting Component | Amazon Bedrock | 会議発話と合意案生成 |
| C-10 Household Data Store | Amazon DynamoDB | 家庭データ保存 |
| C-12 Presentation Component | AWS Lambda | 説明サマリー生成 |

## 4. データフロー

### 4.1 デモログイン

1. Mobile Demo Web UI がデモログインを送信する
2. API Gateway Facade が Demo Auth and Household Context を呼び出す
3. Demo Auth and Household Context が Household Data Store から家庭プロフィールを取得する
4. Mobile Demo Web UI が佐藤家のデモ状態を表示する

### 4.2 Standard言い換え

1. Mobile Demo Web UI が不満・依頼を送信する
2. API Gateway Facade が Rewrite Orchestration Service を開始する
3. Plan Flow Controller が Standard Flow を選ぶ
4. Message Intake Component が入力を `MessageIntent` に正規化する
5. Risk and Landmine Analyzer が `RiskAssessment` を作る
6. Rewrite Generation Component がAmazon Bedrockで言い換え候補を生成する
7. Safety Control Component がアプリルールと Bedrock Guardrails で検査する
8. Household Data Store がセッション、リスク判定、言い換え候補を保存する
9. Mobile Demo Web UI が結果を表示する

### 4.3 AI家族会議

1. Mobile Demo Web UI がAI家族会議を開始する
2. AI Family Meeting Component が議題を作成する
3. 妻AI、夫AI、調停AIの論理役割を作成する
4. Amazon Bedrockで発話ターンを生成する
5. Safety Control Component が発話を検査する
6. AI Family Meeting Component が合意案を生成する
7. Presentation Component がラスト画面指標を作成する
8. Household Data Store が会議結果を保存する
9. Mobile Demo Web UI が「直接会話: 0件」を含む結果を表示する

### 4.4 Pro将来拡張

1. Presentation Component がProプラン説明を要求する
2. Pro Vital Timing Component が疑似バイタルシナリオを読み込む
3. タイミング推奨を作成する
4. Presentation Component が「タイミングすらAIに委ねる」価値とリスクを説明する

## 5. 循環依存の回避

| 方針 | 内容 |
|---|---|
| UIはドメインロジックを持たない | UIは表示とAPI呼び出しに集中する |
| Safety Controlは生成AIへ直接依存しない | 安全判定は生成結果の検査に集中する |
| Data Storeは生成AIへ依存しない | 保存と取得の責務に限定する |
| Presentation Componentは本体処理を実行しない | 説明用サマリーの生成に限定する |

## 6. 代替構成での依存差分

| 対案 | 依存関係の変化 |
|---|---|
| Bedrock Flows中心 | Step Functionsの一部がBedrock Flowsに置き換わり、プロンプト、Lambda、Guardrailsのノード接続として表現される |
| AppSync Events中心 | AI Family Meeting Componentから発話イベントをAppSync Eventsへ発行し、UIがWebSocket購読する |
| AgentCore中心 | 妻AI、夫AI、調停AIがAgentCore Runtime上のエージェント候補になり、AgentCore MemoryやGatewayが将来依存として加わる |

## 7. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- データフローは番号付きリストで表現した
- 複雑な図の代替として依存関係マトリクスを記載した
