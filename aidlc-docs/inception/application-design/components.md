# Components: 夫婦間API Gateway

## 1. 設計方針

Application Design の回答に基づき、主アーキテクチャは **基本サーバーレス構成** とする。  
Amazon Bedrock AgentCore は、妻AI、夫AI、調停AIをより本格的なエージェントとして扱う将来進化形として整理する。

| 項目 | 決定 |
|---|---|
| 主アーキテクチャ | API Gateway, Lambda, Step Functions, Amazon Bedrock, Bedrock Guardrails, DynamoDB |
| 対案・将来進化形 | AgentCore風構成 |
| 設計粒度 | 応募・ピッチ資料に使える論理設計 |
| AI家族会議 | 妻AI、夫AI、調停AIを論理コンポーネントとして分ける |
| データ設計 | DynamoDBテーブル設計とアクセスパターンまで含める |
| 安全制御 | アプリケーションルールと Bedrock Guardrails を1つの安全制御コンポーネントとして扱う |
| プラン差分 | Free / Standard / Pro ごとに処理フローを分けてデモ上の差を強く見せる |
| Proバイタル | 将来拡張コンポーネントとして扱う |
| ピッチ説明 | Presentation Component を含める |

## 2. コンポーネント一覧

| ID | Component | 目的 | MVP/将来 |
|---|---|---|---|
| C-01 | Mobile Demo Web UI | ユーザーと審査員が体験する画面を提供する | MVP |
| C-02 | Demo Auth and Household Context | デモアカウントと1家庭データを紐づける | MVP |
| C-03 | API Gateway Facade | フロントエンドから家庭内交渉APIを受ける | MVP |
| C-04 | Plan Flow Controller | Free / Standard / Pro の体験差を制御する | MVP |
| C-05 | Message Intake Component | 不満・依頼・謝罪などの入力を正規化する | MVP |
| C-06 | Risk and Landmine Analyzer | 地雷ワードと家庭内炎上リスクを判定する | MVP |
| C-07 | Safety Control Component | アプリルールと Bedrock Guardrails で危険表現を抑制する | MVP |
| C-08 | Rewrite Generation Component | Amazon Bedrockで言い換え文面を生成する | MVP |
| C-09 | AI Family Meeting Component | 妻AI、夫AI、調停AIの論理役割で合意案を生成する | MVP |
| C-10 | Household Data Store | DynamoDBに家庭設定、会話、会議結果を保存する | MVP |
| C-11 | Pro Vital Timing Component | 疑似バイタル情報を使った送信タイミング制御を説明する | 将来拡張 |
| C-12 | Presentation Component | AWS活用、事業性、ブラックユーモアの説明材料を提供する | MVP |

## 3. コンポーネント詳細

### C-01: Mobile Demo Web UI

目的:

- モバイルファーストのデモ体験を提供する
- 入力、リスク判定、言い換え、AI家族会議、ラスト画面を一連の体験として見せる

主な責務:

- デモログイン画面を表示する
- Free / Standard / Pro の比較を表示する
- 入力フォームと生成結果確認画面を表示する
- AI家族会議の発話ターンを表示する
- ラスト画面で「直接会話: 0件」などの指標を表示する

インターフェース:

- API Gateway Facade へリクエストを送る
- Presentation Component の説明データを表示する

### C-02: Demo Auth and Household Context

目的:

- 本番品質の認証ではなく、デモ用の家庭コンテキストを固定的に提供する

主な責務:

- デモアカウントを受け付ける
- デモ家庭IDを払い出す
- 家庭メンバー、プラン、初期プロフィールを取得する

インターフェース:

- `householdId` と `currentActorId` を返す
- Household Data Store を参照する

### C-03: API Gateway Facade

目的:

- フロントエンドから見た「夫婦間API Gateway」の入口を表現する

主な責務:

- デモログインAPIを受ける
- 入力送信APIを受ける
- リスク判定APIを受ける
- 言い換え生成APIを受ける
- AI家族会議開始APIを受ける
- ピッチ説明APIを受ける

インターフェース:

- Mobile Demo Web UI からHTTPリクエストを受ける
- Step FunctionsまたはLambdaへ処理を委譲する

### C-04: Plan Flow Controller

目的:

- 価格が高いほどユーザーが何も考えなくてよい、というテーマ性を設計に反映する

主な責務:

- Free / Standard / Pro の処理フローを選択する
- Freeでは手動指定を要求する
- Standardでは基本自動調整を実行する
- Proでは将来拡張として疑似バイタル連携を説明する

インターフェース:

- `planType` を受け取り、実行するフロー定義を返す
- Risk and Landmine Analyzer、Rewrite Generation Component、Pro Vital Timing Component の呼び出し条件を制御する

### C-05: Message Intake Component

目的:

- ユーザーの入力を後続処理で扱いやすい構造に変換する

主な責務:

- 空入力を拒否する
- 入力文、感情、家庭内タスク、相手への依頼を正規化する
- Freeプランの手動指示を受け取る
- Standardプランの不足情報を家庭設定から補う

インターフェース:

- Mobile Demo Web UI から入力データを受け取る
- 正規化済みの `MessageIntent` を返す

### C-06: Risk and Landmine Analyzer

目的:

- 家庭内炎上リスクを可視化し、生成AIに渡す注意点を作る

主な責務:

- 地雷ワードを検出する
- 家庭内炎上リスクスコアを計算する
- リスクラベルを付与する
- 後続の言い換え生成に使う制約を作る

インターフェース:

- `MessageIntent` を受け取る
- `RiskAssessment` を返す

### C-07: Safety Control Component

目的:

- ブラックユーモアを維持しつつ、攻撃的・強制的・操作的な文面を抑制する

主な責務:

- アプリケーションルールで明確な禁止表現を検出する
- Bedrock Guardrails による入力・出力チェックを呼び出す
- 高リスク時の警告、再生成、停止を判定する
- 自動送信はデモ演出であり、本番では慎重に扱うことをPresentation Componentへ渡す

インターフェース:

- 入力文、生成候補、会議発話を検査する
- `SafetyDecision` を返す

### C-08: Rewrite Generation Component

目的:

- 元の意図を保ちながら、責める表現を弱めた文面を生成する

主な責務:

- Amazon Bedrockへプロンプトを渡す
- プラン別の生成条件を反映する
- RiskAssessment を使って避けるべき表現を制御する
- 生成候補をSafety Control Componentへ渡す

インターフェース:

- `MessageIntent`, `RiskAssessment`, `PlanFlowDefinition` を受け取る
- `RewriteCandidate` を返す

### C-09: AI Family Meeting Component

目的:

- 妻AI、夫AI、調停AIが合意形成するように見せ、便利さと怖さを同時に表現する

主な責務:

- 妻AI、夫AI、調停AIの論理役割を定義する
- 家庭内議題を抽出する
- 会話ターンを生成する
- 合意案とラスト画面指標を生成する

インターフェース:

- `RewriteCandidate` と家庭コンテキストを受け取る
- `FamilyMeetingResult` を返す

### C-10: Household Data Store

目的:

- デモ家庭、会話、生成結果、会議結果をDynamoDBに保存する

主な責務:

- 家庭プロフィールを保存する
- プラン設定を保存する
- 入力文と生成結果を保存する
- AI家族会議結果を保存する
- Presentation Componentが使うサマリーを提供する

インターフェース:

- DynamoDBテーブル `HouseholdGatewayTable` を提供する
- PK/SKアクセスパターンで家庭単位のデータを取得する

### C-11: Pro Vital Timing Component

目的:

- Proプランの将来拡張として、疑似バイタル情報による送信タイミング制御を説明する

主な責務:

- 疑似バイタル情報の例を管理する
- タイミング推奨の説明を作る
- 将来のスマートウォッチ連携時に必要な同意、監査、手動承認の論点を保持する

インターフェース:

- `VitalScenario` を受け取る
- `TimingRecommendation` を返す

### C-12: Presentation Component

目的:

- 審査員向けに、AWS活用、事業性、テーマ適合、安全配慮を説明する

主な責務:

- 主案と対案のAWS構成を説明する
- Free / Standard / Pro の事業性を説明する
- ラスト画面のブラックユーモアを補足する
- 安全制御と将来拡張の境界を説明する

インターフェース:

- `ArchitectureSummary`, `PlanComparison`, `SafetySummary` を返す

## 4. DynamoDBテーブル設計

Application Design時点では、単一テーブル設計を候補とする。

### Table: HouseholdGatewayTable

| 属性 | 内容 |
|---|---|
| PK | エンティティの親キー。例: `HOUSEHOLD#demo-sato` |
| SK | エンティティ種別と時系列キー。例: `PROFILE#main`, `SESSION#2026-05-03T09:00:00Z` |
| entityType | `HouseholdProfile`, `MemberProfile`, `MessageSession`, `RewriteCandidate`, `FamilyMeetingResult`, `VitalScenario` |
| planType | `Free`, `Standard`, `Pro` |
| createdAt | 作成日時 |
| updatedAt | 更新日時 |
| payload | 各エンティティのJSON本文 |

### GSI候補

| Index | PK | SK | 用途 |
|---|---|---|---|
| GSI1 | `entityType` | `createdAt` | デモ用に最新の会話や会議結果を取得する |
| GSI2 | `planType` | `createdAt` | プラン別の比較データを表示する |

### 主なアクセスパターン

| ID | Access Pattern | キー設計 |
|---|---|---|
| AP-01 | デモ家庭プロフィールを取得する | PK=`HOUSEHOLD#demo-sato`, SK=`PROFILE#main` |
| AP-02 | 家庭メンバーを取得する | PK=`HOUSEHOLD#demo-sato`, SK begins_with `MEMBER#` |
| AP-03 | 会話セッションを保存する | PK=`HOUSEHOLD#demo-sato`, SK=`SESSION#{sessionId}` |
| AP-04 | 言い換え候補を保存する | PK=`HOUSEHOLD#demo-sato`, SK=`REWRITE#{sessionId}` |
| AP-05 | AI家族会議結果を保存する | PK=`HOUSEHOLD#demo-sato`, SK=`MEETING#{meetingId}` |
| AP-06 | Pro疑似バイタルシナリオを取得する | PK=`HOUSEHOLD#demo-sato`, SK begins_with `VITAL_SCENARIO#` |
| AP-07 | 審査員向けサマリーを取得する | GSI1で `FamilyMeetingResult` の最新データを取得する |

## 5. PBT一部適用の設計上の受け皿

| PBTルール | Application Designでの扱い | 対象候補 |
|---|---|---|
| PBT-02 | JSON保存・復元の往復性を後続へ渡す | HouseholdProfile, MessageSession, FamilyMeetingResult, VitalScenario |
| PBT-03 | リスクスコア、分類、プラン分岐の不変条件を後続へ渡す | RiskAssessment, PlanFlowDefinition, TimingRecommendation |
| PBT-07 | ドメイン型ジェネレータが必要な対象を明記する | MessageIntent, HouseholdProfile, VitalScenario |
| PBT-08 | 後続のテスト実行でseedとshrinkingを要求する | Construction Phase |
| PBT-09 | 実装言語確定後にPBTフレームワークを選ぶ | Construction Phase |

## 6. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- Markdown表と箇条書きのみで構成した
- 複雑な図の代替として、コンポーネント表とアクセスパターン表を記載した
