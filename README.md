# 6/22 MCD Exam Readiness ワークショップ
###### tags: `MuleSoftTraining`

- [MuleSoft Exam Readiness Kit - 認定資格試験への準備キット -](https://salesforce.quip.com/JBhRAIbXU9VO)
  - 試験の予約方法や、当日の流れなど、試験準備に関して必要な情報はこちらにて！ 
  - [MCD データシート](https://training.mulesoft.com/oltpublish/cmsres/downloads/MCD_level1_datasheet_japanese.pdf) で出題範囲を確認
  - [MCD 模擬試験](https://training.mulesoft.com/course/development-fundamentals-mule4/quiz-diy-japanese) で実際の試験時間・問題数・難易度に近い問題を解いてみる
    - 少し古い試験範囲から、模擬試験に出題されている場合もあります


# 1. アプリケーションネットワークの構築 (開発:基礎モジュール1, モジュール２)

## Q1: 実装とインターフェース
インターフェースに変更がない=API クライアントには変更が必要ない
※ユニットテストがあることによって、この実装の変更が起こしうる、APIクライアントへの影響をテストできる
![Q1](https://i.imgur.com/cnT0LvW.png)

ある日突然「コンセント」のインターフェースが変わったら...
![malaysia](https://i.imgur.com/iqcC9W0.png)

## :star: API-led Connectivity (API主導の接続性)
- Experience API(EAPI)
  - API クライアント毎の違いを吸収する(データ型、フォーマット, セキュリティ..) 
- Process API(PAPI) 
  - ビジネスロジック、連携、統合、オーケストレーション
- System API(SAPI)
  - バックエンドのデータを取得・更新・作成
  - 統一のデータ型への変換を行うことも(SAP Customer <-> SFDC Customer)


## Spec-Driven Development (仕様駆動型開発)
- どちらのツール、どちらの言語でもOK！
   - RAML 0.8/1.0 > *今回の試験では**RAML**が出題!*
   - OAS 2.0(a.k.a Swagger)/3.0 
   - (AsyncAPI)
- なぜ？
  - その API の役割や機能について合意をとる
  - フィードバックをもらって改善する
  - データ型、エンドポイント名、フィールド名、パラメータ (必須 or not)、などについての事前確認 
  - 仕様通りのバリデーションを自動で適用
  - Studio にインポートして、土台となるスケルトン (インタフェースフロー)を自動で生成する
  - Exchange に公開したタイミングで
    - API ポータルが自動生成される
    - 仕様をもとにドキュメントが自動生成されます
    - API コンソールが自動生成される
  	- コネクタの自動生成 (REST Connect)
    - モッキングサービスが自動生成される (プロトタイプ)
      - [シミュレーションヘッダー](https://docs.mulesoft.com/jp/design-center/apid-behavioral-headers)機能で、より実際に近いモック呼び出しが可能(MS2-Delay etc..)
    ![](https://i.imgur.com/85GOSWH.png)

## C4E (Center for Enablement)
Anypoint Platform をより活用し、アセット(資産)の **再利用** を促進するための機能横断チーム・組織づくり

# 2. API の設計 (開発:基礎モジュール3)
## 「XXX の適切な呼び出し方は...?」系の問題では以下の3つを確認する
- 1. 何をしたいのか (HTTPメソッド、 CRUD)
  - **GET** (記事のデータを*取得*する)
  - PUT(完全に置き換える・更新する)
  - PATCH(一部を置き換える・更新する)
  - POST
  - DELETE
- 2. フォーマット(形式)
  - **XML**, JSON..
- 3. 必須 (required) のパラメータ
  - クエリパラメータ
  - URI パラメータ
  - ヘッダー
※RAML では "?" や `required: false` に設定しなければ、パラメータはデフォルトで必須となる

## REST API を設計するための言語 (ツール)
- **RAML** 0.8 / **1.0** < MCD ではこちらが重要!
   - https://raml.org/
   - https://github.com/raml-org/raml-spec/blob/master/versions/raml-10/raml-10.md/
- OAS 3.0 / 2.0 (Swagger)
- (EDA であれば AsyncAPI もサポートを開始)

## REST API > HTTP メソッド
GET, POST, PUT, DELETE, PATCH...

## クエリパラメータ vs URI パラメータ
### クエリパラメータ (?/&)
フィルタリングをするために使用する
 - GET:/customers **?country=JP**
   - 日本のお客様情報を取得 
- GET:/palyers **?position1=pitcher**
   - ピッチャーの選手情報を取得 
 - GET:/palyers **?position1=pitcher&position2=outfielder**
   - ピッチャー兼外野手のどちらにも当てはまる、選手情報を取得 
     - Ohtani san!
※2つ以上のパラメータを使用する場合には ? と & を使用する

### URI パラメータ ({xxx})　// 波括弧部分は動的に変更されることを示しています
 - GET:/customers/1234
   - /customers/**{custId}**
   - 1234の固有のIDを持った、お客様情報を取得 ("1"人)
 - GET: http://api.soccer.restapi.org/leagues/{leagueId}/teams/{teamId}/players/{playerId}
   - /leagues/**EU02**/teams/**10**/players/**651**

### RAML
YAML がベース = インデントが命！ (Q6)
```
/customers:
    get:
        queryParameters:
```

エラー
```
/customers:
get: # get メソッドのインデントに問題あり
    queryParameters:
```

## RAML フラグメント(部品)
データ型(data type), データ例(data example)..

## フラグメントの参照方法
`!include` (Q7)

# 3. Mule Eventの参照と変更 (開発:基礎 モジュール6)
## Mule Event
![Mule Event](https://i.imgur.com/AwLpK95.png)

- Mule Event (Mule イベント)
  - *Mule Message (Mule メッセージ)*
    - **Attributes** (属性: データの付随情報 (クエリパラメータ、ヘッダー、URI パラメータ))
      - GET:/customers?id=48493
        - `#[attributes.queryParams.id]  `>>> `48493`
        - `#[attributes.headers.method]` >>> `GET`
      - GET:/customers/48493
        - `#[attributes.uriParams.id] ` >>> `48493`
        - `#[attributes.headers.method]` >>> `GET` 
      
    - **Payload** (ペイロード: データの本体)
      - `#[payload]` {"year": 2022}
      - `#[payload.year]` >>> 2022
      
  - **Variables** (変数)
    - `#[vars.hogeFlag]` >>> `true`
    - `#[vars.tranId]` >>> `41674810`
    - `#[vars.theResult]`

## Q10 "The year is 2020" 
- payload
```
{
　　"year":"2020" // payload.year => "2020"
}
```
- `#["The year is 2020"]` // ハードコード
- "The year is 2020" // ハードコードなしで、これをログ出力したい

`#[ ]`// DataWeaveでの記述である合図

1. `#["The year is" ++ payload.year]`// ++ concatination 結合の関数
2. `The year is #[payload.year]`

## Target Variable (Q13) 
![](https://i.imgur.com/37g44F6.png)
![](https://i.imgur.com/QWAGGMS.png)

### Target Variable (ターゲット変数)
- いま持っている payload を上書きせずに、外部のシステムからデータを取得し、Mule Event の Variables に保存するための設定
- 変数名を指定して、payload にデータを入れる (=上書きする) 代わりに、取得したデータを変数にセットします
- 各コネクタの Advanced タブにて設定

## typeOf（データの型をチェック） Q14
`typeOf("The year is 2020")`// `String`
`typeOf(1234)` // `Number`
`typeOf(true)` // `Boolean`
`typeOf(null)` // `Null`
`typeOf({"hoge":"2021"})` // `Object`
`typeOf([1,2,3])` // `Array`

```
%dw 2.0
output application/xml // フォーマットを指定
---
payload
```

## DataWeave Playground(ブラウザベースのテスト環境)
https://developer.mulesoft.com/learn/dataweave/

## payload を上書きしないための2つの方法(Q16)
1. target variable を使用する (新しいデータで、今ある payload を上書きしない)
2. 変数 (vars) は上書きされないので、上書きされたくないデータは、payload ではなく vars に入れておく

- Event 
  - Message
    - Attributes
      - `#[queryParams.color]` >>> `red`
      - `#[payload]` >>> `"Order01"`
  - Vars: 
     - `#[vars.quantity]` >>> `1`

# 4. Mule アプリケーションの構造化 (開発:基礎 モジュール7)
## コネクタ設定 > グローバル要素(global element) 
1度設定したコネクタの設定は、別の xml や別フローで再利用することが可能
 
## 同一アプリケーションの別フローを HTTP Request で呼び出すと.. (Q18)
HTTP はクライアントとサーバーのやりとりのためのプロトコルであり、
サーバー側にそのままの Mule Event を渡すことができない 
=> 新たな 別 Mule Event が、呼び出された側のフローで生成されてしまう
- パフォーマンスの問題？？？ (ネットワークのホップ, TCPハンドシェイク etc...)

## Flow Reference (Q19)
 - 同一 Mule アプリケーションの別フローを呼び出す
 - payload, attribtues, variables など、フローを跨いで共有可能

## yaml/propertiesの設定を動的に参照する(Q20)
プロパティのプレースホルダーで、コネクタ設定のハードコードを回避
`${training.db.host}`// 動的にyamlの値を取りに行く

config.yaml
```
traininig: 
  db:
    host: "here.is.what.is.taken.from.the.expression"
```

## 参照するyamlの設定(Q21)
Global Element >　Configuration properties

![q21](https://i.imgur.com/MrM3ear.jpg)

`${env}-config.yaml`// 参照するyamlファイルを、${env}環境変数でコントロールすることも可能

- DataWeave からは `p('db.port')`の形式で参照可能
 
# 5. API 実装のためのインターフェース
## API Kit
- RAML仕様から、土台となるインターフェースフローを自動生成してくれるツール
- APIKit を使用して、インターフェースフローを自動生成する (scaffolding = 土台作り)
- API 仕様に基づいたバリデーションも自動生成
  - 必須のパラメータがなかったら _400 Bad Request_ 

## APIKit 仕様から API のインターフェースを自動生成する(sccafold)

方法1. プロジェクト名を右クリック > Manage APIs > 管理する API を新規追加

方法2. 右クリック > New > Mule Project > Exchange, Design Center, もしくはローカルファイルから API 仕様を読み込ませる 
※ Anyoint Platform アカウントへのログインが必要

# 6. コネクタの活用
外部のシステムとの連携をシンプルに、スピーディーにするためのモジュール
- HTTP, Slack, SAP, Salesforce, ServiceNow, Google Drive, Kafka, Database...

## REST Connect コネクタ
 - RAML/OAS から自動生成されるコネクタ
  - Exchange に API 仕様を公開したタイミングで自動生成される
- REST API 呼び出しの複雑さをラッピングする
   - 必須のパラメータ
   - クエリパラメータ
   - オペレーションとして ドラッグ&ドロップが可能になる (vs 手動で HTTP リクエストを設定)
     - HTTP メソッド
     - パス
```
/flights:
  get:
    displayName: getFlights(フライト情報を取得)
    description: アメリカン航空のフライト情報を JSON <Array.Object>で取得 
```
※ RAML の displayName を変更すれば、日本語のオペレーション名にすることも可能

     
## Q27 WHERE の条件に当てはまらない場合の return value
`SELECT * FROM aTable 
WHERE ID = 100;` >>> `[]` //空の配列

## Q28 Database のオペレーションで、Mule Event の値を使用する
`:(コロン)` を使用して、Input Parameters セクションとマッピングする

`INSERT INTO orders.ORDER(orderId, cutomerName, status, startDate) 
VALUES (:oid, :custId, :status, now())
`// now() は DBの関数

Input Parameters(Mule Event を使用可能)
```
#[{
  oid: payload.oid,
  custId: payload.custId,
  status: payload.status
}]
```

![Input Parameters](https://i.imgur.com/oFsk79l.png)


## Q29 ++ 関数を Object に使用する
https://developer.mulesoft.com/learn/dataweave/
```
%dw 2.0
output application/json
---
{
    tran_id: 1234,
    account_id: 999,
    name: "Max",
    position: "sell" 
} 
++ {date: (now() >> "JST" as TimeZone)}
// ++ オブジェクトにフィールドを追加。*++* は、その他文字列の結合、配列の結合にも使うことができる　
```

```
{
  tran_id: 1234,
  account_id: 999,
  name: "Max",
  position: "sell",
  date: |2022-02-21T04:45:44.946157Z|
}
```

## Q30 JMS, VM コネクタの非同期処理
- Publish 
  - キューにメッセージを追加 (非同期)
  - Fire & Forget 
- Publish Consume
  - キューにメッセージを追加し、処理の完了まで待つ (同期)

### Q31 
1. HTTP Listner 
2. DB Select // 結果はpayloadに保存
3. Set Variable // `#[vars.dbResult] = payload`
4. HTTP Request // 結果はpayloadに保存

DBのデータ: `#[vars.dbResult]`
HTTP Requestのデータ: `#[payload]`

# 7.レコードの処理
- For Each 
  - シングルスレッド
  - 同期処理
  - sequential (順番に) 処理を行う
  - batch block size (デフォルト 1)
    - [1,2,3,4,5] >>> 1 -> 2 -> 3 -> 4 -> 5
  - batch block size (2)
    - [1,2,3,4,5] >>> 1,2 -> 3,4 -> 5
  - For Each スコープの中で、payload を書き換えても (e.g. Set Payload)、**For Each スコープを終えれば、payload はスコープに入る前の元々のデータを保持し続ける**
  - 変数は、全てのループで共有される 
    - For Each スコープの前で宣言された変数も、For Each スコープ内で参照・変更・削除可能

![](https://i.imgur.com/ZNzUPNO.png)

- Batch Job
  - マルチスレッド
  - 非同期処理
  - parallell(並列に)処理を行う
  - 変数は、各レコードに固有のもの
    - レコード1 `#[vars.hoge = true]` 
      - (レコード1 の `#[vars.hoge]`は、そのまま BatchStep A, B, C で参照・変更可能)
    - レコード2 ``#[vars.hoge = false]`　
      - (レコード2 の `#[vars.hoge]` は、そのまま BatchStep A, B, C で参照・変更可能) 
  - Batch Step 
    - Batch Jobの処理の単位
    - Batch Jobは1つ以上のBatch Stepで構成されている
    - それぞれの Step は、それぞれの条件に基づいて、レコードを受け入れる (acceptPolicy, acceptExpression)
      - e.g. acceptExpression: `#[vars.hogeFlag==true]`
             acceptPolicy: `ONLY_FAILURES` (処理に失敗したレコードだけを処理する Batch Step)
                
  - Batch Aggregator 
    - Bulk Insert など、まとめて処理を行う
    - 2種類の設定
      1. Fixed Size (固有のサイズ)
      2. Streaming 
  
  - maxFailedRecords
    - 失敗したレコードを許容する数 (デフォルト 0)

  - batchBlockSize
    - 各スレッドがどのくらいの単位で、各レコードをまとめて処理するか (デフォルト100)
    e.g.  1000  レコード -> batch block size == 100
     - 1-100(100単位)
     - 101-200(100単位)
     - 201-300(100単位)
     ...
     - 901-1000(100単位)
もし、batch block size が1で、４つのレコードを処理する場合...
     ![batchblock=1](https://i.imgur.com/1ayUy4v.png)

  - onComplete フェーズで、payloadはサマリーレポートに置き換わる
```
  {
    "onCompletePhaseException": null,
    "loadingPhaseException": null,
    "totalRecords": 2000,
    "elapsedTimeInMillis": 8909,
    "failedOnCompletePhase": false,
    "failedRecords": 1,
    "loadedRecords": 2000,
    "failedOnInputPhase": false,
    "successfulRecords": 1999,
    "inputPhaseException": null,
    "processedRecords": 2000,
    "failedOnLoadingPhase": false,
    "batchJobInstanceId": "9d6a7e50-5bd0-11ec-8902-a483e7ab0cd9"
  }
```
※変数は、各 Batch Step を超えて参照・変更ができる
※Batch Job スコープの完了後は、各変数を参照することはできない

   
## ウォーターマーク　
重複して処理をしないために、前回のフロー実行時に処理した値のうち、最大の値 (ID) を記憶しておく
e.g 前回の実行時 102 まで処理完了 > 次は 103 以上の値を処理する
 - 自動
   -  Database: On Table Row
   -  File: On New or Updated File
- 手動
   - Object Store: Key-Value ペアのストレージ

Object Store を使用して、あるフローの前回の実行時の状態を記録しておくことが可能


# 8.データの変換 (開発:基礎 モジュール11)

## Q38 関数
```
fun newProdCode(prodCode:Number, prodCategory:String):String =
  prodCode as String ++ prodCategory

var newProdCode2 = (prodCode:Number, prodCategory:String):String 
  -> prodCode as String ++ prodCategory
```
https://developer.mulesoft.com/learn/dataweave/

- DW の関数宣言
   - fun
     `fun funName (arg1, arg2) = arg1 ++ arg2;` 
   - var
      `var funName = (arg1, arg2) -> arg1 ++ arg2;`

- [lookup](https://docs.mulesoft.com/dataweave/2.4/dw-mule-functions-lookup)
`lookup("aFlowName", {})`
// フローの名前, payload を引数に渡す

- モジュールの import 
https://docs.mulesoft.com/dataweave/2.4/dw-strings

モジュールのみインポート
```
import dw::core::Strings
Strings::pluralize("child")
```
モジュールから関数をインポート
```
import pluralize from dw::core::Strings
pluralize("child")
```
モジュールから全ての関数をインポート
```
import * from dw::core::Strings
pluralize("child")
```

```
dw::core::Strings::pluralize("child")
```

※ now() など Core モジュールに入っているものは import なしで使える


確認事項: セレクタ(`.` , `..`, `.*`), 演算子, Core モジュール DW 関数 (`map`, `orderBy` etc),  XML -> Java, Java -> JSON , type(型, as String), @ で XML アトリビュートを操作する

# 9. ルーティング (開発:基礎 モジュール9)
- Choice
  - if, else-if, else
- Scatter-Gather 
  - マルチスレッド
  - 並列処理
  - 非同期に、複数のルートを処理する
  - 結果を、複数の Event を含むオブジェクトで返す
```
{
  "0": {
    ... 
    payload: ...
    ...
  } 
  ... 
  "1": {
    ... 
    payload: ...
    ...
  }
}
```
- First Successful //参考程度
  - あるルートの処理が成功したら、そこで完了して次のプロセッサへ
- Round Robin //参考程度
  - フロー実行時ごとに、順番に実行 (0 -> 1 -> 2 -> 0 -> 1 ...)

# 10.エラー処理
## Q49 グローバルエラーハンドラー(アプリケーションエラーハンドラー)
global elementes > configuration > default error handler
![q49](https://i.imgur.com/i2F005Y.jpg)

## エラーハンドラのスコープ
0. Mule デフォルトエラーハンドラ
- カスタマイズ不可
1. **アプリケーション(グローバルエラーハンドラ)**
- *エラーハンドラが設定されていない時に限り*、実行されるバックアップ用エラーハンドラ 
2. **フローレベル** のエラーハンドラ
- 設定されたフローで起きたエラーを処理
3. **プロセッサレベル** のエラーハンドラ 
- Try スコープで設定された範囲で起きたエラーを処理
  
## エラーハンドラの種類
1. On Error Propagate 
- エラーを伝播する (エラーが起きた際に、エラーを再スローする)

2. On Error Continue 
- エラーを伝播しない (エラーが起きた際に、エラーを再スローせずに飲み込む)

## Error Object
- Mule Event
  - Message
    - attributes
    - payload 
  - Variables
  - **Error** (エラーオブジェクト)
    - _errorType_ (HTTP:NOT_FOUND, HTTP:BAD_REQUEST) \<Object\>
       - _namespace_ (名前空間) \<String\>
         - HTTP, VALIDATION etc
       - _identifier_ (識別子)  \<String\>
         - NOT_FOUND, BAD_REQUEST
    - _description_ \<String\>
      - エラーの詳細
    
Error When 
`#[error.errorType.namespace=="HTTP"]`
 
# 11.アプリケーションのデバッグとトラブルシューティング (DEV:FUN4 モジュール 6)
- DEV:FUN4 モジュール 6　>>> デバッガーの使用方法

# 12.アプリケーションのデプロイと管理 (DEV:FUN4 モジュール 5)
- CloudHub 
  - MuleSoftがホストする Mule ランタイム実行環境
  - CloudHub ワーカーというクラウド上の仮想マシンで、Mule ランタイムをホストし、そこで Mule アプリケーションを実行する
  - 内部的に AWS を使用 (CloudHub ワーカーは、AWS 上の VM (EC2 インスタンス))
  - 管理のリソースを最小に抑えることができる(多くの部分を MuleSoft が管理・ホストする)
- Runtime Manager 
  - ランタイムやアプリケーションのデプロイを管理する
- API Manager
  - Proxy アプリケーションを作成　(Endpoint With Proxy) し、実装を保護・管理するためのポリシーを適用する
  - Basic Endpoint という、保護・実装の一体型の仕組みもあります
    ![proxy](https://i.imgur.com/jYhPHUe.png)

    
- Autodiscovery（オートディスカバリー）
  - API を保護するための仕組み。ID  (Autodiscovery ID) でAPIの保護を管理している
  - API Manager は Autdiscovery ID をもとに「どの API に、どのような保護を実行するべきか」を管理

# その他
皆さんからのフィードバックを活かし、MuleSoft のトレーニングと資格試験をより良いものにしていきます。
ぜひ、[アンケート](https://salesforce.quip.com/Ed56AdCkBwDb)にご協力お願い致します！

おまけ: https://www.youtube.com/watch?v=FNDDfsT152M
