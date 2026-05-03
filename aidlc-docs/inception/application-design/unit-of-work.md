# Unit of Work: 夫婦間API Gateway

## 1. 分解方針

Units Generation Planning の承認済み回答に基づき、夫婦間API Gateway を **5つの中粒度Unit of Work** に分解する。

このUnitは、microservicesの分割ではなく、少人数ハッカソン向けの **実装・資料整理上の論理単位** である。  
後続で実装する場合、コードは単一アプリとして `src/`, `tests/`, `config/` を基本に整理する。

## 2. 確定方針

| 項目 | 方針 |
|---|---|
| Unit粒度 | 中粒度 |
| Unit数 | 5 Unit |
| Story配置の優先 | ピッチ訴求順 |
| プラン差分 | Safety and Plan Control Unitが統括 |
| AI家族会議 | 独立Unit |
| DynamoDB設計 | Data and Presentation Architecture Unitが所有 |
| 所有境界 | 少人数ハッカソン向けの論理分解 |
| コード組織 | Greenfield single unit。`src/`, `tests/`, `config/` |
| Units生成後 | Constructionには進まず、Inception Phase完了として一度停止 |

## 3. Unit一覧

| Unit | 名称 | ピッチ上の役割 | 主な責務 |
|---|---|---|---|
| U-01 | Demo Experience UI | 家庭内あるあるを短時間で伝える | モバイルWeb体験、デモログイン、入力、比較表示、ラスト表示 |
| U-02 | Message Rewrite Core | Standard中心の価値を伝える | 入力正規化、地雷ワード検出、炎上リスク、言い換え生成 |
| U-03 | Safety and Plan Control | 事業性と安全配慮を両立する | Free / Standard / Pro差分、安全制御、Pro将来拡張 |
| U-04 | AI Family Meeting | ブラックユーモアの中核を担う | 妻AI、夫AI、調停AIの論理会議、合意案、直接会話0件 |
| U-05 | Data and Presentation Architecture | AWS活用と応募資料の説得力を担う | DynamoDB設計、AWS構成説明、ピッチ用サマリー |

## 4. Unit詳細

### U-01: Demo Experience UI

目的:

- 審査員とユーザーが、家庭内あるあるからブラックユーモアまでを一連の体験として理解できるようにする。

責務:

- デモログイン画面を提供する
- 佐藤家のデモ状態を表示する
- 不満・依頼の入力画面を提供する
- Free / Standard / Pro の比較表示を提供する
- AI家族会議画面とラスト画面を表示する

所有コンポーネント:

- C-01 Mobile Demo Web UI
- C-02 Demo Auth and Household ContextのUI接点

主なストーリー:

- US-01
- US-02
- US-05
- US-08

完了条件:

- デモ開始からラスト画面までの画面構成が説明できる
- 「家庭内あるある」「事業性」「直接会話: 0件」が短時間で伝わる

### U-02: Message Rewrite Core

目的:

- Standardプランの中核価値である「言いたいことだけ入力すれば、AIが言い方を整える」を成立させる。

責務:

- 入力文を `MessageIntent` に正規化する
- 地雷ワードを検出する
- 家庭内炎上リスクを計算する
- Amazon Bedrock向けの言い換え条件を作る
- 言い換え候補と説明を生成する

所有コンポーネント:

- C-05 Message Intake Component
- C-06 Risk and Landmine Analyzer
- C-08 Rewrite Generation Component

主なストーリー:

- US-02
- US-03
- US-04

PBT一部適用候補:

- 地雷ワード検出の不変条件
- RiskScoreの0から100の範囲制約
- `MessageIntent` のJSON保存・復元

完了条件:

- 不満入力、地雷ワード検出、リスク判定、言い換え生成の責務境界が明確である
- Standardプランの価値が他Unitから参照できる

### U-03: Safety and Plan Control

目的:

- Free / Standard / Pro のマネタイズ構造と、安全制御の説明を統括する。

責務:

- Free / Standard / Pro のPlan Flowを定義する
- Freeでは手動指定が必要であることを示す
- Standardでは基本自動調整を示す
- Proでは疑似バイタル情報によるタイミング制御を将来拡張として扱う
- アプリケーションルールとBedrock Guardrailsによる安全制御をまとめる

所有コンポーネント:

- C-04 Plan Flow Controller
- C-07 Safety Control Component
- C-11 Pro Vital Timing Component

主なストーリー:

- US-05
- US-06
- FS-01

PBT一部適用候補:

- PlanTypeごとの必須ステップ不変条件
- Freeのみ手動指示が必要になる分岐条件
- `VitalScenario` の正規化と範囲制約

完了条件:

- 価格が高いほどユーザーが考えなくなる構造が説明できる
- ブラックユーモアが危険サービスに見えないよう、安全制御の境界が明確である

### U-04: AI Family Meeting

目的:

- 「人をダメにする」テーマの中核として、人間が直接話さずAI同士が家庭内合意を作る体験を設計する。

責務:

- 家庭内議題を生成する
- 妻AI、夫AI、調停AIの論理役割を定義する
- 会議ターンを生成する
- 合意案をまとめる
- ラスト画面指標を生成する

所有コンポーネント:

- C-09 AI Family Meeting Component

主なストーリー:

- US-07
- US-08

完了条件:

- AI家族会議が、便利さと怖さの両方を表現できる
- 「直接会話: 0件」「家庭内摩擦: 低下」「関係性の自律性: 低下」がラスト演出として成立する

### U-05: Data and Presentation Architecture

目的:

- DynamoDB設計、AWS構成、応募・ピッチ資料に使う説明材料を統括する。

責務:

- `HouseholdGatewayTable` の単一テーブル設計を所有する
- HouseholdProfile, MessageSession, RewriteCandidate, FamilyMeetingResult, VitalScenario のデータ契約を定義する
- API Gateway, Lambda, Step Functions, Bedrock, Guardrails, DynamoDB の採用理由を説明する
- AgentCore風将来進化形を資料上で説明する
- 審査員向けのAWS活用サマリーを作る

所有コンポーネント:

- C-03 API Gateway Facade
- C-10 Household Data Store
- C-12 Presentation Component

主なストーリー:

- US-01
- US-09
- FS-01

PBT一部適用候補:

- HouseholdProfileのJSON保存・復元
- FamilyMeetingResultのJSON保存・復元
- VitalScenarioのJSON保存・復元

完了条件:

- AWSマネージドサービスの使いどころを説明できる
- 各Unitが依存するデータ契約を確認できる
- 応募資料に転記できるアーキテクチャ説明が揃う

## 5. Greenfieldコード組織戦略

現時点では実装しないが、後続でConstruction Phaseへ進む場合は、Greenfield single unit構成を候補とする。

```text
<WORKSPACE-ROOT>/
├── src/
│   ├── app/
│   ├── modules/
│   │   ├── demo-experience/
│   │   ├── message-rewrite/
│   │   ├── safety-plan-control/
│   │   ├── ai-family-meeting/
│   │   └── data-presentation/
│   └── shared/
├── tests/
│   ├── unit/
│   ├── property/
│   └── integration/
├── config/
└── aidlc-docs/
```

注記:

- `aidlc-docs/` はドキュメント専用であり、アプリケーションコードは置かない
- Unitはmicroservicesではなく、単一アプリ内の論理モジュール候補として扱う
- Property-Based Testingを行う場合、`tests/property/` を候補とする

## 6. Unit境界の検証

| 観点 | 判定 | 理由 |
|---|---|---|
| Story coverage | Compliant | US-01からUS-09、FS-01をすべて割り当てた |
| Unit granularity | Compliant | 中粒度で、5 Unitに収まっている |
| AI Family Meeting独立性 | Compliant | U-04として独立させた |
| Plan差分の集約 | Compliant | U-03に集約し、他UnitはPlan Flowに依存する |
| Data ownership | Compliant | U-05がDynamoDB設計とデータ契約を所有する |
| Code organization | Compliant | Greenfield single unitとして候補を記載した |

## 7. Extension Compliance

| Extension | 状態 | Units Generationでの扱い |
|---|---|---|
| Security Baseline | 無効 | PoC/プロトタイプ扱いのためスキップ |
| Property-Based Testing | 一部有効 | PBT対象候補をU-02, U-03, U-05へ割り当てた |

## 8. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- コード組織は単純なテキストツリーとして記載した
- Markdown表とコードブロックの構文を単純に保った
