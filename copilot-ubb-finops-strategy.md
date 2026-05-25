# 記事タイトル案

## 第一候補

**GitHub Copilot Usage Based Billing時代のFinOps戦略：管理者が今すぐ整理すべきコスト、価値、ガードレール**

## ほかの候補

- **GitHub Copilot UBB移行前に管理者がやるべきFinOps実践**
- **GitHub CopilotのAI Creditsをどう管理するか：UBB時代の予算設計とCost Center戦略**
- **Copilotを止めずにコストを制御する：Usage Based Billing時代のFinOps入門**
- **GitHub CopilotのUsage Based Billingに備える：Enterprise管理者のためのAI Credit管理**
- **AI Creditsは削減ではなく配分するもの：GitHub Copilot UBB時代のFinOps設計**

---

# GitHub Copilot Usage Based Billing時代のFinOps戦略：管理者が今すぐ整理すべきコスト、価値、ガードレール

## この記事で言いたいこと

GitHub Copilot の Usage Based Billing 移行において、管理者がやるべきことは単に「Copilot の利用を制限すること」ではない。

重要なのは、AI Credits の利用状況を可視化し、コストの発生要因を理解し、価値の高い活動に適切にコストを配分できる運用モデルを作ることである。

つまり、Copilot UBB 対応は「コスト削減」ではなく、「AI 利用に対する FinOps」の問題として捉えるべきである。

GitHub Copilot Business / Enterprise は 2026年6月1日から Usage Based Billing に移行し、GitHub AI Credits を単位として Copilot の利用量が管理される。AI Credit の消費量は、利用するモデルと token 消費量によって決まる。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises?utm_source=openai))

---

## 1. まず管理者がやるべきこと：現状の支出を理解する

UBB 移行前に管理者が最初にやるべきことは、現状の Copilot 利用が AI Credits 換算でどれくらいの支出になるのかを把握することである。

GitHub は、管理者向けに Billing Preview と usage report CSV を提供している。usage report では user × model × day の粒度で利用状況を確認でき、`aic_quantity` と `aic_gross_amount` により AI Credits 換算の消費量と推定コストを確認できる。([docs.github.com](https://docs.github.com/en/enterprise-cloud%40latest/copilot/how-tos/manage-and-track-spending/prepare-for-usage-based-billing?utm_source=openai))

ここで見るべきことは、単に「総額が上がるか下がるか」ではない。

確認すべき観点は次の通り。

- 現在の使い方のまま UBB に移行した場合、コストは増えるのか
- コストが増える場合、どのユーザー、どのモデル、どのユースケースが原因なのか
- 高コスト利用は無駄なのか、それとも価値の高い業務に紐づいているのか
- 特定のユーザーやチームに利用が偏っているのか
- その偏りは業務上妥当なのか

ここで重要なのは、「高コストユーザーを探す」ことではなく、「高コスト利用の理由を理解する」ことである。

---

## 2. コストが跳ね上がる主な要因

Copilot UBB におけるコストは、主に「どのモデルを使うか」と「どれだけ token を消費するか」で決まる。GitHub Docs でも、軽量モデルを使った短い質問と、frontier model を使った長い coding agent session ではコストが異なると説明されている。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

主なコスト増加要因としては、次のものが考えられる。

### 高コストモデルの常用

Claude Opus のような高性能・高コストなモデルを常用している場合、AI Credit 消費が大きくなる可能性がある。

ただし、高コストモデルの利用自体が悪いわけではない。

複雑な設計判断、マルチファイルのリファクタリング、レガシー移行、難易度の高い障害調査などでは、高性能モデルを使うことに十分な合理性がある。

問題は、「高価なモデルを使っていること」ではなく、「タスクの価値や難易度に対してモデル選択が right-sizing されていないこと」である。

### Agent / Subagent の多段利用

Subagent を複数チェーン的に呼び出す agentic workflow では、token 消費が増えやすい。

特に、オーケストレーターとなる agent と配下の subagent がすべて高価なモデルを使っている場合、消費量が乗数的に増える可能性がある。

一方で、agentic workflow は高付加価値な業務にも向いている。たとえば、大規模なコード調査、移行計画、リファクタリング支援、テスト生成、セキュリティ負債削減などでは、agentic workflow の価値が大きい。

したがって、管理者は「agent を禁止する」のではなく、次の観点で見直すべきである。

- オーケストレーターには高性能モデルが必要か
- 配下の subagent まで同じ高性能モデルである必要があるか
- 単純なタスクは軽量モデルに delegate できないか
- agent が毎回過剰な context を読み込んでいないか

### System instruction / Custom instruction の肥大化

Copilot が毎回参照する instruction や context が膨大になると、入力 token が増えやすくなる。

特に、次のような状態は見直し対象になる。

- 全タスクで巨大なルールセットを読み込んでいる
- 特定の作業でしか必要ない docs を常時参照している
- repository / organization / agent / custom instruction が重複している
- 「とりあえず全部読ませる」設計になっている

改善の方向性は、常時投入ではなく条件付き参照にすることである。

たとえば、

> 〇〇をするときは、〇〇の docs を参照する

という形で、必要なときだけ関連情報を参照する設計にする。

これは Copilot のプロンプト設計の話であると同時に、FinOps 的には「常時リソース投入」から「必要時利用」へ変える right-sizing の話でもある。

### Tools / MCP / Web参照による context 増加

MCP や外部 tool、Web ページ参照も token 消費の要因になり得る。

ただし、「MCP サーバーを常時起動していること」自体が必ず token 消費を生む、というより、問題は tool definition、schema、取得結果、Web ページ内容などが不必要に広く context に含まれることである。

したがって、見るべきポイントは次の通り。

- MCP tool の定義が毎回過剰に渡されていないか
- 取得結果が長すぎないか
- 必要な情報だけに絞って返せているか
- Web ページや docs を丸ごと渡していないか
- Skill 化や tool 化によって、必要時に最小限の情報だけを動的取得できないか

ただし、Skill 化すれば必ず安くなるわけではない。重要なのは、動的参照、最小 context、取得結果の要約・構造化である。

---

## 3. 「高コスト利用」ではなく「高付加価値利用」を識別する

FinOps の観点では、コストを下げるだけでは不十分である。

重要なのは、価値の低い消費を抑え、価値の高い消費を止めないことである。

Well-Architected の Managing AI credits でも、重要プロジェクト、Platform Engineering、power users、agentic workflows、frontier model を使うユースケース、POC や hackathon など、価値や業務上の必要性に応じた配分設計が推奨されている。([wellarchitected.github.com](https://wellarchitected.github.com/library/governance/recommendations/managing-ai-credits/))

たとえば、次のような活動は高付加価値利用として識別すべきである。

- 重要なプロジェクト
- 売上や顧客影響の大きいプロダクト開発
- Platform Engineering Team
- 開発者体験を改善するチーム
- レガシーモダナイゼーション
- セキュリティ負債の削減
- テストカバレッジ改善
- 大規模移行プロジェクト
- AI agent を前提とした開発生産性改善
- PoC、hackathon、短期集中プロジェクト

このような活動に対しては、標準ユーザーより高い User-level budget や、専用 Cost Center を設定することが合理的である。

逆に、単純な質問、短いコード説明、軽微な修正、定型的なテスト生成などに常に高コストモデルを使っている場合は、right-sizing の対象になる。

---

## 4. AI Credits の共有プールを正しく理解する

Copilot Business では 1ユーザーあたり月 1,900 AI Credits、Copilot Enterprise では 1ユーザーあたり月 3,900 AI Credits が含まれる。これらの included AI Credits はユーザーごとの個別バケットではなく、billing entity level で共有プール化される。たとえば Copilot Business ユーザーが100人いる Enterprise では、190,000 AI Credits の共有プールになる。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

ここで重要なのは、共有プールは Cost Center ごとに事前配分されるものではない、という点である。

つまり、

```text
Enterprise: 190,000 AIC
  ├─ Cost Center A: 50,000 AIC
  ├─ Cost Center B: 80,000 AIC
  └─ Cost Center C: 60,000 AIC
```

という allocation tree ではない。

実際には、

```text
Enterprise全体のincluded AI Credits共有プール: 190,000 AIC
```

があり、対象ユーザーの利用がそこから消費される。

Cost Center は、この included AI Credits の取り分を事前に切り出すための仕組みではない。

---

## 5. Budget は allocation tree ではなく concurrent guardrails

Copilot UBB の budget を理解するうえで重要なのは、budget は「上位のお財布から下位組織に配分する allocation tree」ではなく、「複数レイヤーで同時に適用される guardrails」だということである。

### allocation tree 的な誤解

誤解しやすいモデルは次のようなもの。

```text
Enterprise予算
  ├─ Cost Center予算
  │   ├─ User A予算
  │   └─ User B予算
  └─ Cost Center予算
      ├─ User C予算
      └─ User D予算
```

このモデルでは、Enterprise の予算を Cost Center に切り出し、さらに User に切り出すように見える。

しかし、Copilot UBB の budget はこの考え方ではない。

### concurrent guardrails としての理解

実際には、次のような複数のガードレールが同時に適用される。

```text
Included AI Credits共有プール
+
Enterprise-level budget
+
Cost Center budget
+
Organization-level budget
+
User-level budget
```

あるユーザーが Copilot を使うとき、該当する budget layer が同時にチェックされる。

たとえば、

- 共有プールに AI Credits が残っているか
- 共有プール枯渇後の additional usage が許可されているか
- Enterprise-level additional spend budget に余裕があるか
- Cost Center に所属している場合、その Cost Center budget に余裕があるか
- User-level budget に達していないか

といった判定が重なる。

GitHub Docs では、Enterprise、Organization、Cost Center、User の4レベルで AI Credits spend を制御する budget を設定できると説明されている。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

### 一文でいうと

> Copilot UBB の budget は、単一のお財布から下位組織へ予算を配分するモデルではなく、included AI Credits の共有プールと追加支出用 budget に対して、Enterprise / Cost Center / User などのガードレールが同時に適用されるモデルである。

---

## 6. Enterprise-level budget の考え方

Enterprise-level budget は、Enterprise 全体の追加支出リスクを抑えるためのガードレールである。

ここで注意すべきなのは、Enterprise-level budget を「厳格な停止装置」として使いすぎると、開発生産性への影響が大きくなる点である。

共有プールが枯渇し、かつ additional usage が許可されていない場合、利用は次の billing cycle までブロックされる。User-level budget を使い切った場合も、そのユーザーの Copilot access は停止される。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

Code completions と next edit suggestions は AI Credits 課金対象外だが、Copilot Chat、Copilot CLI、Copilot cloud agent、Copilot Spaces、Spark、third-party coding agents などは AI Credits を消費する。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

そのため、Enterprise-level で hard stop を強くかけすぎると、コード補完以外の多くの Copilot 機能が利用できなくなり、業務影響が大きくなる可能性がある。

Enterprise-level budget は、まずは次の目的で使うのが妥当である。

- 追加支出の上限を定義する
- 予算閾値に応じて alert を出す
- Finance / Platform / Engineering leadership にエスカレーションする
- 想定外の支出増を検知する

---

## 7. User-level budget の考え方

User-level budget は、ユーザー単位の過剰消費を防ぐためのガードレールである。

特に UBB 移行後は、全ユーザーに Universal User-level budget を設定し、そのうえで必要に応じて power users に custom override を設定する設計が現実的である。

User-level budget の目的は、典型的な利用を過度に制限することではない。

目的は次の2つである。

- 共有プールを一部ユーザーが過剰に消費することを防ぐ
- 共有プール枯渇後の additional spend を個人単位で制御する

したがって、最初から低すぎる ULB を設定するのは避けるべきである。

まずは現状の usage report を確認し、通常利用の大半を妨げない水準に Universal ULB を設定する。そのうえで、Platform Engineering、重要プロジェクト、agentic workflow を多用するチームなどには、業務上の正当性に基づいて高めの override を設定する。

ここで重要なのは、「ユーザーに自己責任でクレジット管理させる」というより、「管理者が透明性、教育、適切なデフォルト、例外申請プロセスを用意する」ことである。

---

## 8. Cost Center の意義：請求分離からFinOps境界へ

従来、Cost Center は主に「部署ごとに請求先を分けたい」場合に利用されることが多かった。

たとえば、

- 会社全体で GitHub Enterprise を使う
- 部署ごとに Organization を分ける
- GitHub Enterprise に紐づく Azure Subscription とは別に、部署ごとの Azure Subscription に請求したい
- 部署ごとの予算や会計責任に合わせて請求先を明示的に分けたい

というユースケースである。

しかし、Copilot UBB 移行後は Cost Center の意味が広がる。

Cost Center は、included AI Credits の取り分を配分するための仕組みではない。Cost Center は、共有プール枯渇後の additional spend を、どの責任単位で、どこまで許容するかを管理するための FinOps 境界になる。

GitHub Docs では、Cost Center は利用量と支出を business unit に帰属させ、accountability、forecasting、cost allocation を改善するための仕組みであり、budget も適用できると説明されている。また、Copilot のような license-based product では、Cost Center の課金は Cost Center に所属する user に基づいて行われる。([docs.github.com](https://docs.github.com/en/billing/concepts/cost-centers))

---

## 9. なぜ Org だけでは不十分なのか

GitHub Enterprise の管理階層は、おおまかに次のようになる。

```text
Enterprise
  └─ Organization
      └─ Repository
```

この階層は、アクセス管理、リポジトリ管理、ポリシー管理、ライセンス割り当てには適している。

しかし、FinOps の責任境界としては必ずしも十分ではない。

理由は、Org が必ずしも財務上の責任単位と一致しないからである。

たとえば、

- 1つの部署が複数 Org を使っている
- 1つの Org に複数部署の repository が混在している
- Copilot のライセンス付与元 Org と実際の業務所属が一致しない
- Platform Engineering のように横断組織が複数 Org を支援している
- 重要プロジェクトが一時的に複数 Org / repository を横断している

このような場合、Org 単位だけでは「誰が、何のために、どれだけ AI Credits を消費したのか」を説明しづらい。

Cost Center を使うことで、GitHub 上の管理境界とは別に、business unit、team、project、role に応じたコスト責任境界を設計できる。

---

## 10. Cost Center usage と Enterprise budget の関係

ここは誤解しやすい。

Cost Center に budget があるからといって、included AI Credits が Cost Center に個別配分されるわけではない。

たとえば Copilot Business ユーザーが100人いる Enterprise では、included AI Credits は 190,000 AIC の共有プールになる。この 190,000 AIC が Cost Center ごとに切り分けられるわけではない。([docs.github.com](https://docs.github.com/en/copilot/concepts/billing/usage-based-billing-for-organizations-and-enterprises))

Cost Center budget が意味を持つのは、主に共有プール枯渇後の additional spend に対してである。

整理すると次の通り。

```text
Included AI Credits共有プールが残っている間:
  - Cost Center所属ユーザーも非所属ユーザーも共有プールを消費する
  - Cost CenterにAICの取り分があるわけではない

Included AI Credits共有プールが枯渇した後:
  - additional usageを許可している場合、追加課金として利用継続できる
  - Cost Center budgetは、そのCost Centerがどこまで追加支出してよいかを制御する
```

つまり、

> Cost Center budget は、Enterprise の included AIC の取り分ではなく、共有プールを使い切った後に、その Cost Center がどこまで追加課金してよいかを決めるガードレールである。

---

## 11. Azure Subscription 設計の3パターン

Cost Center を使う場合、Azure Subscription との紐づけ方も設計ポイントになる。

GitHub Docs では、Azure で支払う場合、Cost Center に Azure Subscription を追加して、Enterprise default とは異なる Azure Subscription に usage を請求できると説明されている。([docs.github.com](https://docs.github.com/en/billing/concepts/cost-centers))

また、1つの Azure Subscription に接続できる Enterprise や Cost Center の数に制限はなく、複数の Azure Subscription を1つの Enterprise account で使いたい場合は Cost Center を作成し、各 Cost Center を異なる Azure Subscription に接続できる。Cost Center が Azure Subscription に接続されていない場合、その usage は Enterprise account の Azure Subscription に請求される。([docs.github.com](https://docs.github.com/en/enterprise-server%403.20/billing/reference/azure-subscription?utm_source=openai))

### パターン1：Cost CenterごとにAzure Subscriptionを分ける

```text
Enterprise default Azure Subscription
Cost Center A -> Azure Subscription A
Cost Center B -> Azure Subscription B
Cost Center C -> Azure Subscription C
```

最も分離度が高いモデル。

向いているケース。

- 部署ごとに Azure 予算を持っている
- 部門別 chargeback を厳密に行いたい
- 子会社や事業部ごとに請求責任を分けたい
- Azure Cost Management 側でも明確に分離したい

メリット。

- 請求責任が明確
- 部署ごとの Azure 予算と連携しやすい
- chargeback に向いている
- 部署ごとの自律性が高い

デメリット。

- Azure Subscription の数が増える
- 管理負荷が高い
- 権限管理、棚卸し、異動時のメンテナンスが複雑
- Cost Center 設計と Azure 設計の両方が必要

### パターン2：複数のCost CenterでAzure Subscriptionを共有する

```text
Cost Center A -> Engineering Shared Subscription
Cost Center B -> Engineering Shared Subscription
Cost Center C -> Engineering Shared Subscription
```

中間モデル。

向いているケース。

- GitHub 側では Cost Center ごとに利用状況や budget を分けたい
- ただし Azure Subscription を大量に増やしたくない
- 請求は大きな部門単位でまとめたい
- showback を中心に運用したい

メリット。

- GitHub 側では Cost Center ごとに管理できる
- Azure 側の Subscription 乱立を防げる
- 管理負荷と自律性のバランスがよい

デメリット。

- Azure invoice 上では同じ Subscription に複数 Cost Center の費用が載る
- Cost Center ID と社内部署・プロジェクトの対応表が必要
- 厳密な chargeback には追加の運用が必要

### パターン3：Enterprise defaultのAzure Subscriptionに集約する

```text
Enterprise default Azure Subscription
  ├─ Cost Center A
  ├─ Cost Center B
  └─ Cost Center C
```

最もシンプルなモデル。

向いているケース。

- まずは UBB 移行に備えて可視化と budget control を始めたい
- Azure Subscription を分けるほどの会計要件がない
- 中央の Platform / Finance チームが一括管理する
- showback で十分

メリット。

- 構成がシンプル
- 導入しやすい
- Azure 管理の負荷が低い
- Cost Center による利用状況把握と budget 管理は可能

デメリット。

- 請求先の分離は弱い
- 部署ごとの自律性は限定的
- 中央管理者の負担が残る

---

## 12. おすすめの進め方

多くの企業にとって、最初から Cost Center ごとに Azure Subscription を分けるのは重すぎる可能性がある。

UBB 移行の初期段階では、次の順番が現実的である。

### Step 1: Usage reportで現状把握

- user × model × day の消費を見る
- 高消費ユーザーを特定する
- 高消費の原因を model / agent / context / tool の観点で分解する
- 重要プロジェクトや Platform Engineering など、高付加価値利用を識別する

### Step 2: Enterprise-level budgetとalertを設定

- Enterprise 全体の追加支出上限を定義する
- いきなり hard stop にしすぎない
- 閾値 alert を設定する
- Finance / Engineering leadership への通知経路を決める

### Step 3: Universal User-level budgetを設定

- 全ユーザーに標準の ULB を設定する
- 低すぎる設定は避ける
- 通常利用を妨げない水準にする
- Power user / 重要プロジェクトには override を検討する

### Step 4: Cost Centerを設計する

Cost Center は、単に Org 構造に合わせるのではなく、次の観点で設計する。

- 財務責任単位
- business unit
- Platform Engineering
- 重要プロジェクト
- AI agent を多用するチーム
- POC / hackathon / migration factory
- 追加支出の意思決定を委任したい単位

### Step 5: Azure Subscriptionの分離度を決める

- まずは Enterprise default に集約する
- 必要に応じて複数 Cost Center で Azure Subscription を共有する
- 厳密な chargeback が必要な場合のみ、Cost Center ごとに Azure Subscription を分ける

---

## 13. 組織運用として必要なこと

継続的なコスト最適化には、継続的に利用状況を監査できる状態が必要である。

一度 budget を設定して終わりではない。

必要な運用は次の通り。

- Billing Manager や FinOps owner を明確にする
- Cost Center ごとの cost owner を決める
- usage report を定期的に確認する
- 高消費ユーザーと高付加価値利用を区別する
- budget threshold alert への対応手順を決める
- Power user override の申請・承認プロセスを作る
- モデル選択ガイドラインを作る
- custom instruction / MCP / tool 利用の棚卸しを行う
- 月次または隔週で利用状況をレビューする
- UBB 移行直後はより短いサイクルで確認する

特に重要なのは、コスト最適化の責任を Enterprise 管理者だけに集中させないことである。

どの利用が高付加価値か、どこまで追加支出を許容すべきかは、各部署やプロジェクトの文脈に依存する。

したがって、Cost Center を使って責任境界を作り、各部署やチームの owner に一定の判断を委任することが、Copilot UBB 時代の現実的な FinOps 運用になる。

---

## 14. 避けるべき誤解

### 誤解1：高コストモデルは使うべきではない

正しくは、高コストモデルは高付加価値タスクに使うべきである。

禁止ではなく right-sizing が重要。

### 誤解2：Cost Centerはincluded AICを分配する仕組みである

正しくは、included AI Credits は billing entity level の共有プールであり、Cost Center ごとに事前配分されるわけではない。

Cost Center budget は、主に共有プール枯渇後の additional spend を責任単位ごとに管理するためのもの。

### 誤解3：Org ownerがいればCost Centerは不要

Org は GitHub 上の管理境界であり、必ずしも財務責任境界ではない。

FinOps 的には、business unit、team、project、role に応じた Cost Center 設計が必要になる場合がある。

### 誤解4：Budgetは親から子へ配分するツリーである

正しくは、budget は concurrent guardrails である。

Enterprise、Cost Center、User-level budget などが同時に適用される。

### 誤解5：コスト削減がFinOpsである

FinOps の目的は単なる削減ではない。

価値の低い消費を抑え、価値の高い利用にコストを再配分することが重要である。

---

## まとめ

GitHub Copilot の Usage Based Billing 移行において、管理者がやるべきことは、Copilot の利用を単純に制限することではない。

やるべきことは次の5つである。

1. Billing Preview と usage report で現状の利用と支出を把握する
2. 高コスト利用の原因を model / agent / context / tool の観点で分解する
3. 重要プロジェクトや Platform Engineering など、高付加価値利用を識別する
4. Enterprise / Cost Center / User-level budget を多層ガードレールとして設計する
5. Cost Center を使って、追加支出の責任境界と意思決定を分散する

Copilot UBB 時代の FinOps は、「AI Credits を使わせない」ための統制ではない。

むしろ、AI Credits を価値の高い活動に配分し、無駄な消費を抑えつつ、開発生産性を最大化するための運用設計である。
