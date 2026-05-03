# Unit of Work Story Map: 夫婦間API Gateway

## 1. Story Map方針

Story Grouping は、ユーザー回答に基づき **ピッチ訴求順** を優先する。

説明順:

1. 家庭内あるある
2. Standard中心の価値
3. 事業性と安全配慮
4. ブラックユーモア
5. AWS活用

## 2. Story to Unit Map

| Story | 名称 | Primary Unit | Supporting Unit | MVP/将来 |
|---|---|---|---|---|
| US-01 | デモ家庭としてログインする | U-01 Demo Experience UI | U-05 Data and Presentation Architecture | MVP |
| US-02 | 言いにくい不満をそのまま入力する | U-01 Demo Experience UI | U-02 Message Rewrite Core | MVP |
| US-03 | 地雷ワードと家庭内炎上リスクを確認する | U-02 Message Rewrite Core | U-03 Safety and Plan Control | MVP |
| US-04 | StandardプランでAIに言い方をおまかせする | U-02 Message Rewrite Core | U-03 Safety and Plan Control | MVP |
| US-05 | FreeとStandardの差から事業性を見せる | U-03 Safety and Plan Control | U-01 Demo Experience UI, U-05 Data and Presentation Architecture | MVP |
| US-06 | Guardrailsとアプリルールで危険な文面を抑制する | U-03 Safety and Plan Control | U-02 Message Rewrite Core, U-04 AI Family Meeting | MVP |
| US-07 | AI同士の家族会議を見る | U-04 AI Family Meeting | U-02 Message Rewrite Core, U-03 Safety and Plan Control | MVP |
| US-08 | ラスト画面でブラックユーモアを伝える | U-04 AI Family Meeting | U-01 Demo Experience UI, U-05 Data and Presentation Architecture | MVP |
| US-09 | AWS活用の説明材料を得る | U-05 Data and Presentation Architecture | U-03 Safety and Plan Control | MVP |
| FS-01 | Proプランでバイタル情報から送信タイミングを制御する | U-03 Safety and Plan Control | U-05 Data and Presentation Architecture | 将来拡張 |

## 3. Unit別Story Map

### U-01: Demo Experience UI

| Story | 役割 |
|---|---|
| US-01 | デモログインと佐藤家の初期表示 |
| US-02 | 入力画面と家庭内あるあるの導入 |
| US-05 | Free / Standard比較の画面表示 |
| US-08 | ラスト画面の表示 |

主な価値:

- 審査員が短時間で企画の面白さを理解できる
- ユーザー体験の入口と出口をつなぐ

### U-02: Message Rewrite Core

| Story | 役割 |
|---|---|
| US-02 | 入力を構造化し、後続処理に渡す |
| US-03 | 地雷ワードと家庭内炎上リスクを判定する |
| US-04 | Standardプランの言い換え生成を成立させる |

主な価値:

- MVPの実用価値を担う
- PBT一部適用の中心になる

### U-03: Safety and Plan Control

| Story | 役割 |
|---|---|
| US-05 | Free / Standardの差と事業性を説明する |
| US-06 | Guardrailsとアプリルールで危険表現を抑制する |
| FS-01 | Pro将来拡張としてバイタルタイミング制御を説明する |

主な価値:

- 価格が上がるほど人が考えなくなる構造を担う
- ブラックユーモアへの懸念を安全制御でケアする

### U-04: AI Family Meeting

| Story | 役割 |
|---|---|
| US-07 | 妻AI、夫AI、調停AIの会議体験を作る |
| US-08 | 直接会話0件のブラックユーモアを成立させる |

主な価値:

- 「人をダメにする」テーマを最も強く表現する
- 便利さと怖さを同時に見せる

### U-05: Data and Presentation Architecture

| Story | 役割 |
|---|---|
| US-01 | デモ家庭データを提供する |
| US-09 | AWS活用説明を提供する |
| FS-01 | Pro将来拡張のデータ契約を支える |

主な価値:

- AWS Summit向けの説得力を担う
- 各Unitのデータ契約を安定させる

## 4. Epic to Unit Map

| Epic | 名称 | Primary Unit |
|---|---|---|
| E-01 | 夫婦間あるある導入 | U-01 Demo Experience UI |
| E-02 | Standard中心の言い換え体験 | U-02 Message Rewrite Core |
| E-03 | プラン差分と事業性 | U-03 Safety and Plan Control |
| E-04 | AI同士の家族会議 | U-04 AI Family Meeting |
| E-05 | 安全性と審査員向け説明 | U-03 Safety and Plan Control, U-05 Data and Presentation Architecture |

## 5. MVP / Future Boundary

| 対象 | MVP | 将来拡張 |
|---|---|---|
| デモログイン | U-01, U-05 | 本番認証、詳細な権限管理 |
| 不満入力 | U-01, U-02 | 音声入力、外部チャット連携 |
| 地雷ワード検出 | U-02 | 家庭ごとの学習最適化 |
| Standard言い換え | U-02, U-03 | 長期履歴に基づく自動最適化 |
| Free / Standard比較 | U-01, U-03 | 実課金、サブスクリプション管理 |
| Safety Control | U-03 | 本番品質の監査ログ、同意管理、手動承認 |
| AI家族会議 | U-04 | AgentCoreによる本格マルチエージェント化 |
| Proバイタル | U-03, U-05 | 実スマートウォッチ連携 |
| AWS説明 | U-05 | 実デプロイ、運用監視 |

## 6. Coverage Check

| Story | Covered | Notes |
|---|---|---|
| US-01 | Yes | U-01 primary, U-05 supporting |
| US-02 | Yes | U-01 primary, U-02 supporting |
| US-03 | Yes | U-02 primary |
| US-04 | Yes | U-02 primary |
| US-05 | Yes | U-03 primary |
| US-06 | Yes | U-03 primary |
| US-07 | Yes | U-04 primary |
| US-08 | Yes | U-04 primary, U-01 supporting |
| US-09 | Yes | U-05 primary |
| FS-01 | Yes | U-03 primary, U-05 supporting |

## 7. PBT Carry-Forward Map

| PBT対象 | Story | Unit | 後続工程への渡し方 |
|---|---|---|---|
| 地雷ワード検出 | US-03 | U-02 | Functional Designで不変条件を定義する |
| 炎上リスクスコア計算 | US-03 | U-02 | Functional Designでスコア範囲と分類境界を定義する |
| プラン別制御ロジック | US-04, US-05 | U-03 | Functional DesignでPlanTypeごとの必須ステップを定義する |
| 家庭設定JSON保存・復元 | US-01, US-07 | U-05 | Code GenerationでPBT-02の往復テスト候補にする |
| 疑似バイタル情報の正規化 | FS-01 | U-03 | Functional Designで範囲制約を定義する |

## 8. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- Story MapはMarkdown表で表現した
- すべてのUser StoryとFuture Storyを少なくとも1つのUnitへ割り当てた
