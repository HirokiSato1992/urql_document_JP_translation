`urql/exchange-auth` パッケージには `urql` 用のアドオン `authExchange` が含まれており、JWT トークンによる API 認証にありがちな複雑な認証や再認証のフローを簡単に実装できるようにすることを目的としています。

## インストールとセットアップ

まず、`@urql/exchange-auth` を `urql` と同時にインストールします。

```sh
yarn add @urql/exchange-auth
# もしくは
npm install --save @urql/exchange-auth
```

次に、このパッケージが公開する `authExchange` を `Client` に追加する必要があります。`authExchange` は非同期の exchange なので、すべての `fetchExchange` の前に置く必要がありますが、`cacheExchange` などの他の同期交換の後には置く必要があります。

```js
import { createClient, dedupExchange, cacheExchange, fetchExchange } from 'urql';
import { authExchange } from '@urql/exchange-auth';

const client = createClient({
  url: 'http://localhost:3000/graphql',
  exchanges: [
    dedupExchange,
    cacheExchange,
    authExchange({
      /* config */
    }),
    fetchExchange,
  ],
});
```

`authExchange` はオプションのオブジェクトを受け取ります。オプションは、認証方法がどのように動作するかを設定するために使用されます。内部的には、`authExchange` は認証状態を保持しており、その形はオプションに渡された関数によって決定される。

- `addAuthToOperation` は、認証情報をどのようにオペレーションに追加するかを `authExchange` に伝えるために指定する必要があります。例えば、オペレーションのフェッチヘッダに認証状態を追加する方法などです。
- トークンのリフレッシュやその他の再認証を含む認証フローを `authExchange` に処理させるには、 `getAuth` を指定しなければならない。GraphQL API に変異を送信したり、 `fetch` を使用して帯域外の API リクエストを送信したりすることができる。
- `didAuthError` を指定すると、 `authExchange` が API から認証エラーを検出し、 `getAuth` メソッドと再認証のフローを起動するようになります。
- `willAuthError` を指定すると、期限切れのトークンを検出したり、認証エラーのために操作が失敗しそうかを判断して、 `getAuth` メソッドと再認証のフローを早期に起動させることができます。

## オプション

| Option               | Description                                                  |
| :------------------- | :----------------------------------------------------------- |
| `addAuthToOperation` | 現在の `authState` (`null | T`) とエラー ([`CombinedError`](https://formidable.com/open-source/urql/docs/api/core/#combinederror) 参照) を含むパラメータオブジェクトを受け取ります。これは、認証状態が追加されたのと同じオペレーションを、例えば認証ヘッダとして返す必要があります。 |
| `getAuth`            | このメソッドは `authState` (`null | T`) を受け取ります。そして、`fetch`コールか、[`client.mutation`に似た `mutate`method](https://formidable.com/open-source/urql/docs/api/core/#clientmutation) を使用して認証状態を更新し、新しい `authState` オブジェクトを返さなければならない。もし `null` を受け取った場合は、例えばローカルストレージに保存されている認証状態を返す必要があります。エラーを投げることも可能で、その場合は認証フローが中断され、認証エラーがフォールスルーされます。 |
| `didAuthError`       | エラーが認証エラーであるかどうかを示す `boolean` を返すことができる。`CombinedError` と `authState: T | null` をパラメータとして与えます。 |
| `willAuthError`      | 認証フローを早期に起動するために、有効期限が切れたトークンなどの理由でオペレーションが失敗する可能性があるかどうかを示す `boolean` を提供し、`operation: Operation` と `authState: T | null` をパラメータとして与えます。 |

## 例

`addAuthToOperation` メソッドには、`authState` を操作のフェッチヘッダに追加する関数が頻繁に投入されます。

```js
function addAuthToOperation: ({
  authState,
  operation,
}) {
  // トークンが認証状態でない場合、変更せずに操作を返す
  if (!authState || !authState.token) {
    return operation;
  }

  // fetchOptions は関数にすることができます (クライアント API を参照) が、用途に応じて簡略化することができます。
  const fetchOptions =
    typeof operation.context.fetchOptions === 'function'
      ? operation.context.fetchOptions()
      : operation.context.fetchOptions || {};

  return {
    ...operation,
    context: {
      ...operation.context,
      fetchOptions: {
        ...fetchOptions,
        headers: {
          ...fetchOptions.headers,
          "Authorization": authState.token,
        },
      },
    },
  };
}
```

[他の例は、ここで紹介するドキュメントをご覧ください](https://github.com/FormidableLabs/urql/tree/main/exchanges/auth#quick-start-guide)