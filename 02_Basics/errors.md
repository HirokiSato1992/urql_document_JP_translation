GraphQL APIを使用する際、2種類のエラーに遭遇する可能性があります。ネットワークエラーと、APIからのGraphQLエラーです。どちらかのエラーに遭遇することはよくあることなので、どちらかを保持し、抽象化できる [`CombinedError`](https://formidable.com/open-source/urql/docs/api/core/#combinederror-class) クラスが用意されています。

ERROR が返される場所 (典型的には API の結果) で `urql` を使用すると、 `CombinedError` に遭遇することがある。この `CombinedError` には、何が問題だったのかを表す 2 つのプロパティのいずれかを指定することができる。

- `networkError` プロパティには、`urql` がネットワークリクエストを行うのを止めるようなエラーが含まれる。
- `graphQLErrors` プロパティには、[GraphQL API から `errors` 配列に受信した `GraphQLError` を正規化したもの](https://graphql.org/graphql-js/error/) を含む配列を指定することができる。

さらに、エラーの `message` はデバッグのために生成され、エラーから結合されます。



![Combined errors](https://formidable.com/open-source/urql/static/urql-combined-error.bfc38c65.png)複合的なエラー



成功したリクエストでは `data` と同時に `error` が返されることがあることは注目に値します。これは、GraphQL では、クエリが部分的に失敗していても、何らかのデータを含んでいることがあるからである。その場合、 `graphQLErrors` と共に `CombinedError` が渡されるが、 `data` はまだセットされている可能性がある。