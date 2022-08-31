デフォルトでは、`urql` は *Document Caching* と呼ばれる概念を使用します。各クエリの結果をキャッシュすることで、GraphQL API に同じリクエストを繰り返し送信することを回避します。

これは、ブラウザのキャッシュのように動作する。`urql` は、クエリとその変数に基づいて送信される各リクエストのキーを作成します。

デフォルトの *ドキュメントキャッシング* ロジックは、デフォルトの `cacheExchange` で実装されています。["エクスチェンジ" については "アーキテクチャ" のページで詳しく説明します](https://formidable.com/open-source/urql/docs/architecture/)

## 操作キー



![Keys for GraphQL Requests](https://formidable.com/open-source/urql/static/urql-operation-keys.3685c7d3.png)GraphQLリクエストのキー



一旦結果が来ると、そのキーによって無期限にキャッシュされる。つまり、それぞれのユニークなリクエストは正確に1つのキャッシュされた結果を持つことができます。

しかし、キャッシュされた結果を無効にする必要があります。これは、一部の結果が古くなっていることが分かっている場合に、リクエストを再度送信して更新するためです。ドキュメントキャッシングでは、以前にクエリされたデータに対して実行されるミューテーションによって結果が無効になる可能性があると想定しています。

GraphQL では、クエリの *selection set* に `__typename` フィールドを追加することで、クライアントが追加の型情報を要求することができます。このフィールドは結果内のオブジェクトの型名を返し、クエリと変異の間の共通性やデータ依存性を検出するために使用します。

![Document Caching](https://formidable.com/open-source/urql/static/urql-document-caching.25bfb628.png)ドキュメントキャッシング



要するに、他のクエリの結果が同様に含む型を含む変異を送信すると、そのクエリの結果はキャッシュから削除されます。

これはキャッシュの無効化の積極的な形です。しかし、正規化されたデータやIDを扱わない一方で、コンテンツ駆動型のサイトではうまく機能します。

## リクエストポリシー

定義された *リクエストポリシー* は、デフォルトのドキュメントキャッシュが行うことを変更します。デフォルトでは、キャッシュはキャッシュされた結果を優先し、そうでない場合は `cache-first` と呼ばれるリクエストを送信します。これは `cache-first` と呼ばれるものである。

- `cache-first` (デフォルト) はキャッシュされた結果を優先し、それ以前の結果がキャッシュされていない場合は API リクエストの送信にフォールバックする。
- `cache-and-network` はキャッシュされた結果を返しますが、常に API リクエストを送信するため、データを最新の状態に保ちながら素早く表示するのに適しています。
- `network-only` は常に API リクエストを送信し、キャッシュされた結果は無視される。
- `cache-only` は常にキャッシュされた結果か `null` を返します。

キャッシュアンドネットワークポリシーは特に便利で、キャッシュされたデータを即座に表示するだけでなく、バックグラウンドでキャッシュのデータをリフレッシュすることができるからです。これは、キャッシュされた結果に対して `fetching` が `false` になることを意味するが、バックグラウンドで API リクエストが進行している場合もある。

このため、結果には `result.stale` という別のフィールドが存在し、キャッシュされた結果が古いか、バックグラウンドで別のリクエストが送信されていることを示す。

[どのようなリクエストポリシーが利用可能かについては、APIドキュメントを参照してください。](https://formidable.com/open-source/urql/docs/api/core/#requestpolicy-type)

## ドキュメントキャッシュの欠点

このキャッシュには小さなトレードオフがあります! もしデータのリストをリクエストして、APIが空のリストを返した場合、キャッシュはそのリストの `__typename` を見ることができず、それを無効化することができません。

この問題を解決するには、クエリのコンテキストに `additionalTypenames` を指定する方法と、[代わりに "Normalized Caching" に切り替える](https://formidable.com/open-source/urql/docs/graphcache/normalized-caching/)方法の2つがあります。

### 型名の追加

これは、空のリストに対する最初の修正である `additionalTypenames` について詳しく説明するものである。

これが発生する例です。

```js
const query = `query { todos { id name } }`;
const result = { todos: [] };
```

この時点では、このクエリでどのような型が使用できるかはわかりません。したがって、デフォルトのキャッシュを使用する際のベストプラクティスは、このクエリに対して `additionalTypenames` を追加することです。

```js
// リファレンスを安定させる。
const context = useMemo(() => ({ additionalTypenames: ['Todo'] }), []);
const [result] = useQuery({ query, context });
```

これで、キャッシュはリストが空の場合でも、このクエリを無効にするタイミングを知ることができます。

時々、突然変異は `__typename` によって突然変異に直接接続されていないデータを無効にしなければならないので、この機能を突然変異に使うこともできます。

```js
const [result, execute] = useMutation(`mutation($name: String!) { createUser(name: $name) }`);

const onClick = () => {
  execute({ name: 'newName' }, { additionalTypenames: ['Wallet'] });
};
```

これで `mutation` は、それが完了したときに、無効化すべき追加の型があることを知った。