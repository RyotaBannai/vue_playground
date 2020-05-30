### SSR
- サーバサイドレンダリング (SSR) とは? Vue.js は`クライアントサイドアプリケーション`を構築するためのフレームワーク。通常では、Vue コンポーネントは`ブラウザで DOM を生成し操作がされる`。しかし、`同じ Vue コンポーネントを`サーバ上の HTML 文字列に描画し、`ブラウザに直接送信し、最終的に静的なマークアップとしてクライアント上の完全なインタラクティブアプリケーション`に "`ハイドレート (hydrate)`" することもできる。`サーバで描画された Vue.js のアプリケーション`は、`アプリケーションのコードの大部分が、サーバとクライアントの両方で実行される`という意味で、"`アイソモルフィック (isomorphic)`" や "`ユニバーサル (universal)`" と見なすことができる。
### なぜSSR?
- 従来の SPA (シングルページアプリケーション) と比べて、SSR の利点は主に次の点にある。
1. 検索エンジンのクローラが完全に描画されたページを直接解析するため、SEO が向上: 現在のところ、Google と Bing は`同期的` JavaScript アプリケーションのインデックスを作成できる。`同期がキーワード`。アプリケーションが読み込み中にスピナが表示され、`Ajax 経由でコンテンツを取得する場合`、`クローラーは非同期結果が完了するまで待たない`。つまり、`SEO が重要なページで非同期にコンテンツを取得する場合は、SSR が必要な場合がある`
2.　特に`インターネットの遅さ`や`遅いデバイス`では、コンテンツの再生時間が短縮される`。サーバで描画されたマークアップは、すべての JavaScript がダウンロードされて表示されるまで待つ必要がない`ので、ユーザは完全に描画されたページをすぐに見ることができます。これにより、一般的にユーザーエクスペリエンスが向上し、コンテンツの所要時間が直接コンバージョン率に関連付けられているアプリケーションにとっては重要になる
- トレードオフ
1. 開発上の制約。`ブラウザ固有のコードは、特定のライフサイクルフック内でのみでしか使用できない`。一部の外部ライブラリは、サーバで描画されたアプリケーションで実行できるように特別な処理が必要な場合がある。
2.  サーバ側の負荷が増える。 Node.js の完全なアプリケーションを描画することは、静的ファイルを提供するだけでなく、CPU を多用することになるので、トラフィックが多いことが見込まれる場合は、対応するサーバの負荷に備え、キャッシュの対策を行う。
### プリレンダリング
- もし、幾つかのマーケティングのページの SEO を向上させるためだけに SSR を調べているとしたら (たとえば /, /about, /contact など)、代わりに プリレンダリング (事前描画) を使用することを検討する方が良い。webpack を使用している場合、prerender-spa-plugin を使用することで簡単にプリレンダリングを実装することができる。
#### vue-hackernews-2.0のコードを読めばいい感じになるhttps://github.com/vuejs/vue-hackernews-2.0/tree/master/src
- `renderer` のページテンプレートに`<!--vue-ssr-outlet-->` コメントを必ずいれる。これはサーバーがアプリケーションのマークアップ（vueコンポネント、またはページごとのコンテンツ）が注入される場所。
- `context オブジェクト`も Vue アプリインスタンスと共有することができ、コンポーネントがテンプレート展開のために`データを動的に追加する`ことができる
#### 
- *.vue コンポーネントを使用する際の、`重要な CSS の自動注入`
- `clientManifest` を使用する際の、`アセットリンクとリソースヒントの自動注入`
- クライアントサイドハイドレーションのために Vuex の状態を埋め込む際に `XSS 防止の自動注入`
### サーバサイドの描画を行う上で、知っておく必要がある重要な項目
- 複数のリクエストにまたがった状態の汚染がないよう`各リクエストは新しく独立したアプリケーションのインスタンスが必要`
- `サーバー上では、データがリアクティブである必要はない`ので、デフォルトで無効になっています。データをリアクティブにしないことで、`データをリアクティブなオブジェクトに変換する際のパフォーマンスコストを無視できる`
- コンポーネントのライフサイクルフック: 動的な更新がないので、ライフサイクルフックのうち、`beforeCreate` と `created` **のみが SSR 中に呼び出される**。`beforeMount` や `mounted` などの`他のコンポーネントサイクルフック`は、**クライアントでのみ実行される**
- `beforeCreate` と `created` において、例えば `setInterval` でタイマーを設定するような、`グローバルな副作用を引き起こすコード`避けるべき。また他に、クライアントサイドのコードに限り、タイマーを設定してから `beforeDestroy` または `destroyed` 設定することができます。SSR 中に破棄フックは呼び出されないため、タイマーは実行されずに設定だけのコードが残ります。これを回避するために、代わりに`副作用コード`を `beforeMount` または `mounted` に移動してください。
### プラットフォーム固有の API にアクセス
- `ユニバーサルコード`では、プラットフォーム固有の API へのアクセスは想定されていないので、`window` や `document` `といったブラウザ環境のグローバル変数（ブラウザ API）を直接使用すると、Node.js ではエラーが発生します`
- サーバーとクライアントでコードを共有するものの、タスクが使用する API がプラットフォームによって異なる場合は、`プラットフォーム固有の実装を ユニバーサル な API の内部でラップする`か、それを行う`ライブラリを使用する`(例えば、axiosを使う（httpクライアント）)
- サードパーティライブラリが**ユニバーサルに使用することを考慮していない場合**、それをサーバサイドによって描画されるアプリケーションに統合することは難しいので注意
- `カスタムディレクティブ`: ほとんどの カスタムディレクティブ は`直接 DOM を操作するため、SSR 中にエラーが発生`します。これを回避するには、2つの方法がある。
1. 抽象化の仕組みとしてコンポーネントを使用し、カスタムディレクティブの`代わりに仮想 DOM レベル（例えば、render 関数を使用すること)で実装`する
2. コンポーネントに簡単に置き換えができないカスタムディレクティブの場合、サーバーレンダラを生成する際の `directives オプションを使用`して、そのオプションの "サーバーサイドのバージョン" を用意する