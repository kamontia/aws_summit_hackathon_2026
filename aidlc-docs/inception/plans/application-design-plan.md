# Application Design Plan: 夫婦間API Gateway

## 1. 目的

Application Design では、夫婦間API Gateway の MVP を構成する論理コンポーネント、サービス責務、主要メソッド、依存関係を整理する。

この段階では詳細な業務ロジックや実装コードは作らない。  
Construction Phase に進む場合に使えるよう、AWS構成、UI体験、安全制御、AI同士の家族会議、データ保存の境界を明確にする。

## 2. 前提コンテキスト

- [x] `requirements.md` を確認した
- [x] `stories.md` を確認した
- [x] `personas.md` を確認した
- [x] `execution-plan.md` を確認した
- [x] Greenfieldであり、既存アプリケーションコードは存在しないことを確認した
- [x] Security Baseline は無効化済みであることを確認した
- [x] Property-Based Testing は一部適用であることを確認した

## 3. 設計対象

### 3.1 MVP対象

- 簡易ログインとデモ家庭
- 不満・依頼入力
- 地雷ワード検出と家庭内炎上リスク判定
- Standard中心の言い換え生成
- Free / Standard / Pro のプラン差分表示
- AI同士の家族会議シミュレーション
- ラスト画面のブラックユーモア指標
- AWS活用説明に使える論理アーキテクチャ

### 3.2 将来拡張として扱う対象

- Proプランの実スマートウォッチ連携
- 本番品質の同意管理
- 本番品質の監査ログ
- 実際の自動メッセージ送信
- 詳細な権限管理

## 4. 実行手順

- [x] 1. 既存AIDLC成果物を読み込み、設計前提を整理する
- [x] 2. Application Design の計画ファイルを作成する
- [x] 3. ユーザーが設計質問の `[Answer]:` をすべて記入する
- [x] 4. 回答を検証し、未回答・曖昧回答・矛盾がないか確認する
- [x] 5. 曖昧さがある場合、追加の確認質問ファイルを作成する
- [x] 6. アーキテクチャ主案と対案の扱いを確定する
- [x] 7. MVPの主要コンポーネントを定義する
- [x] 8. コンポーネントごとの責務とインターフェースを定義する
- [x] 9. コンポーネントメソッドの高レベルシグネチャを定義する
- [x] 10. サービス層の責務とオーケストレーションを定義する
- [x] 11. コンポーネント間の依存関係と通信パターンを定義する
- [x] 12. PBT一部適用候補を設計上の純粋関数・データ変換へ紐づける
- [x] 13. `aidlc-docs/inception/application-design/components.md` を生成する
- [x] 14. `aidlc-docs/inception/application-design/component-methods.md` を生成する
- [x] 15. `aidlc-docs/inception/application-design/services.md` を生成する
- [x] 16. `aidlc-docs/inception/application-design/component-dependency.md` を生成する
- [x] 17. `aidlc-docs/inception/application-design/application-design.md` を生成する
- [x] 18. 生成物の整合性、Markdown構文、図表代替テキスト、拡張ルール適用状況を検証する
- [x] 19. Application Design 完了レビューを提示する

## 5. 生成予定成果物

| 成果物 | 目的 |
|---|---|
| `components.md` | コンポーネント名、目的、責務、インターフェースを整理する |
| `component-methods.md` | 各コンポーネントの主要メソッド、入力、出力、高レベル目的を整理する |
| `services.md` | サービス層の責務、オーケストレーション、AWSサービス候補との対応を整理する |
| `component-dependency.md` | 依存関係、通信パターン、データフローを整理する |
| `application-design.md` | 上記成果物を統合した設計サマリーを作成する |

## 6. 設計質問

以下の質問に回答してください。各質問の `[Answer]:` の後ろに、選択肢の文字を記入してください。  
どれにも合わない場合は最後の `X) Other` を選び、同じ行または次の行に希望内容を書いてください。

### Question 1
Application Design の主アーキテクチャはどの方針で整理しますか？

A) 基本サーバーレス構成を主案にし、AgentCore風構成を対案として整理する  
B) Bedrock Flows中心構成を主案にし、生成AIワークフローの見せやすさを重視する  
C) AppSync Events中心構成を主案にし、リアルタイム家族会議UIのデモ映えを重視する  
D) AgentCore中心構成を主案にし、妻AI・夫AI・調停AIのエージェント性を重視する  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 2
Application Design の粒度はどこまで必要ですか？

A) 応募・ピッチ資料に使える論理設計まで  
B) 後でモバイルファーストWebプロトタイプを実装できる程度まで  
C) AWSアーキテクチャ説明に必要なコンポーネント対応まで  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 3
AI同士の家族会議は、設計上どのように表現しますか？

A) 画面上は複数AIに見せるが、MVP実装では単一オーケストレーションで会話ターンを生成する  
B) 妻AI、夫AI、調停AIを論理コンポーネントとして分け、実装方式は後続で決める  
C) 最初から複数エージェントの協調処理として設計する  
X) Other (please describe after [Answer]: tag below)

[Answer]: B


### Question 4
データモデルの設計粒度はどこまで必要ですか？

A) 概念エンティティと関係のみ  
B) 主要フィールドとJSON形状まで。PBT一部適用にも使える粒度にする  
C) DynamoDBテーブル設計とアクセスパターンまで  
X) Other (please describe after [Answer]: tag below)

[Answer]: C


### Question 5
安全制御コンポーネントの境界はどの粒度で分けますか？

A) アプリケーションルールと Bedrock Guardrails をまとめて1つの安全制御コンポーネントとして扱う  
B) ポリシー判定、生成AIガードレール、監査ログを分けて設計する  
C) MVPでは標準粒度に留め、詳細な同意・監査・承認設計は将来拡張に送る  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 6
Free / Standard / Pro のプラン差分は、設計上どの程度コンポーネント化しますか？

A) Plan Policy コンポーネントとして分け、各機能がプランに応じた振る舞いを参照する  
B) UI表示の差分として扱い、内部ロジックはなるべく共通化する  
C) Free、Standard、Proごとに処理フローを分けてデモ上の差を強く見せる  
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 7
Proプランの疑似バイタル情報は、Application Design でどの位置付けにしますか？

A) 将来拡張として独立コンポーネントにし、MVPでは入力例と設計説明だけに留める  
B) MVPにも疑似入力UIとタイミング推奨ロジックを含める前提で設計する  
C) 応募資料上の将来構想としてのみ扱い、MVP設計からは外す  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 8
ピッチ・応募資料向けの説明機能は、Application Design の対象に含めますか？

A) 含める。審査員向けにAWS活用と事業性を説明する Presentation Component を置く  
B) 含めない。アプリケーション設計とは分け、Units Generation で資料作成単位として扱う  
C) 最小限だけ含める。ラスト画面とAWS構成説明に必要な責務だけ定義する  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## 7. 回答後の検証観点

- すべての `[Answer]:` が埋まっていること
- アーキテクチャ方針とデータモデル粒度が矛盾していないこと
- AI同士の家族会議の表現と実装現実性が矛盾していないこと
- Proプランの扱いがMVP範囲と混ざっていないこと
- 安全制御の粒度がブラックユーモアと安全配慮の両方を満たすこと
- PBT一部適用対象が純粋関数・データ変換・JSON往復に収まっていること

## 7.1 回答分析結果

| 質問 | 回答 | 判定 |
|---|---|---|
| Question 1 | A | 有効。比較資料作成後に基本サーバーレス主案、AgentCore風将来進化形として確定 |
| Question 2 | A | 有効 |
| Question 3 | B | 有効 |
| Question 4 | C | 有効 |
| Question 5 | A | 有効 |
| Question 6 | C | 有効 |
| Question 7 | A | 有効 |
| Question 8 | A | 有効 |

対応:

- `application-design-architecture-comparison.md` を作成し、アーキテクチャ候補の特徴を比較した
- `application-design-clarification-questions.md` を作成し、Question 1 の確定回答を依頼する
- Question 1 は `A` として確定したため、Application Design 成果物を生成した

## 8. Extension Compliance

| Extension | 状態 | Application Design Planningでの扱い |
|---|---|---|
| Security Baseline | 無効 | PoC/プロトタイプ扱いのためスキップ |
| Property-Based Testing | 一部有効 | PBT-02, PBT-03, PBT-07, PBT-08, PBT-09を後続Constructionへ渡せるよう、設計質問と実行手順に反映する |

## 9. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- Markdown表と質問形式は単純な構造にした
- すべての質問に `X) Other` と `[Answer]:` を含めた
