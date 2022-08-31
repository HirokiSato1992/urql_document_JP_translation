## useQuery

以下のプロパティを持つ、単一の必須オプションオブジェクトを入力として受け入れます。

| Prop            | Type                     | Description                                                  |
| :-------------- | :----------------------- | :----------------------------------------------------------- |
| `query`         | `string | DocumentNode` | 実行されるクエリです。プレーンな文字列クエリまたは GraphQL DocumentNode として受け取ることができます。 |
| `variables`     | `?object`                | GraphQL リクエストで使用される変数です。                     |
| `requestPolicy` | `?RequestPolicy`         | キャッシュ戦略を指定するオプションの [リクエストポリシー](https://formidable.com/open-source/urql/docs/api/core/#requestpolicy) です。 |
| `pause`         | `?boolean`               | [実行を一時停止する](https://formidable.com/open-source/urql/docs/basics/react-preact/#pausing-usequery)ことを指示するブーリアン・フラグです。 |
| `context`       | `?object`                | クエリのコンテキスト情報を保持します。                       |

このフックは `[result, executeQuery]` 形式のタプルを返します。

- `result` は [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) の形をしたオブジェクトで、クエリをフェッチしているかどうかを示す `fetching: boolean` プロパティが追加されています。
- `executeQuery` 関数はオプションで [`Partial`](https://formidable.com/open-source/urql/docs/api/core/#operationcontext) を受け取ることができ、呼び出されたときに現在のクエリを再実行することができます。pause` を `true` に設定すると、一時停止中のフックをオーバーライドして、クエリを実行します。

[UseQuery` API の使用方法については、 "Queries" ページを参照してください](https://formidable.com/open-source/urql/docs/basics/react-preact/#queries)

## useMutation

`string | DocumentNode` 型の `query` 引数をひとつ受け取り、 `[result, executeMutation]` 形式のタプルを返します。

- `result` は [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) の形をしたオブジェクトで、突然変異が実行されているかどうかを示す `fetching: boolean` プロパティが追加されています。
- `executeMutation` 関数は変数と、オプションで [`Partial`](https://formidable.com/open-source/urql/docs/api/core/#operationcontext) を受け取ることができ、変異の実行を開始するために使用されるかもしれません。この関数は、[`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) に解決される `Promise` を返します。

[`useMutation` API の使用方法については、 "Mutations" ページを参照してください](https://formidable.com/open-source/urql/docs/basics/react-preact/#mutations)

## useSubscription

以下のプロパティを持つ単一の必須オプションオブジェクトを入力として受け取ります。

| Prop        | Type                     | Description                                                  |
| :---------- | :----------------------- | :----------------------------------------------------------- |
| `query`     | `string | DocumentNode` | 実行されるクエリです。プレーンな文字列クエリまたは GraphQL DocumentNode として受け取ることができます。 |
| `variables` | `?object`                | GraphQL リクエストで使用される変数です。                     |
| `pause`     | `?boolean`               | [実行を一時停止する](https://formidable.com/open-source/urql/docs/basics/react-preact/#pausing-usequery)ことを指示するブーリアン・フラグです。 |
| `context`   | `?object`                | クエリのコンテキスト情報を保持する。                         |

フックはオプションで2番目の引数を受け付けます。この引数は、以下のような型シグネチャを持つハンドラ関数です。

```js
type SubscriptionHandler<T, R> = (previousData: R | undefined, data: T) => R;
```

この関数は、以前のデータ(または `undefined`)とサブスクリプションイベントから入ってくる新しいデータで呼ばれ、時間の経過とともにデータを「減らす」ために使われ、 `result.data` の値を変化させるかもしれません。

このフックは `[result, executeQuery]` 形式のタプルを返します。

- `result` は [`OperationResult`](https://formidable.com/open-source/urql/docs/api/core/#operationresult) のような形のオブジェクトです。
- `executeSubscription` 関数はオプションで [`Partial`](https://formidable.com/open-source/urql/docs/api/core/#operationcontext) を受け付け、呼び出されると現在のサブスクリプションをリスタートします。pause` が `true` に設定されている場合、サブスクリプションを開始し、一時停止していたフックを上書きします。

`fetching: boolean` プロパティは、サーバーが積極的にサブスクリプションを終了させるときに `false` に変更されるかもしれません。デフォルトでは、 `urql` はサブスクリプションを開始することができません。なぜなら、これにはいくつかの追加設定が必要だからです。

[`useSubscription` API の使用方法については、 "Subscriptions" ページを参照してください。](https://formidable.com/open-source/urql/docs/advanced/subscriptions/)

## Query コンポーネント

このコンポーネントは [`useQuery`](https://formidable.com/open-source/urql/docs/api/urql/#usequery) のラッパーで、フックが望ましくない場合のために [render prop API](https://reactjs.org/docs/render-props.html) を公開しています。

`Query` コンポーネントの API は、[`useQuery`](https://formidable.com/open-source/urql/docs/api/urql/#usequery) の API をミラーリングしています。<Query>` が受け取る小道具は、 `useQuery` のオプションオブジェクトと同じです。

クエリーの結果を受け取る関数コールバックは `children` に渡さなければならず、React 要素を返さなければなりません。フックのタプルの第2引数である `executeQuery` は、クエリ結果に追加されるプロパティとして渡されます。

## Mutation コンポーネント

このコンポーネントは [`useMutation`](https://formidable.com/open-source/urql/docs/api/urql/#usemutation) のラッパーで、フックが望ましくない場合のために [render prop API](https://reactjs.org/docs/render-props.html) を公開しています。

`Mutation` コンポーネントは `query` プロップを受け取り、関数のコールバックは `children` に渡して、ミューテーションの結果を受け取り、React 要素を返さなければなりません。`useMutation` が返すタプルの第二引数 `executeMutation` は、ミューテーションの結果オブジェクトに追加されるプロパティとして渡されます。

## Subscription コンポーネント

このコンポーネントは [`useSubscription`](https://formidable.com/open-source/urql/docs/api/urql/#usesubscription) のラッパーで、フックが好ましくない場合のために [render prop API](https://reactjs.org/docs/render-props.html) を公開しています。

`Subscription` コンポーネントの API は [`useSubscription`](https://formidable.com/open-source/urql/docs/api/urql/#usesubscription) の API をミラーリングしています。`<Mutation>` が受け取るプロップは `useSubscription` の options オブジェクトと同じです。オプションで `handler` プロップを渡すことができ、 `useSubscription` フックの場合は代わりに第二引数になります。

サブスクリプションの結果を受け取るコールバック関数は `children` に渡さなければならず、React 要素を返さなければなりません。フックのタプルの第二引数 `executeSubscription` は、サブスクリプションの結果に追加されるプロパティとして渡されます。

## Context

React では、[`Client`](https://formidable.com/open-source/urql/docs/api/core/#client) が使用されることを想定して、プロバイダーを追加して `urql` を使用します。内部的には、`urql` が [React Context](https://reactjs.org/docs/context.html) を作成することを意味する。

このコンテキストの作成されたすべてのパーツは `urql` によってエクスポートされる。

-  `Context`
- `Provider`
- `Consumer`

例を簡単に説明すると、`urql` は `url` を `'/graphql'` に設定したデフォルトのクライアントを作成する。このクライアントは、`urql` のフックをラップする `Provider` が存在しない場合に使用される。しかし、このデフォルトクライアントが誤って使用されないように、デフォルトクライアントの警告がコンソールに出力される。

### useClient

`urql` は `useClient` フックもエクスポートする。これは、以下のような便利なラッパーである。

```js
import React from 'react';
import { Context } from 'urql';

const useClient = () => React.useContext(Context);
```

しかし、このフックは上記のデフォルトのクライアント警告を出力する役割も担っているので、 `urql` の `Context` で `useContext` を手動で使用するよりも推奨されます。