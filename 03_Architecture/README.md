`urql` は高度にカスタマイズ可能で柔軟な GraphQL クライアントであり、偶然にもいくつかのデフォルトの [コアパッケージのコア動作](https://formidable.com/open-source/urql/docs/basics/core/) が付属しています。

デフォルトでは、`urql` はアプリを素早く構築できるような最小限の機能を提供することを目的としている。しかし、`urql` は、私たちの使用状況や要求に応じて成長する GraphQL クライアントとして設計されています。最小または最初のGraphQLアプリの構築から、その全機能を利用するようになったとき、`urql`を拡張して好みに合わせてカスタマイズするためのツールを自由に使うことができるのです。

## GraphQL クライアントの使用

以前 GraphQL API を使用したことがあり、アプリで GraphQL を使用することは、データを取得するためのクエリでプレーンな HTTP リクエストを送信するのと同じくらい簡単であることに気付いたかもしれません。

また、GraphQL は、クエリの送信やデータの管理に伴う多くの手作業を抽象化する機会を提供します。最終的には、状態管理の技術的な詳細を詳細に扱うことなく、アプリの構築に集中することができます。

具体的には、`urql`はGraphQLを使用する際の3つの共通点を簡素化します。

- クエリやミューテーションの送信と、その結果の *宣言的* な受信
- キャッシュと状態管理の内部的な抽象化
- *拡張性*とあなたのAPIとの統合の中心点を提供すること

以下のセクションでは、`urql` がこれらの3つの問題を解決する方法と、内部で抽象化されたロジックについて説明します。

## クライアントへのリクエストと操作

もし `urql` が列車だとしたら、終点である私たちの API に到着するまでにいくつかの駅を経由することになるでしょう。それは私たちがクエリやミューテーションを定義するところから始まります。GraphQL のリクエストは、クエリドキュメントと変数に抽象化することができる。`urql` では、これらの GraphQL リクエストはユニークなオブジェクトとして扱われ、クエリドキュメントと変数によって一意に識別される (そのため、この 2 つから `key` が生成される)。この `key` は、クエリドキュメントと変数のハッシュ値であり、[`GraphQLRequest`](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) を一意に識別する。

API にリクエストを送信するときは、まず `urql` の [`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) を使用します。`url` や `requestPolicy` など、GraphQL リクエストをどのように実行するかに関する追加情報を受け取ることができます。

```js
import { Client } from '@urql/core';

new Client({
  url: 'http://localhost:3000/graphql',
  requestPolicy: 'cache-first',
});
```

[「基本」のセクション](https://formidable.com/open-source/urql/docs/basics/) で見てきたバインディングは、[`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) と直接対話し、その上の薄い抽象化された存在です。しかし、["Core Usage" のページ](https://formidable.com/open-source/urql/docs/basics/core/#one-off-queries-and-mutations) にあるように、いくつかのメソッドは直接クライアントに対して呼び出すことができます。

クエリやミューテーションを `Client` に送信すると、内部的には [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) として管理されます。"Operation" は `GraphQLRequest`s の拡張版である。`query`、`variables`、`key`のプロパティを持つだけでなく、`"query"` や `"mutation"` など、実行される操作の `kind` を識別することができる。また、`Client` のオプションは、操作のメタデータを保持する `operation.context` からも取得することができる。



![Operations and Results](https://formidable.com/open-source/urql/static/urql-event-hub.1106f5ab.png)運用と結果



オペレーションを受け取り、それを実行するのは `Client` の責任である。バインディングは内部で `client.executeQuery` や `client.executeMutation` 、あるいは `client.executeSubscription` メソッドを呼び出し、その結果の "ストリーム" を取得することができる。この "ストリーム "にコールバックを登録することで、結果を受け取ることができる。

図では、各オペレーションがリクエストの開始を示すシグナルであり、その時点でコールバックで最終的に結果を受け取ることが期待できることがわかる。結果に興味がなくなったら、特別な "teardown" シグナルが `Client` に発行されます。クライアントの外ではオペレーションを見ることはできませんが、オペレーションは `Client` 上の "Exchanges" を通して行われます。

## クライアントとエクスチェンジ

繰り返しになりますが、選択したフレームワークに対して `urql` のバインディングを使用すると、`Client` 上でメソッドが呼び出されますが、バインディングからバックグラウンドで作成された操作を見ることはありません。`client.executeQuery` のようなメソッドを呼び出し (またはバインディングで呼び出され)、コールバックでサブスクライブすると内部でオペレーションが発行され、後でコールバックが結果と共に呼び出される。



![Operations stream and results stream](https://formidable.com/open-source/urql/static/urql-client-architecture.5b424745.png)操作のストリームと結果のストリーム



私たちは、一度に一つの [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) とその [`OperationResult`s](https://formidable.com/open-source/urql/docs/api/core/#operationresult) にしか興味がないことが分かっていますが、 `Client` はこれらを一つの大きなストリームとして扱います。クライアントには、すべての操作の受信フローが表示されます。

前に学んだように、それぞれの操作は `key` を持ち、受け取るそれぞれの結果はオリジナルの `operation` を持つ。`OperationResult` は `operation` プロパティも保持しているので、 `Client` はどの結果が個々のオペレーションに対応しているかを常に把握することができます。しかし、内部的には、すべてのオペレーションが同時に処理されます。しかし、私たちの視点から見ると

- "ストリーム"を購読し、コールバックで結果を得ることを期待します。
- クライアントがオペレーションを発行し、キャッシュが（同期的に）応答するか、リクエストが私たちの API に送信されると、最終的にいくつかの結果が戻ってきます。
- 最終的に購読を解除すると、`Client` は元のオペレーションと同じ `key` を持つ "teardown" オペレーションを発行し、フローを終了する。

クライアント自身は、オペレーションをどのように処理すればよいのかわかりません。その代わり、"exchanges "を介してオペレーションを送信します。エクスチェンジは[Reduxのミドルウェア](https://redux.js.org/advanced/middleware)のようなもので、すべてのオペレーションとすべての結果にアクセスすることができます。複数のExchangeが連鎖して、オペレーションを処理し、ロジックを実行します。そのうちの一つが `fetchExchange` で、その名の通りAPIにリクエストを送ります。

### オペレーションがエクスチェンジに到達する方法

これで、操作と `Client` に到達する方法がわかりました。

- `Client` へのバインディングやコールはすべて **operation** を作成します。
- このオペレーションは `"query"`, `"mutation"`, `"subscription"` のいずれかであり、一意の `key` を持つ。
- このオペレーションは **Exchanges** に送られ、最終的には `fetchExchange` (または同様の交換) に到達します。
- この操作は API に送信され、`OperationResult` にラップされた **result** が戻ってきます。
- クライアントが `OperationResult` を `operation.key` でフィルタリングして、コールバックを介して **結果のストリーム** を返します。

先ほどの列車の例えに戻ると、オペレーションは列車のように線路の端から終点である私たちのAPIまで移動します。そして結果は、同じ路線を逆走するように戻ってくる。

### 取引所

デフォルトの `@urql/core` に含まれ、`Client` に適用されるエクスチェンジのセットは以下の通りである。

- `dedupExchange`: 保留中の操作を重複排除する (保留中 = 結果を待っている状態)
- `cacheExchange`: [ドキュメントキャッシュ"](https://formidable.com/open-source/urql/docs/basics/document-caching/) によるデフォルトのキャッシュロジック。
- `fetchExchange`: `fetch` を使用して API に操作を送信し、結果を出力ストリームに追加する

Exchanges` オプションを `Client` に手動で渡さない場合は、これらのオプションが適用されます。見てわかるように、exchange は操作と結果に対して多くの力を発揮します。彼らは `Client` のロジックの多くを決定し、重複排除、キャッシュ、APIへのリクエスト送信のようなことを引き受けます。

私たちが利用できるエクスチェンジは以下の通りです。

- [`errorExchange`](https://formidable.com/open-source/urql/docs/api/core/#errorexchange): エラーが発生したときにグローバルコールバックが呼び出されるようにします。
- [`ssrExchange`](https://formidable.com/open-source/urql/docs/advanced/server-side-rendering/): サーバー側のレンダラーが、クライアント側の水分補給の結果を収集できるようにします。
- [`retryExchange`](https://formidable.com/open-source/urql/docs/advanced/retry-operations/): 操作のリトライを許可します。
- [`multipartFetchExchange`](https://formidable.com/open-source/urql/docs/advanced/persistence-and-uploads/#file-uploads): マルチパートのファイルアップロード機能を提供します。
- [`persistedFetchExchange`](https://formidable.com/open-source/urql/docs/advanced/persistence-and-uploads/#automatic-persisted-queries)自動パーシステッドクエリのサポートを提供します。
- [`authExchange`](https://formidable.com/open-source/urql/docs/advanced/authentication/): 複雑な認証フローを簡単に実装できるようになる。
- [`requestPolicyExchange`](https://formidable.com/open-source/urql/docs/api/request-policy-exchange/): 一定時間後に `cache-only` と `cache-first` の操作を自動的に `cache-and-network` にアップグレードする。
- [`refocusExchange`](https://formidable.com/open-source/urql/docs/api/refocus-exchange/): 開いているクエリを追跡し、ウィンドウがフォーカスを取り戻したときにそれらを再描画します。
- `devtoolsExchange`: [urql-devtools](https://github.com/FormidableLabs/urql-devtools) を使用する機能を提供します。

さらに、`@urql/core` の `cacheExchange` で実装されている [ドキュメントキャッシュ](https://formidable.com/open-source/urql/docs/basics/document-caching/) を `urql` の [正規化キャッシュ、グラフキャッシュ](https://formidable.com/open-source/urql/docs/graphcache/) と交換することができるようになった。

[交換についてや、交換をゼロから書く方法については "Authoring Exchanges" ページを参照してください](https://formidable.com/open-source/urql/docs/advanced/authoring-exchanges/)

## `urql` のストリームパターン

前のセクションでは `Client` がどのように動作するかについて多くを学びましたが、いつも曖昧な言葉で学んできました。例えば、「結果のストリーム」を取得するとか、`urql` がすべての操作を「操作のひとつのストリーム」として捉えて、それをエクスチェンジに送るとかです。しかし、**ストリームとは何なのでしょうか**。

一般に、私たちは*ストリーム*を、時間をかけて非同期イベントをプログラムすることを可能にする抽象化として参照しています。JavaScript の文脈では、特に [Observables](https://github.com/tc39/proposal-observable) と [Reactive Programming with Observables.](http://reactivex.io/documentation/observable.html) について考えています。これらの概念は難しく聞こえるかもしれませんが、高レベルの視点から見ると、私たちが話していることは約束と反復子（例えば配列）を組み合わせたものと考えることができます。私たちは複数のイベントを扱っていますが、私たちのコールバックは時間経過とともに呼び出されます。これは、配列に対して `forEach` を呼び出し、その結果が非同期で入ってくることを期待しているようなものです。

ユーザーとしては、[基礎編](https://formidable.com/open-source/urql/docs/basics/)で紹介したフレームワークのバインディングを使用している場合、これらのストリームを実際に見ることはありませんし、バインディングが内部で使用しているので、使用することもないかもしれません。しかし、`Client`を直接使ったり(https://formidable.com/open-source/urql/docs/basics/core/#one-off-queries-and-mutations)、Exchangeを書いたりすると、ストリームを見ることになり、そのAPIを扱わなければならなくなります。

### Wonka ライブラリ

`urql` はストリームに [Wonka](https://github.com/kitten/wonka) ライブラリを使用します。このライブラリには、 `urql` ライブラリとエコシステムのために特別に作られた、いくつかの利点がある。

- 非常に軽量でツリーシェイクが可能であり、minzip でのサイズは約 3.7kB である。
- [Reason](https://reasonml.github.io/) で書かれ、 [Flow](https://flow.org/) と [TypeScript](https://www.typescriptlang.org/v2/) をサポートしているので、クロスプラットフォーム、クロスランゲージに対応している。
- Wonkaは、予測可能で反復可能なツールチェーンであり、可能な限り同期イベントを発行します。

Wonkaの典型的な使い方は、ある値の*source*と*sink*を作成することです。

```js
import { fromArray, map, subscribe, pipe } from 'wonka';

const { unsubscribe } = pipe(
  fromArray([1, 2, 3]),
  map(x => x * 2),
  subscribe(x => {
    console.log(x); // 2, 4, 6
  })
);
```

Wonka では、Observables と同様に、購読が返す `unsubscribe` メソッドを呼び出すことで、ストリームをキャンセルすることができます。

[Wonkaの詳細はドキュメントをご覧ください](https://wonka.kitten.sh/basics/background)。

### クライアントによるストリームのパターン

クライアントで [`client.executeQuery`](https://formidable.com/open-source/urql/docs/api/core/#clientexecutequery) や [`client.query`](https://formidable.com/open-source/urql/docs/api/core/#clientquery) といったメソッドを呼び出すと、Wonkaストリームが返されます。これらは、本質的には単なるコールバックの集まりです。

このストリームを開始するには、[`wonka` の `subscribe`](https://wonka.kitten.sh/api/sinks#subscribe) 関数を使用することができます。この関数にコールバックを渡すと、 `Client` が処理を開始したときに結果を受け取ることができる。購読を停止すると、`Client` は特別な "teardown" オペレーションを私たちの取引所に送信してこのオペレーションを停止します。

```js
import { pipe, subscribe } from 'wonka';

const QUERY = `
  query Test($id: ID!) {
    getUser(id: $id) {
      id
      name
    }
  }
`;

const { unsubscribe } = pipe(
  client.query(QUERY, { id: 'test' }),
  subscribe(result => {
    console.log(result); // { data: ... }
  })
);
```

`Client`で利用可能なAPIについては、[Core API docs](https://formidable.com/open-source/urql/docs/api/core/)を参照してください。