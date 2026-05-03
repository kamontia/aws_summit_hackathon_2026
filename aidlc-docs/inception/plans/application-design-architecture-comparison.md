# Application Design Architecture Comparison: 夫婦間API Gateway

## 1. 目的

Application Design Plan の Question 1 で、主アーキテクチャが未決定となったため、候補ごとの特徴を比較する。

この比較は、応募・ピッチ資料に使える論理設計を前提にしている。実装フェーズに進む場合は、リージョン、利用可能サービス、ハッカソン環境の制約を改めて確認する。

## 2. 比較対象

| 候補 | 中心サービス | 狙い |
|---|---|---|
| A | API Gateway, Lambda, Step Functions, Amazon Bedrock, DynamoDB | 実現性と説明しやすさを重視する |
| B | Amazon Bedrock Flows | 生成AIワークフローの見せやすさを重視する |
| C | AWS AppSync Events | AI同士の家族会議をリアルタイムUIとして見せる |
| D | Amazon Bedrock AgentCore | 妻AI、夫AI、調停AIのエージェント性を強く見せる |

## 3. 候補A: 基本サーバーレス構成

### 構成イメージ

- API Gateway: モバイルWebアプリからのAPI入口
- Lambda: 地雷ワード検出、プラン分岐、安全制御前処理、整形処理
- Step Functions: 入力検証、リスク判定、生成AI呼び出し、結果保存、エラー処理の流れを管理
- Amazon Bedrock: 言い換え生成、AI家族会議の発話生成
- Bedrock Guardrails: 入力と生成結果の安全制御
- DynamoDB: デモ家庭、プラン、会話履歴、家族会議結果の保存

### 強み

- ハッカソンで説明しやすく、実装に進む場合も現実的である
- Step Functions により、処理順序、分岐、エラー処理を図として説明しやすい
- DynamoDBのテーブル設計まで落としやすく、Question 4 の回答 C と相性がよい
- Free / Standard / Pro を別フローとして表現しやすく、Question 6 の回答 C と相性がよい

### 弱み

- 「最新・変わり種」の印象はやや弱い
- AI同士の家族会議は、見せ方を工夫しないと単なる順次処理に見える
- エージェント性は論理表現に留まりやすい

### この企画への適合

最も安定した主案。応募資料では「夫婦間API Gateway」という名前とも整合し、API Gatewayを入口にした家庭内交渉APIとして説明できる。

一方で新規性を補うため、AgentCore風の論理コンポーネントや AppSync Events のリアルタイム案を対案として添えるのがよい。

## 4. 候補B: Bedrock Flows中心構成

### 構成イメージ

- Bedrock Flows: 地雷ワード検出、リスク判定、言い換え、家族会議生成をノードとして接続
- Lambda Node: ルールベース判定やプラン分岐
- Prompt Node: 言い換えや調停発話生成
- DynamoDB: 入力、家庭設定、結果保存
- Bedrock Guardrails: Prompt NodeやKnowledge Base Nodeへの安全制御

### 強み

- 生成AIワークフローを視覚的に説明しやすい
- 「AIが家庭内会話を処理している」感じを審査員に伝えやすい
- Bedrockのマネージド感が強く、AWS Summit向けの見栄えがよい
- Flowのバージョンやエイリアスで、テストからデプロイへの流れを説明しやすい

### 弱み

- Free / Standard / Pro の派手な差分や、細かいUI向け状態管理は別途アプリ側で整理が必要
- DynamoDBアクセスパターンやAPI設計は、Flowsだけでは完結しない
- ハッカソン実装では、Flow定義に寄せすぎると修正速度が落ちる可能性がある

### この企画への適合

応募資料上の見せ方は強い。特に「家庭内炎上リスク判定 -> 言い換え -> Guardrails -> AI家族会議」という流れをAWSサービスとして見せたい場合に有効。

ただし、今回の回答では Application Design の粒度が「応募・ピッチ資料向け」でありつつ、データモデルはDynamoDBテーブル設計まで求めている。そのため、主案にするならアプリ側コンポーネント設計を別途厚めに書く必要がある。

## 5. 候補C: AppSync Events中心構成

### 構成イメージ

- AppSync Events: AI同士の家族会議の発話イベントをWebSocketで配信
- Channel: 家庭ごと、会議セッションごとのイベント流
- Event Handler: 発話イベントの変換、購読制御、必要に応じたLambda連携
- Amazon Bedrock: 会議ターンや合意案の生成
- DynamoDB: 会議ログと合意結果の保存

### 強み

- AI同士の家族会議をリアルタイムに流すデモと相性がよい
- 「妻AIが発言 -> 夫AIが反論 -> 調停AIがまとめる」という演出を強くできる
- WebSocketの接続管理を自前で持たずに済む
- AppSync Events はHTTPとWebSocketでイベント発行し、WebSocketで購読できるため、リアルタイムUIの説明に向いている

### 弱み

- 今回の主成果物が応募・ピッチ資料なら、リアルタイム配信は過剰になる可能性がある
- 入力、リスク判定、言い換え、安全制御、保存の主処理は別途必要
- 企画の核が「言い換え」と「プラン差分」なら、AppSync Eventsだけを主案にすると軸がUI演出へ寄りすぎる

### この企画への適合

AI家族会議のデモ映えを最優先する場合は強い。  
ただし、Application Design全体の主案というより、家族会議画面の拡張候補として置くのが扱いやすい。

## 6. 候補D: AgentCore中心構成

### 構成イメージ

- AgentCore Runtime: 妻AI、夫AI、調停AIなどのエージェント実行環境
- AgentCore Memory: 家庭ごとの会話履歴、好み、地雷傾向を記憶する構想
- AgentCore Gateway: LambdaやAPIをエージェント用ツールとして公開する構想
- AgentCore Observability / Evaluations: エージェントの挙動確認、評価、改善の説明
- Bedrock Guardrails: 入力・出力の安全制御

### 強み

- 「AI同士が家族会議をする」という企画コンセプトとの相性が最も強い
- 妻AI、夫AI、調停AIを論理コンポーネントとして分ける Question 3 の回答 B と相性がよい
- AWSの新しさ、変わり種感、エージェント基盤の印象を出しやすい
- Memory、Gateway、Policy、Observability などを使うと、将来の事業拡張を語りやすい

### 弱み

- 応募資料だけなら魅力的だが、実装に進む場合は利用可能リージョン、アカウント権限、学習コストの確認が必要
- MVPの本質である「入力 -> リスク判定 -> 言い換え -> 結果表示」には、エージェント基盤がやや重い可能性がある
- 安全制御やデータ保存は別サービスとの組み合わせが必要

### この企画への適合

コンセプトの刺さりは強い。  
ただし、Question 2 が「応募・ピッチ資料に使える論理設計まで」であるため、主案にすると実現性より先進性を強調する設計になる。

「主案は基本サーバーレス、対案または将来進化形としてAgentCore」という整理が、実現性と新規性のバランスを取りやすい。

## 7. 推奨判断

### 推奨: Aを主案、Dを対案または将来進化形

理由:

- 応募・ピッチ資料で最も説明しやすい
- 「夫婦間API Gateway」というサービス名とAPI Gateway主案が一致する
- DynamoDBテーブル設計まで落としやすい
- Free / Standard / Pro の処理フロー差分を明確に描ける
- AI同士の家族会議は、論理上は妻AI・夫AI・調停AIに分けつつ、実装上は単一オーケストレーションでも成立する
- AgentCoreを対案に置くことで、AWSの新しさと将来拡張を補える

### 採用時の書き方

Application Design では次のように整理するのがよい。

| レイヤー | 主案 | 対案・拡張 |
|---|---|---|
| API入口 | API Gateway | AppSync Events for real-time family meeting |
| オーケストレーション | Step Functions | Bedrock Flows |
| AI生成 | Amazon Bedrock | AgentCore Runtime as future multi-agent runtime |
| 安全制御 | Bedrock Guardrails + app rules | AgentCore Policy as future agent tool control |
| データ保存 | DynamoDB | AgentCore Memory as future household memory |
| デモ演出 | Web UI + generated turns | AppSync Events for live meeting stream |

## 8. 公式ドキュメント参照

- Amazon Bedrock AgentCore overview: https://docs.aws.amazon.com/bedrock-agentcore/latest/devguide/what-is-bedrock-agentcore.html
- Amazon Bedrock Flows: https://docs.aws.amazon.com/bedrock/latest/userguide/flows.html
- AWS AppSync Events concepts: https://docs.aws.amazon.com/appsync/latest/eventapi/event-api-concepts.html
- Amazon Bedrock Guardrails: https://docs.aws.amazon.com/bedrock/latest/userguide/guardrails-how.html
- Step Functions with Lambda: https://docs.aws.amazon.com/lambda/latest/dg/with-step-functions.html

## 9. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- Markdown表と箇条書きのみで構成した
- 公式ドキュメントURLを参照として記載した
