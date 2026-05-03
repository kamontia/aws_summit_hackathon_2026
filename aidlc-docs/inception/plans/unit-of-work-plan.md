# Unit of Work Plan: 夫婦間API Gateway

## 1. 目的

Units Generation では、夫婦間API Gateway を後続の実装・資料作成に使える作業単位へ分解する。

このプロジェクトは現時点では応募・ピッチ資料とInception成果物が主目的であり、Construction Phase は延期されている。  
ただし、後で実装へ進めるよう、UI、AI処理、安全制御、データ、ピッチ説明を明確なUnit of Workとして整理する。

## 2. 前提

- [x] Requirements Analysis を確認した
- [x] User Stories を確認した
- [x] Application Design を確認した
- [x] Greenfieldであり、既存アプリケーションコードが存在しないことを確認した
- [x] 主アーキテクチャが基本サーバーレス構成であることを確認した
- [x] AgentCore風構成は将来進化形として扱うことを確認した
- [x] Security Baseline は無効であることを確認した
- [x] Property-Based Testing は一部適用であることを確認した

## 3. 分解方針の初期案

初期案では、以下の5つのUnit of Workに分ける。

| Unit候補 | 目的 | 主な対象 |
|---|---|---|
| U-01 Demo Experience UI | モバイルファーストのデモ体験とラスト演出 | US-01, US-02, US-05, US-08 |
| U-02 Message Rewrite Core | 入力、地雷ワード検出、リスク判定、言い換え生成 | US-02, US-03, US-04 |
| U-03 Safety and Plan Control | 安全制御とFree / Standard / Proの処理差分 | US-05, US-06, FS-01 |
| U-04 AI Family Meeting | 妻AI、夫AI、調停AIの論理会議と合意案 | US-07, US-08 |
| U-05 Data and Presentation Architecture | DynamoDB設計、AWS説明、応募・ピッチ資料の説明材料 | US-01, US-09, FS-01 |

この初期案は、ユーザー回答により調整する。

## 4. 実行手順

- [x] 1. Units Generation のルールを読み込む
- [x] 2. Requirements, User Stories, Application Design を読み込む
- [x] 3. 初期Unit候補を作成する
- [x] 4. `unit-of-work-plan.md` を作成する
- [x] 5. ユーザーが分解方針質問の `[Answer]:` をすべて記入する
- [x] 6. 回答を検証し、未回答・曖昧回答・矛盾がないか確認する
- [x] 7. 曖昧さがある場合、追加確認質問を作成する
- [x] 8. Unit of Work Plan の承認を得る
- [x] 9. 承認済み方針に基づいて `unit-of-work.md` を生成する
- [x] 10. 承認済み方針に基づいて `unit-of-work-dependency.md` を生成する
- [x] 11. 承認済み方針に基づいて `unit-of-work-story-map.md` を生成する
- [x] 12. Greenfieldのコード組織戦略を `unit-of-work.md` に記載する
- [x] 13. すべてのストーリーがUnitに割り当てられていることを確認する
- [x] 14. Unit境界と依存関係を検証する
- [x] 15. Extension Compliance と Content Validation を記録する
- [x] 16. Units Generation 完了レビューを提示する

## 5. 生成予定成果物

| 成果物 | 目的 |
|---|---|
| `aidlc-docs/inception/application-design/unit-of-work.md` | Unit定義、責務、所有コンポーネント、Greenfieldコード組織戦略 |
| `aidlc-docs/inception/application-design/unit-of-work-dependency.md` | Unit間依存、実装順、共有データ、連携点 |
| `aidlc-docs/inception/application-design/unit-of-work-story-map.md` | User StoryとUnitの対応、MVP/将来拡張の境界 |

## 6. 分解方針質問

以下の質問に回答してください。各質問の `[Answer]:` の後ろに、選択肢の文字を記入してください。  
どれにも合わない場合は最後の `X) Other` を選び、同じ行または次の行に希望内容を書いてください。

### Question 1
Unit of Work の分解粒度はどれがよいですか？

A) ピッチ・応募資料と後続実装の両方に使いやすい中粒度。UI、AI処理、安全/プラン、AI家族会議、データ/資料で分ける  
B) 実装しやすさ重視の細粒度。各サービスやコンポーネントをなるべく別Unitに分ける  
C) 資料作成重視の粗粒度。アプリ本体、AWS構成、ピッチ資料の3Unit程度にまとめる  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 2
Story Grouping はどの観点を優先しますか？

A) ユーザージャーニー順。ログイン、入力、言い換え、家族会議、ラスト演出の流れを優先する  
B) 技術境界順。UI、バックエンド、AI、安全制御、データを優先する  
C) ピッチ訴求順。家庭内あるある、事業性、ブラックユーモア、AWS活用を優先する  
X) Other (please describe after [Answer]: tag below)

[Answer]: C

### Question 3
Free / Standard / Pro のプラン差分は、Unitとしてどのように扱いますか？

A) Safety and Plan Control Unitにまとめ、他UnitはPlan Flowを参照する  
B) 各Unitにプラン差分を分散させ、UI、生成、会議のそれぞれで差を出す  
C) Proは将来拡張Unitとして独立させ、Free / StandardだけをMVP Unitに含める  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 4
AI同士の家族会議は、Unitとしてどのように扱いますか？

A) 独立Unitにする。妻AI、夫AI、調停AIの論理役割とラスト演出をまとめる  
B) Message Rewrite Coreに含める。言い換え生成の延長として扱う  
C) Presentation Unitに含める。ピッチ上の演出として扱い、実装単位としては分けない  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 5
データストアとDynamoDB設計は、どのUnitが所有しますか？

A) Data and Presentation Architecture Unitが所有し、他Unitはデータ契約に依存する  
B) 各Unitが自分のデータを所有し、DynamoDB単一テーブル内で論理分割する  
C) Message Rewrite Coreが中心データを所有し、他Unitは必要最小限だけ参照する  
X) Other (please describe after [Answer]: tag below)

[Answer]: A


### Question 6
Team Alignment と所有境界は、どの想定で分けますか？

A) 1人または少人数ハッカソン向け。Unitは実装順と資料整理のための論理分解にする  
B) 複数人チーム向け。UI担当、バックエンド担当、AI担当、資料担当で並行作業しやすくする  
C) 将来プロダクト化向け。サービス所有境界と運用責務まで意識して分ける  
X) Other (please describe after [Answer]: tag below)

[Answer]: A

### Question 7
Greenfieldのコード組織戦略はどれを前提にしますか？

A) Greenfield multi-unit monolith。`src/{unit-name}/` と `tests/{unit-name}/` に分ける  
B) Greenfield single unit。まず `src/`, `tests/`, `config/` の単一アプリとして作る  
C) Greenfield multi-unit microservices。`{unit-name}/src/`, `{unit-name}/tests/` に分ける  
D) 現時点では実装しないため、コード組織戦略は候補としてのみ記載する  
X) Other (please describe after [Answer]: tag below)

[Answer]:  B


### Question 8
Units Generation後の扱いはどれがよいですか？

A) Inception Phase完了まで進める。Units生成後、Constructionには進まず一度止める  
B) Units生成後にConstruction Phaseの計画まで進める  
C) Units生成後、応募用企画書やピッチ資料への反映を優先する  
X) Other (please describe after [Answer]: tag below)

[Answer]: A


## 7. 回答後の検証観点

- すべての `[Answer]:` が埋まっていること
- Unit粒度とコード組織戦略が矛盾していないこと
- AI家族会議の扱いがApplication Designと矛盾していないこと
- ProプランがMVPと将来拡張のどちらに置かれるか明確であること
- すべてのUser Storyが少なくとも1つのUnitに割り当てられること
- PBT一部適用対象が後続Constructionへ渡せること

## 7.1 回答分析結果

| 質問 | 回答 | 判定 |
|---|---|---|
| Question 1 | A | 有効。中粒度Unitで分解する |
| Question 2 | C | 有効。ピッチ訴求順を優先してStoryを配置する |
| Question 3 | A | 有効。Safety and Plan Control Unitがプラン差分を統括する |
| Question 4 | A | 有効。AI Family Meetingは独立Unitにする |
| Question 5 | A | 有効。Data and Presentation Architecture UnitがDynamoDB設計を所有する |
| Question 6 | A | 有効。少人数ハッカソン向けの論理分解とする |
| Question 7 | B | 有効。コードは単一アプリ構成を前提にし、Unitは実装・資料整理上の論理単位として扱う |
| Question 8 | A | 有効。Units生成後はConstructionへ進まず、Inception Phase完了として一度止める |

### 整合性判断

- Q1は中粒度Unit分解、Q7は単一アプリ構成であり、矛盾ではない。Unit of Workは「開発・資料整理の論理単位」とし、実装に進む場合のコードは単一アプリの `src/`, `tests/`, `config/` 配下で整理する。
- Q2のピッチ訴求順は、Unitの表示順とStory Mapの説明順に反映する。
- Q3とQ7により、Proは将来拡張としてSafety and Plan Control Unitで扱い、実コードは単一アプリ内の論理モジュール候補として記載する。
- 追加確認質問は不要である。

## 7.2 確定した生成方針

| 項目 | 方針 |
|---|---|
| Unit粒度 | 中粒度 |
| Unit数 | 5 Unit |
| Story配置の優先 | ピッチ訴求順 |
| プラン差分 | Safety and Plan Control Unitが統括 |
| AI家族会議 | 独立Unit |
| DynamoDB設計 | Data and Presentation Architecture Unitが所有 |
| 所有境界 | 少人数ハッカソン向けの論理分解 |
| コード組織 | Greenfield single unit。`src/`, `tests/`, `config/` を候補として記載 |
| Units生成後 | Constructionには進まず、Inception Phase完了として一度停止 |

## 8. Extension Compliance

| Extension | 状態 | Units Generation Planningでの扱い |
|---|---|---|
| Security Baseline | 無効 | PoC/プロトタイプ扱いのためスキップ |
| Property-Based Testing | 一部有効 | PBT対象候補をUnit境界へ紐づける。Planning時点では実装コードがないためブロッキング不適合なし |

## 9. Content Validation

- Mermaid図は使用していない
- ASCII図は使用していない
- Markdown表と質問形式のみで構成した
- すべての質問に `X) Other` と `[Answer]:` を含めた
