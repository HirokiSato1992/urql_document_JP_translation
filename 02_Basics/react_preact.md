このガイドでは、React と Preact を使って `urql` と `Client` をインストール、セットアップし、データの問い合わせと変更を行う方法について説明します。urql` パッケージと `@urql/preact` パッケージはほとんどの API を共有し、同じように使用されるので、React に関するドキュメントを読むと、`urql` パッケージの代わりに `@urql/preact` パッケージを使用したいと思う以外は、すべての例が基本的に同じになっています。

## はじめに

### インストール

`urql` のインストールは予想通りすぐに終わり、最初は他のパッケージは必要ありません。選択したパッケージマネージャでパッケージをインストールすることにします。

```sh
yarn add urql graphql
# もしくは
npm install --save urql graphql
```

Preact で `urql` を使用するには、 `urql` の代わりに `@urql/preact` をインストールし、そのパッケージからインポートする必要がある。そうしないと、Preact のサンプルはすべて同じになってしまう。

GraphQL に関連するライブラリのほとんどは、 `graphql` パッケージを相互依存関係としてインストールして、特定のバージョン管理要件に適応できるようにする必要がある。そのため、`urql` と同時に `graphql` もインストールする必要がある。

`urql` と `graphql` の両パッケージは [セマンティックバージョニング](https://semver.org/) に従っており、すべての `urql` パッケージは `graphql` と互換性のあるバージョンの範囲を定義しているはずです。しかし、将来的な変更に注意してください。その場合、パッケージマネージャは `graphql` が定義された依存関係の範囲から外れていることを警告するかもしれません。

### `Client`のセットアップ

`urql` および `@urql/preact` パッケージは `createClient` というメソッドをエクスポートしており、このメソッドを使って GraphQL クライアントを作成することができます。この中央の `Client` が、すべての GraphQL リクエストと結果を管理する。

```js
import { createClient } from 'urql';

const client = createClient({
  url: 'http://localhost:3000/graphql',
});
```

最低限必要なのは、`Client` を作成するときに API の `url` を渡して始めることです。

もうひとつの一般的なオプションは `fetchOptions` である。このオプションでは、与えられた API `url` にリクエストを送信したときに `fetch` に渡されるオプションをカスタマイズすることができる。オプションオブジェクトを渡すか、オプションオブジェクトを返す関数を渡すことができる。

次の例では、`Client` が GraphQL API に送信する各 `fetch` リクエストにトークンを追加しています。

```js
const client = createClient({
  url: 'http://localhost:3000/graphql',
  fetchOptions: () => {
    const token = getToken();
    return {
      headers: { authorization: token ? `Bearer ${token}` : '' },
    };
  },
});
```

### `Client`の提供

React と Preact で `Client` を利用するには、[Context API](https://reactjs.org/docs/context.html) を介して提供する必要があります。これは、`Provider` エクスポートの助けを借りて行うことができる。

```jsx
import { createClient, Provider } from 'urql';

const client = createClient({
  url: 'http://localhost:3000/graphql',
});

const App = () => (
  <Provider value={client}>
    <YourRoutes />
  </Provider>
);
```

これで、`Provider`の内部と下にあるすべてのコンポーネントと要素は、私たちのAPIに送信されるGraphQLクエリを使用できるようになりました。

## クエリ

どちらのライブラリも `useQuery` フックと `Query` コンポーネントを提供しています。後者は同じパラメータを受け取りますが、このガイドでは取り上げません。[レンダープロップスコンポーネントが好きな方は、APIドキュメントで調べてみてください](https://formidable.com/open-source/urql/docs/api/urql/#query-component)

### 最初のクエリを実行する

以下の例では、Todoアイテムを含むGraphQL APIからデータをクエリすることを想像してみましょう。さっそく飛び込んでみましょう

```jsx
import { useQuery } from 'urql';

const TodosQuery = `
  query {
    todos {
      id
      title
    }
  }
`;

const Todos = () => {
  const [result, reexecuteQuery] = useQuery({
    query: TodosQuery,
  });

  const { data, fetching, error } = result;

  if (fetching) return <p>Loading...</p>;
  if (error) return <p>Oh no... {error.message}</p>;

  return (
    <ul>
      {data.todos.map(todo => (
        <li key={todo.id}>{todo.title}</li>
      ))}
    </ul>
  );
};
```

ここでは、TODOを取得するための最初のGraphQLクエリを実装しています。useQuery` はオプションを受け取り、タプルを返します。この例では、`query` オプションに GraphQL クエリを設定しました。返されるタプルは、結果オブジェクトと再実行関数を含む配列です。

結果オブジェクトはいくつかのプロパティを含んでいる。`fetching` フィールドはフックがデータを読み込んでいるかどうかを示し、 `data` には API の結果からの実際の `data` が格納されます。また、 `error` は API へのリクエストが失敗した場合や API の結果に `GraphQLError`s が含まれていた場合に設定されます。これについては後で ["Errors" page](https://formidable.com/open-source/urql/docs/basics/errors/) で詳しく説明します。

### 変数

通常、クエリに変数を渡す必要があります。例えば、ページネーションを扱う場合です。この目的のために、 `useQuery` フックは `variables` オプションも受け付けます。

```jsx
const TodosListQuery = `
  query ($from: Int!, $limit: Int!) {
    todos (from: $from, limit: $limit) {
      id
      title
    }
  }
`;

const Todos = ({ from, limit }) => {
  const [result, reexecuteQuery] = useQuery({
    query: TodosListQuery,
    variables: { from, limit },
  });

  // ...
};
```

`fetch` を使用して手動で GraphQL クエリを送信しているときと同様に、変数は GraphQL API に送信される `POST` リクエストボディにアタッチされます。

`useQuery` フックの `variables` (または `query`) オプションが変更されると、 `fetch` は `true` に切り替わり、新しいリクエストが API に送信されます（結果が既にキャッシュされている場合を除く）。

### `useQuery` を一時停止する

場合によっては、 `useQuery` に、ある条件が満たされたときにクエリを実行させ、そうでないときにはクエリを実行させないようにしたいことがあります。例えば、フォームを作成していて、フィールドに入力されたときだけバリデーションクエリを実行させたい場合です。

React のフックはコメントアウトすることができないので、 `useQuery` フックには `pause` オプションがあり、すべての変更を一時的に *凍結* してリクエストを停止することができます。

前の例では、必須の引数を持つクエリを定義しました。変数 `$from` と `$limit` は、ヌルでない `Int!` 値として定義されています。

これらの変数が空のときは実行しないように、先ほど書いたクエリを一時停止して、`null` 変数が実行されないようにしましょう。これを行うには、 `pause` オプションを `true` に設定します。

```jsx
const Todos = ({ from, limit }) => {
  const shouldPause = from === undefined || from === null ||
                      limit === undefined || limit === null;
  const [result, reexecuteQuery] = useQuery({
    query: TodosListQuery,
    variables: { from, limit },
    pause: shouldPause,
  });

  // ...
};
```

これで、必須の変数である `$from` や `$limit` が与えられない場合は常に、クエリが実行されなくなりました。これは、`result.data` が変化しないことも意味します。つまり、変数が変化しても、以前のデータにアクセスすることができるということです。

### リクエストポリシー

このページの前のセクションで明らかになったように、 `useQuery` フックは `query` と `variables` 以外のオプションも受け取ることができます。もうひとつのオプションとして、 `requestPolicy` にも触れておきましょう。

requestPolicy` オプションは、 `Client` のキャッシュから結果を取得する方法を決定する。デフォルトでは `cache-first` に設定されており、キャッシュから結果を取得することを優先するが、API リクエストを送信することにフォールバックすることを意味する。

リクエストポリシーは `urql` の React API に固有のものではなく、そのコアにある共通の機能である。[4 つの異なるポリシーでキャッシュがどのように動作するかについては、 "Document Caching" のページで詳しく説明されています](https://formidable.com/open-source/urql/docs/basics/document-caching/)

```jsx
const [result, reexecuteQuery] = useQuery({
  query: TodosListQuery,
  variables: { from, limit },
  requestPolicy: 'cache-and-network',
});
```

特に、新しいリクエストポリシーはオプションとして `useQuery` フックに直接渡すことができます。このポリシーは、この特定のクエリに対して使用されます。この場合、 `cache-and-network` が使用され、キャッシュがキャッシュした結果を取得した後でも、API からクエリがリフレッシュされる。

内部的には、 `requestPolicy` はいくつかある "**context** オプション" のひとつに過ぎない。コンテキストは、通常の `query` や `variables` とは別に、メタデータを提供する。つまり、 `Client` のデフォルトの `requestPolicy` を変更する場合にも、それを渡すことができる。

```js
import { createClient } from 'urql';

const client = createClient({
  url: 'http://localhost:3000/graphql',
  // 以下はデフォルトでキャッシュファーストではなく、キャッシュアンドネットワークを使用します。
  requestPolicy: 'cache-and-network',
});
```

### コンテキストオプション

前述のように、 `useQuery` の `requestPolicy` オプションは、 `urql` のコンテキストオプションの一部である。実際には、さらにいくつかの組み込みのコンテキストオプションがあり、 `requestPolicy` オプションはそのうちのひとつである。また、すでに見たことがあるオプションとして `url` オプションがあり、これは APIの URL を決定する。これらのオプションは `Client` に限らず、クエリごとに渡すことができる。

```jsx
import { useMemo } from 'react';
import { useQuery } from 'urql';

const Todos = ({ from, limit }) => {
  const [result, reexecuteQuery] = useQuery({
    query: TodosListQuery,
    variables: { from, limit },
    context: useMemo(
      () => ({
        requestPolicy: 'cache-and-network',
        url: 'http://localhost:3000/graphql?debug=true',
      }),
      []
    ),
  });

  // ...
};
```

見てわかるように、 `useQuery` の `context` プロパティは既知の `context` オプションをすべて受け入れることができ、グローバルではなくクエリごとに変更するために使用することが可能です。`Client` は `context` オプションのサブセットを受け付けますが、 `useQuery` オプションは単一のクエリに対して同じことを実行します。[すべての `Context` オプションの一覧は API ドキュメントに記載されています](https://formidable.com/open-source/urql/docs/api/core/#operationcontext)

### クエリの再実行

`useQuery` フックは、 `query` や `variables` などの入力が変更されるたびにクエリを更新して実行しますが、場合によっては、プログラムで新しいクエリをトリガーする必要があるかもしれません。これが `reexecuteQuery` 関数の目的であり、 `useQuery` が返すタプルの 2 番目の項目である。

プログラムでクエリを実行することは、いくつかのケースで役に立ちます。例えば、フックのデータをリフレッシュするために使用することができます。このような場合、クエリの `requestPolicy` を一度だけオーバーライドして、 `network-only` に設定し、キャッシュをスキップさせることもできます。

```jsx
const Todos = ({ from, limit }) => {
  const [result, reexecuteQuery] = useQuery({
    query: TodosListQuery,
    variables: { from, limit },
  });

  const refresh = () => {
    // クエリの再取得とキャッシュのスキップ
    reexecuteQuery({ requestPolicy: 'network-only' });
  };
};
```

上記の例で `refresh` を呼び出すと、再びクエリが強制的に実行され、 `requestPolicy: 'network-only'` を渡しているので、キャッシュをスキップすることができます。

さらに、 `reexecuteQuery` 関数を使用すると、 `pause` が `true` に設定されている場合でも、プログラムによってクエリを実行することができます（通常、自動クエリはすべて停止されます）。これは、一回限りのアクションを実行したり、ポーリングを設定したりするために使用することができます。

```jsx
import { useEffect } from 'react';
import { useQuery } from 'urql';

const Todos = ({ from, limit }) => {
  const [result, reexecuteQuery] = useQuery({
    query: TodosListQuery,
    variables: { from, limit },
    pause: true,
  });

  useEffect(() => {
    if (result.fetching) return;

    // クエリがアイドルの場合、1秒後に再取得するように設定する
    const timerId = setTimeout(() => {
      reexecuteQuery({ requestPolicy: 'network-only' });
    }, 1000);

    return () => clearTimeout(timerId);
  }, [result.fetching, reexecuteQuery]);

  // ...
};
```

さらに、 `useQuery` を使ったトリックがいくつかあります。[APIについてはAPIドキュメントを参照してください](https://formidable.com/open-source/urql/docs/api/urql/#usequery)

## ミューテーション

どちらのライブラリも `useMutation` フックと `Mutation` コンポーネントを提供しています。後者は同じパラメータを受け取りますが、このガイドでは取り上げません。[レンダープロップスコンポーネントが好きなら、APIドキュメントで調べてみてください](https://formidable.com/open-source/urql/docs/api/urql/#mutation-component)

### ミューテーションを送信する

再び、Todoアイテムのための架空のGraphQL APIをピックアップして、例題に飛び込んでみましょう! ここでは、Todoアイテムのタイトルを*更新*するミューテーションをセットアップします。

```jsx
const UpdateTodo = `
  mutation ($id: ID!, $title: String!) {
    updateTodo (id: $id, title: $title) {
      id
      title
    }
  }
`;

const Todo = ({ id, title }) => {
  const [updateTodoResult, updateTodo] = useMutation(UpdateTodo);
};
```

`useQuery` の出力と同様に、 `useMutation` はタプルを返します。タプルの最初の項目は、やはり `fetching`, `error`, そして `data` である。これは `urql` が [操作結果](https://formidable.com/open-source/urql/docs/api/core/#operationresult) を表示する際の一般的なパターンであるため、同じである。

`useQuery` フックとは異なり、 `useMutation` フックは自動的に実行されることはありません。この例では、この時点では変異は行われません。変異を実行するには、代わりにタプルの2番目の項目であるexecute関数 - この例では `updateTodo` - を呼び出す必要があります。

### 変異の結果を使う

`updateTodo` 関数を呼び出すとき、API から戻ってきた結果を取得する方法が 2 つあります。返されたタプルの最初の値である `updateTodoResult` を使用するか、`updateTodo` が返すプロミスを使用するかのどちらかです。

```jsx
const Todo = ({ id, title }) => {
  const [updateTodoResult, updateTodo] = useMutation(UpdateTodo);


  const submit = newTitle => {
    const variables = { id, title: newTitle || '' };
    updateTodo(variables).then(result => {
    // 結果は `updateTodoResult` とほぼ同じですが、`result.fetching` が設定されていないことが例外です。
    // これは OperationResult です。
    });
  };
};
```

result は突然変異の進行状況を UI で表示しなければならないときに便利で、返されたプロミスは突然変異が完了した後に実行される副作用を追加するときに特に便利です。

### 突然変異のエラーの処理

execute関数を呼び出したときに受け取るプロミスは、決して拒否しないことに注意してください。その代わり、常に結果に解決されたプロミスを返します。

エラーをチェックする場合は、代わりに `result.error` を使用します。これは、変異の実行中に何らかのエラーが発生した場合に `CombinedError` としてセットされます。[エラーについて詳しくは、"エラー" のページを参照してください](https://formidable.com/open-source/urql/docs/basics/errors/)

```jsx
const Todo = ({ id, title }) => {
  const [updateTodoResult, updateTodo] = useMutation(UpdateTodo);

  const submit = newTitle => {
    const variables = { id, title: newTitle || '' };
    updateTodo(variables).then(result => {
      if (result.error) {
        console.error('Oh no!', result.error);
      }
    });
  };
};
```

さらに、 `useMutation` を使ったトリックもあります。
[そのAPIについてはAPIドキュメントを参照してください](https://formidable.com/open-source/urql/docs/api/urql/#usemutation)

## 続きを読む

以上で、ReactやPreactで `urql` を使用するための紹介を終了します。残りのドキュメントはフレームワークに依存せず、一般的な `urql` や `@urql/core` パッケージに適用されるもので、どのフレームワークバインディングでも同じです。したがって、次に内部について詳しく知るために、以下のうちの1つについて学びたいと思うかもしれません。

- [デフォルトの "ドキュメントキャッシュ" はどのように動作するのか](https://formidable.com/open-source/urql/docs/basics/document-caching/)
- [エラーはどのように処理され、表現されるのか](https://formidable.com/open-source/urql/docs/basics/errors/)
- [`urql` のアーキテクチャと構造についての簡単な概要](https://formidable.com/open-source/urql/docs/architecture/)
- [認証、アップロード、永続化クエリのような他の機能の設定](https://formidable.com/open-source/urql/docs/advanced/)