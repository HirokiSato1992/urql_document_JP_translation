`urq`lは、高度にカスタマイズ可能で汎用性の高いGraphQLクライアントで、成長に合わせて正規化キャッシングなどの機能を追加していくことができます。GraphQLの初心者にとって使いやすく、また、動的な単一アプリケーションや高度にカスタマイズされたGraphQLインフラストラクチャをサポートするために拡張できるように構築されています。つまり、`urql`は使いやすさと適応性を優先しているのです。

GraphQLを採用すると、`urql`が主要なデータ層になり、「[ドキュメントキャッシュ](https://formidable.com/open-source/urql/docs/basics/document-caching/)」によるコンテンツの多いページや、「[正規化キャッシュ](https://formidable.com/open-source/urql/docs/graphcache/normalized-caching/)」による動的でデータの多いアプリを扱うことができるようになります。

urqlは、接続された部品やパッケージの集合体として理解することができます。私たちが選択したフレームワークのパッケージを1つインストールするだけでよい場合。そして、APIにGraphQLリクエストを宣言的に送ることができるようになります。フレームワークのパッケージは、`urql`（React用）、`@urql/preact`、`@urql/svelte`、`@urql/vue`のように、すべて[コアパッケージである@urql/core](https://formidable.com/open-source/urql/docs/basics/core/)をラップしています。`urql`をアプリケーションに実装していく過程で、[Exchangesと呼ばれる「アドオンパッケージ」](https://formidable.com/open-source/urql/docs/advanced/authoring-exchanges/)を追加して拡張していくことができます。

この時点でまだ`urql`を使うかどうか迷っている場合は、[比較のページを見て](https://formidable.com/open-source/urql/docs/comparison/)、`urql`があなたが探しているすべての機能をサポートしているかどうかをチェックしてみてください。



## どれで始めるか

以下のスタートアップガイドを用意しています。

- [**React/Preact**](https://formidable.com/open-source/urql/docs/basics/react-preact/) は、React/Preactのバインディングを操作する方法を説明します。
- [**Vue**](https://formidable.com/open-source/urql/docs/basics/vue/) は、Vue 3 のバインディングを使用する方法について説明します。
- [**Svelte**](https://formidable.com/open-source/urql/docs/basics/svelte/) covers how to work with the bindings for Svelte.
- [**Core Package**](https://formidable.com/open-source/urql/docs/basics/core/) は、共有された「コアAPI」と、それらをNode.jsで直接または命令的に使用する方法について説明します。

これらの各セクションでは、フレームワークバインディングのインストールとセットアップ方法、クエリの書き方、ミューテーションの送信方法など、具体的な手順を説明します。



## ドキュメンテーションに従う

このドキュメントは、使用レベルや関心のある分野をカバーするグループやセクションに分かれています。

- **基本編**は、私たちが選んだフレームワークの「入門」ガイドが掲載されており、`urql`について学び始めるのに適したセクションです。
- **アーキテクチャ**は、`urql`がどのように機能し、何から構成されているかを説明し、`クライアント`とエクスチェンジの主要な側面をカバーします。
- **Advanced** は、一般的でない使用例をすべてカバーし、`urql` を使い始める際にすぐに必要でないガイドが含まれています。
- **Graphcache** は、`urql` の最も重要なアドオンで、クライアントに [「正規化キャッシング」 のサポート](https://formidable.com/open-source/urql/docs/graphcache/normalized-caching/)を追加し、より複雑なユースケース、よりスマートなキャッシュ、よりダイナミックなアプリケーションの機能を可能にするものです。
- **ショーケース**は、`urql` のユーザー、サードパーティーのパッケージ、チュートリアルやガイドのような役に立つリソースをリストアップすることを目的としています。
- **API** には、各パッケージのAPIに関する詳細なドキュメントが含まれています。ドキュメントは適宜それぞれにリンクしていますが、ユーティリティやパッケージの使い方が分からない場合は、ここに直接アクセスして特定のAPIの使い方を調べることができます。

ぜひ、`urql`を好きになってください。