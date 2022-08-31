> このページは未完成です。他のパターンを提案したり、よくある問題について [GitHub Discussions](https://github.com/FormidableLabs/urql/discussions) で質問したりして、このページを拡張するのに協力してください。

一般に、`urql` の API 表面は小さくてコンパクトである。アプリを作る際によくある問題は、一見すると組み込みの機能ではないように見えますが、リーンなUIでも対応可能なパターンがいくつかあります。このページでは、GraphQLでよくあるUIのパターンや直面しそうな問題を集め、`urql`でどのように取り組めばよいかを説明します。これらの例はReactで書かれますが、他のどのフレームワークにも適用できます。

## 無限スクロール

「無限スクロール "とは、複数のページにまたがってリストを分割することなく、ページのリストに多くのデータをロードする手法です。

これにはいくつかの方法があります。私たちの [正規化キャッシュの章](https://formidable.com/open-source/urql/docs/graphcache/local-resolvers/#pagination) では、`urql` の正規化キャッシュを使ったアプローチを見ています。これはすぐに始めるには適しています。しかし、このアプローチでは、ページを追跡するためにいくつかのUIコードも必要です。この正規化キャッシュの機能を利用したUI実装をどのように作成するか見てみましょう。

```js
import React from 'react';
import { useQuery, gql } from 'urql';

const PageQuery = gql`
  query Page($first: Int!, $after: String) {
    todos(first: $first, after: $after) {
      nodes {
        id
        name
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
`;

const SearchResultPage = ({ variables, isLastPage, onLoadMore }) => {
  const [{ data, fetching, error }] = useQuery({ query: PageQuery, variables });
  const todos = data?.todos;

  return (
    <div>
      {error && <p>Oh no... {error.message}</p>}
      {fetching && <p>Loading...</p>}
      {todos && (
        <>
          {todos.nodes.map(todo => (
            <div key={todo.id}>
              {todo.id}: {todo.name}
            </div>
          ))}
          {isLastPage && todos.pageInfo.hasNextPage && (
            <button
              onClick={() => onLoadMore(todos.pageInfo.endCursor)}
            >
              load more
            </button>
          )}
        </>
      )}
    </div>
  );
}

const Search = () => {
  const [pageVariables, setPageVariables] = useState([
    {
      first: 10,
      after: '',
    },
  ]);

  return (
    <div>
      {pageVariables.map((variables, i) => (
        <SearchResultPage
          key={'' + variables.after}
          variables={variables}
          isLastPage={i === pageVariables.length - 1}
          onLoadMore={after =>
            setPageVariables([...pageVariables, { after, first: 10 }])
          }
        />
      ))}
    </div>
  );
}
```

ここでは、遭遇したすべての `variables` の配列を保持し、それらを使ってそれぞれの `result` ページをレンダリングしています。これは、常に変化する長いリストを持つのではなく、追加されたページを再レンダリングするだけです。[このパターンの完全なコード例は、Graphcache のページネーションというトピックの example フォルダにあります](https://github.com/FormidableLabs/urql/tree/main/examples/with-graphcache-pagination)。

また、これを実現するために正規化キャッシュを使用する必要はありません。コンポーネント間で個々のリストをチャンクに分割することができる限り、UIコードでこの問題を完全に解決することもできます。[これを実現する方法については、サンプルコードをご覧ください](https://github.com/FormidableLabs/urql/tree/main/examples/with-pagination)

## データのプリフェッチ

新しいページを開く前に、そのページのデータを読み込む必要がある場合があります。たとえば、JSバンドルの読み込み中に読み込む場合などです。これは、Reactバインディングを直接使用せずにメソッドを呼び出すことで、`Client`の助けを借りて行うことができます。

```js
import React from 'react';
import { useClient, gql } from 'urql';

const TodoQuery = gql`
  query Todo($id: ID!) {
    todo(id: $id) {
      id
      name
    }
  }
`;

const Component = () => {
  const client = useClient();
  const router = useRouter();

  const transitionPage = React.useCallback(async (id) => {
    const loadJSBundle = import('./page.js');
    const loadData = client.query(TodoQuery, { id }).toPromise();
    await Promise.all([loadJSBundle, loadData]);
    router.push(`/todo/${id}`);
  }, []);

  return (
    <button onClick={() => transitionPage('1')}>
      Go to todo 1
    </button>
  )
}
```

ここでは、遷移の開始時に `client.query` を呼び出して、クエリを準備しています。そして、このクエリに対して `toPromise()` を呼び出し、クエリをアクティブにしています。私たちの `Client` とそのキャッシュは結果を共有します。つまり、新しいページに移動する前に、すでにクエリを開始したり、完了したりしているのです。

## 遅延クエリ

クエリを "遅延して "開始することが、後々必要になることがよくあります。これは、新しいコンポーネントがマウントされたときに、すぐにクエリを開始しないことを意味します。

useQuery` フックのように自動的に起動する `urql` のパーツには、 [`pause` オプション](https://formidable.com/open-source/urql/docs/basics/react-preact/#pausing-usequery) という概念があります。 このオプションは、フックが自動的に新しいクエリを開始しないようにするために使用します。

```js
import React from 'react';
import { useQuery, gql } from 'urql';


const TodoQuery = gql`
  query Todos {
    todos {
      id
      name
    }
  }
`;

const Component = () => {
  const [result, fetch] = useQuery({ query: TodoQuery, pause: true });
  const router = useRouter();

  return (
    <button onClick={fetch}>
      Load todos
    </button>
  )
}
```

フックの一時停止を解除して取得を開始するか、この例のように、返された関数を呼び出して手動でクエリーを開始することができます。

## focus と stale time に反応する

urqlでは、"Exchanges "という拡張パターンを利用して、クライアントへのデータの出入りを操作することができます。

- [古くなった時間](https://github.com/FormidableLabs/urql/tree/main/exchanges/request-policy)
- [フォーカス](https://github.com/FormidableLabs/urql/tree/main/exchanges/refocus)

これらのパターンのいずれかを導入したい場合、パッケージを追加して `Client` の `exchanges` プロパティに追加します。この2つの場合、キャッシュの前に追加しないとリクエストがアップグレードされません。

```js
import { createClient, dedupExchange, cacheExchange, fetchExchange } from 'urql';
import { refocusExchange } from '@urql/exchange-refocus';

const client = createClient({
  url: 'some-url',
  exchanges: [
    dedupExchange,
    refocusExchange(),
    cacheExchange,
    fetchExchange,
  ]
})
```

このようなパターンに反応するのは、それだけでいいのです。