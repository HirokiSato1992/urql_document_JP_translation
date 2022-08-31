`urql/core` パッケージは、すべてのフレームワークのバインディングの基礎となるものです。各バインディングパッケージ、例えば [`urql` for React](https://formidable.com/open-source/urql/docs/api/urql/) や [`@urql/preact`](https://formidable.com/open-source/urql/docs/api/preact/) は、コアロジックを再利用して `@urql/core` からすべてのエクスポートを再実行します。したがって、ユーティリティに直接アクセスせず、Node.js 環境でもなく、フレームワークバインディングを使用している場合は、フレームワークバインディングパッケージから直接インポートすることになるでしょう。

[`urql` のコアについては "コアパッケージ" のページで詳しく説明しています](https://formidable.com/open-source/urql/docs/basics/core/)

## クライアント

`Client`は、すべての操作と交換パイプラインへの継続的なリクエストを管理します。作成時にいくつかのオプションを指定することができる。

`@urql/core` は `createClient()` も公開しており、これは `new Client()` を呼び出すための便利な代替手段である。

| 入力              | タイプ                             | 説明                                                         |
| :---------------- | :--------------------------------- | :----------------------------------------------------------- |
| `exchanges`       | `Exchange[]`                       | クライアントが `defaultExchanges` のリストの代わりに使用する `Exchange` の配列です。 |
| `url`             | `string`                           | `fetchExchange` が使用する GraphQL API の URL です。         |
| `fetchOptions`    | `RequestInit | () => RequestInit` | `fetchExchange` の `fetch` がリクエストする際に使用する追加の `fetchOptions` |
| `fetch`           | `typeof fetch`                     | `fetchExchange` が `window.fetch` の代わりに使用する `fetch` の代替実装 |
| `suspense`        | `?boolean`                         | サーバーサイドレンダリング時にデータをプリフェッチするための実験的な React サスペンスモードを有効にします |
| `requestPolicy`   | `?RequestPolicy`                   | 使用されるデフォルトのリクエストポリシーを変更します。デフォルトでは `cache-first` となります。 |
| `preferGetMethod` | `?boolean`                         | これは `fetchExchange` によって拾われ、すべてのクエリ（ミューテーションではない）をPOSTではなくHTTP GETメソッドを使用して送信するように強制されます。 |
| `maskTypename`    | `?boolean`                         | [`OperationResult`s](https://formidable.com/open-source/urql/docs/api/core/#operationresult) 上のすべての `data` に対して、 `Client` が自動的に `maskTypename` ユーティリティを適用できるようになります。これにより、 `__typename` プロパティは列挙できないようになります。 |

### client.executeQuery

[`GraphQLRequest`](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) と、オプションで `Partial<OperationContext>` を受け付け、[`Source`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) - クエリ結果のストリームで、購読することが可能です。

内部的には、返されたソースを購読すると、`kind` を `'query'` に設定した [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) を生成し、交換パイプライン上でディスパッチします。このオペレーションを聞いている購読者がいなくなり、クエリーソースの購読を解除すると、`Client` は "teardown" オペレーションをディスパッチします。

- [このメソッドを直接使用する代わりに、`client.query` ショートカットを使用するとよいでしょう](https://formidable.com/open-source/urql/docs/api/core/#clientquery)
- `GraphQLRequest` オブジェクトを生成するユーティリティについては [`createRequest` を参照してください。](https://formidable.com/open-source/urql/docs/api/core/#createrequest)

### client.executeSubscription

これは機能的には `client.executeQuery` と同じですが、代わりに `kind` を `'subscription'` に設定して、サブスクリプションに対する操作を作成します。

### client.executeMutation

これは機能的には `client.executeQuery` と同じですが、代わりに `kind` を `'mutation'` に設定して、ミューテーションに対する操作を作成します。

ミューテーションソースは常に1つの[`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult)で応答し、完了することが保証されています。

### client.query

これは [`client.executeQuery`](https://formidable.com/open-source/urql/docs/api/core/#clientexecutequery) の略記法です。クエリ (`DocumentNode | string`) と変数を別々に受け取り、自動的に [`GraphQLRequest`](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) [`createRequest`](https://formidable.com/open-source/urql/docs/api/core/#createrequest) を生成することができます。

返された `Source<OperationResult>` には `toPromise` メソッドも追加されるので、ストリームを便利にプロミスに変換できるようになります。

```js
import { pipe, subscribe } from 'wonka';

const { unsubscribe } = pipe(
  client.query('{ test }', {
    /* vars */
  }),
  subscribe(result => {
    console.log(result); // 操作結果
  })
);

// または toPromise を使用すると、結果を 1 つに制限することができます。
client
  .query('{ test }', {
    /* vars */
  })
  .toPromise()
  .then(result => {
    console.log(result); // 操作結果
  });
```

[このAPIの使い方は、「コアパッケージ」のページで詳しく説明しています。](https://formidable.com/open-source/urql/docs/basics/core/#one-off-queries-and-mutations)

### client.mutation

これは [`client.query`](https://formidable.com/open-source/urql/docs/api/core/#clientquery) と似ていますが、代わりにミューテーションをディスパッチします。

[この API の使用方法については "Core Package" ページを参照してください](https://formidable.com/open-source/urql/docs/basics/core/#one-off-queries-and-mutations)

### client.subscription

これは [`client.query`](https://formidable.com/open-source/urql/docs/api/core/#clientquery) と似ていますが、返すストリームに対して `toPromise()` ヘルパーメソッドを提供しません。

[このAPIの使用方法については、"Subscription" ページを参照してください。](https://formidable.com/open-source/urql/docs/advanced/subscriptions/)

### client.reexecuteOperation

このメソッドは *Exchanges* で一般的に使用され、`Client` 上の [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) を再実行します。これは、指定された [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) の購読者がまだいる場合にのみ再実行されます。

例えば、このメソッドは `cacheExchange` によって、[`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) がキャッシュ内で無効になり、再実行される必要があるときに使用されます。

### client.readQuery

このメソッドは通常、キャッシュからデータを同期的に読み込むために使用されます。値がすぐに返される場合は [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) を返し、値が返されない場合はすべての副作用をキャンセルした上で `null` を返します。

## コンバインドエラー

`CombinedError` は `urql` で使用され、ネットワークエラーや `GraphQLError` が GraphQL リクエスト中に発生した場合に正規化するために使用される。

| 入力            | タイプ                           | 説明                                                         |
| :-------------- | :------------------------------- | :----------------------------------------------------------- |
| `networkError`  | `?Error`                         | GraphQLリクエストを送信しようとした際に発生した可能性のある予期せぬエラーです。 |
| `graphQLErrors` | `?Array<string | GraphQLError>` | GraphQL API から返された GraphQL Errors (もしあれば)。       |
| `response`      | `?any`                           | フェッチコールの生のレスポンスオブジェクト(もしあれば)       |

[`urql` のエラーについては、"エラー" ページを参照してください](https://formidable.com/open-source/urql/docs/basics/errors/)

## タイプ

### GraphQLRequest

これは、すべてのGraphQLリクエストの**入力**としてよく登場します。これは `query` と、オプションで `variables` から構成されています。

| Prop        | タイプ         | 説明                                                         |
| :---------- | :------------- | :----------------------------------------------------------- |
| `key`       | `number`       | `query` と `variables` の正確な組み合わせを識別するための一意のキーで、安定したハッシュを使用して生成されます。 |
| `query`     | `DocumentNode` | 実行されるクエリ。プレーンな文字列のクエリ、または GraphQL DocumentNode として受け付ける。 |
| `variables` | `?object`      | GraphQL リクエストで使用される変数です。                     |

`key` プロパティは `query` と `variables` の両方のハッシュで、リクエストを一意に識別することができる。GraphQL では変数は順序に依存しないため、 `variables` を渡す際には、同じ変数を異なる順序で渡しても同じ `key` になるように、安定した文字列化されていることが保証される。

[`GraphQLRequest` は、 `createRequest` ヘルパーを使用して手動で作成することができる](https://formidable.com/open-source/urql/docs/api/core/#createrequest)

### OperationType

これは、交換が実行する必要がある*操作の種類*を決定します。これは以下のうちのいずれか1つです。

- `'subscription'`
- `'query'`
- `'mutation'`
- `'teardown'`

`teardown'` オペレーションは、受け取った `'teardown'` オペレーションと同じキーで進行中のオペレーションをキャンセルするようエクスチェンジに指示するという点で特別なものです。

### オペレーション

GraphQL リクエストを通知する、すべての交換のための入力です。[`GraphQLRequest` 型](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) を拡張し、これらの追加プロパティを含みます。

| Prop      | タイプ             | Description                                   |
| :-------- | :----------------- | :-------------------------------------------- |
| `kind`    | `OperationType`    | 実行される GraphQL オペレーションの種類です。 |
| `context` | `OperationContext` | 交換に渡される追加のメタデータ。              |

これは `kind` プロパティの非推奨のエイリアスであり、使用すると非推奨の警告が出力されます。

### RequestPolicy (リクエストポリシー)

これは、キャッシュ・エクスチェンジが操作を実行するために使用する戦略を決定します。カスタムキャッシュエクスチェンジを実装する場合、これらのポリシーを処理することが推奨されます。

- `'cache-first'` (デフォルト)
- `'cache-only'`
- `'network-only'`
- `'cache-and-network'`

[リクエストポリシーについては、「ドキュメントキャッシング」のページをご覧ください](https://formidable.com/open-source/urql/docs/basics/document-caching/#request-policies)

### OperationContext

コンテキストは、しばしば個々の交換のためのオプションやメタデータを保持しますが、`urql` の [`Operation`s](https://formidable.com/open-source/urql/docs/api/core/#operation) を扱うほぼすべての API メソッドから渡すことができるカスタムデータを含むこともできます。

これらのオプションのいくつかは `Client` が初期化されるときに設定されます。したがって、以下のプロパティのリストには、`Client` にも存在するオプションが含まれている可能性があります。

| Prop                  | タイプ                                | 説明                                                         |
| :-------------------- | :------------------------------------ | :----------------------------------------------------------- |
| `fetchOptions`        | `?RequestInit | (() => RequestInit)` | `fetchExchange` の `fetch` がリクエストを行うために使用する追加の `fetchOptions` を指定します。 |
| `fetch`               | `typeof fetch`                        | `fetchExchange` が `window.fetch` の代わりに使用する `fetch` の代替実装｜`RequestPolicy ` `fetchOptions` `window.fetch` `fetchExchange` が使用する `fetch` の代替実装 |
| `requestPolicy`       | `RequestPolicy`                       | キャッシュ戦略を指定するために使用するオプションの [リクエストポリシー](https://formidable.com/open-source/urql/docs/basics/document-caching/#request-policies)です。 |
| `url`                 | `string`                              | GraphQLのエンドポイント、GETの場合は絶対パスを使用する必要がある |
| `meta`                | `?OperationDebugMeta`                 | devtools の開発時にのみ利用可能なメタデータです。            |
| `suspense`            | `?boolean`                            | サスペンスが有効かどうか。                                   |
| `preferGetMethod`     | `?boolean`                            | クエリに HTTP GET を使用するように `fetchExchange` に指示します。 |
| `additionalTypenames` | `?string[]`                           | 特定の型名に依存する操作を指示することができます（ドキュメントキャッシュで使用されます） |

また、カスタムエクスチェンジにさらに情報を送るために使用することができる、追加の型なしパラメータを受け付けます。

### OperationResult

すべての GraphQL リクエストの結果、つまり `Operation` です。典型的な GraphQL API から返されるものと非常に似ていますが、少し強化され、正規化されています。

| Prop         | タイプ                 | 説明                                                         |
| :----------- | :--------------------- | :----------------------------------------------------------- |
| `operation`  | `Operation`            | この操作の結果である。                                       |
| `data`       | `?any`                 | 指定されたクエリによって返されるデータです。                 |
| `error`      | `?CombinedError`       | ネットワークや `GraphQLError` をラップした [`CombinedError`](https://formidable.com/open-source/urql/docs/api/core/#combinederror) インスタンス (存在する場合) |
| `extensions` | `?Record<string, any>` | GraphQL サーバーが返したかもしれない拡張機能。               |
| `stale`      | `?boolean`             | `data` が不完全または古く、結果がすぐに更新されることを示すために、エクスチェンジで `true` に設定されるフラグです。 |

### ExchangeInput

これは [`Exchange`](https://formidable.com/open-source/urql/docs/api/core/#exchange) が [`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) によって初期化されるときに受け取る入力です。

| 入力      | タイプ       | 説明                                                         |
| :-------- | :----------- | :----------------------------------------------------------- |
| `forward` | `ExchangeIO` | 観測可能な操作を受け取り、その結果を返す役割を担う関数       |
| `client`  | `Client`     | urql アプリケーション全体のクライアントライブラリです。各 execute メソッドは GraphQL リクエストを開始し、結果のストリームを返します。 |

### Exchange

エクスチェンジは `urql` のロジックの小さなチャンクを抽象化したものです。これらは小さなビルディングブロックであり、「ミドルウェア」に似ています。

[*Exchange* については "Authoring Exchanges" ページを参照してください。](https://formidable.com/open-source/urql/docs/advanced/authoring-exchanges/)

`Exchange` は、[`ExchangeInput`](https://formidable.com/open-source/urql/docs/api/core/#exchangeinput) を受け取り、`ExchangeIO` 関数を返す関数と定義されています。`ExchangeIO` 関数は操作のストリームを受け取り、その結果のストリームを返さなければなりません。例えば `dedupExchange` のように純粋にデータを変換するだけの交換の場合は、次の交換の `ExchangeIO` 関数である `forward` を呼び出して、結果のストリームを取得します。

```js
type ExchangeIO = (Source<Operation>) => Source<OperationResult>;
type Exchange = ExchangeInput => ExchangeIO;
```


[ストリームをまだご覧になっていない方は、"アーキテクチャ "のページで "ストリームパターン "について詳しく説明されています](https://formidable.com/open-source/urql/docs/architecture/)

## Exchanges

### cacheExchange

["ドキュメントキャッシュ" ページ](https://formidable.com/open-source/urql/docs/basics/document-caching/)に記載されている `cacheExchange` です。これは `Options => Exchange` 型です。

### subscriptionExchange

["Subscriptions" ページに記載](https://formidable.com/open-source/urql/docs/advanced/subscriptions/)されている `subscriptionExchange` です。 これは `Options => Exchange` というタイプです。

これは一つの入力を受け付けます。`{ forwardSubscription }` です。これはエンリッチドオペレーションを受け取る関数で、 `GraphQLResult` を `data` と `errors` とともにストリームする Observable ライクなオブジェクトを返さなければなりません。

`forwardSubscription` 関数は、一般的に [`subscriptions-transport-ws` パッケージ](https://github.com/apollographql/subscriptions-transport-ws) に接続されています。

### ssrExchange

["サーバーサイドレンダリング" のページで説明](https://formidable.com/open-source/urql/docs/advanced/server-side-rendering/)されている `ssrExchange` です。これは `Options => Exchange` という型である。

これは完全にオプションで、サーバーサイドのレンダリングデータに水分補給したキャッシュを投入します。 `isClient` には `true` または `false` を設定して、キャッシュに書き込む (サーバーサイド) かキャッシュから読み込む (クライアントサイド) かを指定できます。また `staleWhileRevalidate` では水分補給したデータを古いデータとして扱い、 `network-only` 要求ポリシーを使って操作を再実行して最新のデータを再取得します。

デフォルトでは、 `isClient` は `Client.suspense` モードが無効な場合は `true` に、 `Client.suspense` モードが有効な場合は `false` に設定されています。

これは、基本編でも説明した、サーバサイドでクエリされたデータを抽出するために使用することができます。また、クライアントサイドでは、サーバサイドのレンダリングデータを復元するために使用されます。

この関数が呼ばれると、`Exchange` が生成され、それには2つのメソッドがあります。

- `.restoreData(data)` は、通常はクライアントサイドでデータをインジェクトするために使用します。
- `.restoreData(data)` は、通常はクライアントサイドでデータを注入するために使用します。 `.extractData()` は、通常はサーバーサイドでレンダリングデータを抽出するために使用します。

基本的に、`ssrExchange` は小さなキャッシュで、サーバーサイドのレンダリングパスの間にデータを収集し、クライアントサイドのキャッシュに同じデータを投入できるようにします。

React の水分補給中にこのキャッシュは空になり、非アクティブになって水分補給後のクエリの結果を変更しなくなります。

このキャッシュは `cacheExchange` のような他のキャッシュ交換の後に使用し、 `fetchExchange` のような *非同期* 交換の前に使用する必要があります。

### debugExchange

受信した `Operation` を `console.log` に、完了した `OperationResult` を `console.log` に書き出す交換機です。

### dedupExchange

まだ対応する `OperationResult` を返していない、進行中の `Operation` を追跡するためのエクスチェンジです。同じ `Operation` がすでに受信されていて、まだ結果を待っている場合は、受信した `Operation` の重複はフィルタリングされます。

### フェッチエクスチェンジ

`Exchange` 型の `fetchExchange` は `'query'` と `'mutation'` 型の操作を `fetch` を使用して GraphQL API に送信する役割を果たします。

### errorExchange

エラーを検査するためのExchangeです。これはロギングや、異なるタイプのエラーに反応するのに便利です（例えば、パーミッションエラーの場合にユーザーをログアウトさせるなど）。

```ts
errorExchange({
  onError: (error: CombinedError, operation: Operation) => {
    console.log('An error!', error);
  },
});
```

## Utilities

### gql

これは `gql` タグ付きテンプレートリテラル関数であり、一般に `graphql-tag` として知られているものと同じである。タグ付きテンプレートリテラルで GraphQL ドキュメントを書くために使用することができ、 `createRequest` の `key`s のキャッシュに対してパースされた `DocumentNode` を返します。

```js
import { gql } from '@urql/core';

const SharedFragment = gql`
  fragment UserFrag on User {
    id
    name
  }
`;

gql`
  query {
    user
    ...UserFrag
  }

  ${SharedFragment}
`;
```

この関数は `graphql-tag` とは異なり、ドキュメント内のフラグメントの名前が重複している場合に開発中に警告を出力する。ただし、グローバルに重複している場合は警告を出力しない。

### stringifyVariables

この関数は `JSON.stringify` のバリエーションで、文字列化されるオブジェクトのキーをソートし、キーの順序が異なる2つのオブジェクトが同じ文字列に安定して文字列化されることを保証します。

```js
stringifyVariables({ a: 1, b: 2 }); // {"a":1,"b":2}
stringifyVariables({ b: 2, a: 1 }); // {"a":1,"b":2}
```

### createRequest

このユーティリティは `string | DocumentNode` 型の GraphQL クエリと、オプションで変数のオブジェクトを受け取り、[`GraphQLRequest` オブジェクト](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) を返します。

[`client.executeQuery`](https://formidable.com/open-source/urql/docs/api/core/#clientexecutequery) やその他の execute メソッドは [`GraphQLRequest`s](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) しか受け付けないので、このヘルパーは最初にリクエストを作成するためによく使用されます。`client.query`](https://formidable.com/open-source/urql/docs/api/core/#clientquery) と [`client.mutation`](https://formidable.com/open-source/urql/docs/api/core/#clientmutation) メソッドも同様にこのヘルパーを使用してリクエストを作成します。

このヘルパーは `GraphQLRequest` に対して一意の `key` を作成します。これは `query` と `variables` (変数) が渡された場合のハッシュ値である。`variables` は [`stringifyVariables`](https://formidable.com/open-source/urql/docs/api/core/#stringifyvariables) を使用して文字列化され、安定した JSON 文字列が出力される。

さらに、このユーティリティは `query` の参照が安定したままであることを保証する。つまり、同じ `query` が文字列として、あるいは新しい `DocumentNode` として渡された場合、出力される `DocumentNode` の参照は常に同じになります。

### makeOperation

このユーティリティは、[`GraphQLRequest` オブジェクト](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) を新しい [`Operation` オブジェクト](https://formidable.com/open-source/urql/docs/api/core/#operation) に変換するか、`Operation` をコピーするために使用されます。また、 `kind` プロパティと、非推奨の警告を出力する `operationName` エイリアスが追加されます。

また、3つの引数を受け取ることができる。

- `Operation` の `kind` ([`OperationType` を参照](https://formidable.com/open-source/urql/docs/api/core/#operationtype))
- コピーすべき [`GraphQLRequest` オブジェクト](https://formidable.com/open-source/urql/docs/api/core/#graphqlrequest) または別の [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) 。
- そして、オプションで [部分的な `OperationContext` オブジェクト](https://formidable.com/open-source/urql/docs/api/core/#operationcontext) を指定することができます。この引数は、第二引数として渡される操作からコンテキストをコピーする場合には、省略することができます。

したがって、このユーティリティの有効な使い方は以下のとおりです。

```js
// ゼロから新しいオペレーションを作成する
makeOperation('query', createRequest(query, variables), client.createOperationContext(opts));

// オペレーションを「teardown」オペレーションにする
makeOperation('teardown', operation);

// 既存の操作のコンテキストを変更しながらコピーする
makeOperation(operation.kind, operation, {
  ...operation.context,
  preferGetMethod: true,
});
```

### makeResult

GraphQL API の結果を [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) に変換するためのヘルパー関数です。

引数として [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation) と API の結果、そしてオプションとしてデバッグ用のオリジナルの `FetchResponse` を順に受け取ります。

### makeErrorResult

これは、一般的なエラーまたはネットワークエラーで失敗した GraphQL API リクエストの [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) を生成するヘルパー関数です。

引数として [`Operation`](https://formidable.com/open-source/urql/docs/api/core/#operation)、エラー、そしてオプションとしてデバッグ用のオリジナルの `FetchResponse` を順に受け取ります。

### formatDocument

このユーティリティは [`cacheExchange`](https://formidable.com/open-source/urql/docs/api/core/#cacheexchange) と [Graphcache](https://formidable.com/open-source/urql/docs/graphcache/) で使用され、GraphQL の `DocumentNode` に `__typename` フィールドを追加するために使用されます。

### maskTypename

このユーティリティは、[`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) の `data` のような GraphQL `data` オブジェクトを受け取り、すべての `__typename` プロパティを非列挙可能としてマークします。

[`formatDocument`](https://formidable.com/open-source/urql/docs/api/core/#formatdocument) は、しばしば `urql` によって自動的に使用され、すべての結果に `__typename` フィールドを追加します。しかし、これでは、よくあるユースケースである、変異時の変数や入力にデータを戻すことができないことがよくあります。このユーティリティはこれらのフィールドを隠すので、この問題を解決することができます。

これは `maskTypename` オプションが有効なときに [`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) によって使用されます。

### defaultExchanges

これは `Client` が `exchanges` オプションを渡さなかった場合に使用する、デフォルトの `Exchange` の配列である。

```js
const defaultExchanges = [dedupExchange, cacheExchange, fetchExchange];
```

### composeExchanges

このユーティリティは `Exchange` の配列を受け取り、それらを1つの `Exchange` に合成します。これは与えられた順番に左から右へと連鎖します。

```js
function composeExchanges(Exchange[]): Exchange;
```

これはいくつかの交換を組み合わせるために使用することができ、[`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) が `exchanges` の入力を処理するためにも使用されます。