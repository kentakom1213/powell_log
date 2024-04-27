+++
title = "Rust + Axum + Shuttle でAPIサーバ構築"
date = "2023-03-02 11:43:00"
[taxonomies]
tags = ["Rust", "Web開発", "プログラミング"]
+++

Rust で API サーバを作成し、デプロイしてみたので共有します。

<!-- more -->

## 概要

{% admonition(type="note", title="開発環境") %}

- 開発言語：[Rust](https://www.rust-lang.org/ja)
- フレームワーク：[Axum](https://github.com/tokio-rs/axum)
- デプロイ：[Shuttle](https://www.shuttle.rs/)
- 開発環境
  - MacbookAir（M1 チップ）（OS: Ventura 13.0）
  - Rust（cargo 1.67.1 (8ecd4f20a 2023-01-10)）

{% end %}

---

web サービスのフロントエンドを開発するにあたり、ただデータを返してくれるだけのモックサーバが必要になったため、勉強も兼ねて Rust で API サーバを建ててみることにしました。

## API の設計

pdf ファイルを管理するアプリなので、pdf の一覧と詳細を取得できれば良いです。

- `/list`：pdf のリストを返します

```shell
$ curl https://{デプロイ先のurl}/list
```

```json
[
  {
    "file_id": 0,
    "name": "微分積分学1"
  },
  {
    "file_id": 1,
    "name": "線形代数学"
  },
  {
    "file_id": 2,
    "name": "システム数学及び演習1"
  },
  {
    "file_id": 3,
    "name": "シミュレーション"
  },
  {
    "file_id": 4,
    "name": "物理基礎2"
  }
]
```

- `/detail/:file_id`：与えられた id を持つ pdf ファイルの詳細情報を返します

```shell
$ curl https://nu-wiki-mock-pdf-detail.shuttleapp.rs/detail/0
```

```json
{
  "file_id": 0,
  "name": "微分積分学1",
  "url": "https://www.nagoya-u.ac.jp/academics/upload_images/5b2f064da9816cd6192d25c3a6d262ae_1.pdf"
}
```

## 実装

### サーバの設計

リクエストを処理するフローは以下の通り。（下図参照）

1. json ファイルを配置
2. データをデータベースに保存
3. オブジェクトを json として返す

![](/images/rust-api-server/api-diagram.svg)

ここからは、各段階を詳しく説明します。

---

### 1.json ファイルを配置

#### ファイル構成

ファイル構成はこんな感じにしました。

```
nu-wiki-mock-pdf-detail
  ├ src
  │  ├ lib.rs           # ルーティングを行う
  │  ├ pdf_list.rs      # pdf_listをjsonから読み込む
  │  └ pdf_detail.rs    # pdf_detailをjsonから読み込む
  ├ static
  │  ├ pdf_list.json    # pdf_listの中身
  │  └ pdf_detail.json  # pdf_detailの中身
  └ Cargo.toml
```

#### static ディレクトリのマウント

json ファイルを`/static`においただけでは存在が認識されないため、shuttle のライブラリを使ってマウントします。

```rust
use shuttle_static_folder::StaticFolder;

#[shuttle_service::main]
async fn axum(
    #[StaticFolder(folder = "static")] static_folder: PathBuf,
) -> shuttle_service::ShuttleAxum {
    // ...

    // ルーティングを行う関数（実質main関数）
    // この中では `static_folder` が `/static` として使える
}
```

### 2.データをデータベースに保存

#### json の読み込み

Rust で json の操作を行うライブラリ`serde`を利用しました。json と同じ形式の構造体を作ることで、簡単に json⇄Rust のオブジェクト ができるので便利です。

#### データベースに追加

データベースと言ってはいますが、SQL などのリレーショナルデータベースではなく単なるグローバル変数です。
Rust ではただグローバルな領域に変数を置いただけではグローバル変数にならないため、**Arc**という型に包みます。
Arc は[スマートポインタ](https://doc.rust-jp.rs/book-ja/ch15-00-smart-pointers.html)であり、
自分が参照された回数をカウントしながら安全な形で同一のデータにアクセスする仕組みです。
また、Arc はデータへの参照を保存するだけで、データの読み込みや書き込みには対応していないため、
**RwLock**という型で包むことで、複数のスレッドからも安全にデータの読み書きをすることができます。

```
+------- Arc --------+
|                    | ← データを保存する箱
|  +--- RwLock ---+  |
|  |              | ← 安全に読み書きできるようにする箱
|  |   [ Data ]   |  |
|  |              |  |
|  +--------------+  |
|                    |
+--------------------+
```

詳細は以下の記事を参照してください。

- [Rust の`Arc`を読む](https://qiita.com/qnighy/items/4bbbb20e71cf4ae527b9)

### 3.オブジェクトを json として返す

#### ルーティング

ルーティングは`lib.rs`の`axum`関数内で行います。
`axum::Router`オブジェクトに、メソッドチェーンの形で`route`関数を追加していけばいいだけなのでかなり簡単です。

```rust
use axum::Router;

#[shuttle_service::main]
async fn axum() -> shuttle_service::ShuttleAxum {
    // ...

    // appのルーティングを定義
    let app = Router::new()
        .route("/list", get(get_list))         // "/list" にリクエストが来たとき
        .with_state(db_list)                   //   → db_listを用いる
        .route("/detail/:id", get(get_detail)) // "/detail/:id" にリクエストが来たとき
        .with_state(db_detail);                //   → db_detailを用いる

    // ...
}
```

#### リクエストの処理

上のコードの 9 行目では`/list`への GET リクエストの時に`get_list`関数を呼び出すことを定義しています。
この`get_list`関数は下のように定義されています。
引数は上のコードの 10 行目の`with_state`メソッドで付加された`db_list`が`State`に包まれた状態で渡されます。
全て説明すると大変なので省略しますが、いくつものレイヤーで包まれたデータをメソッドチェーンで剥いて行って、
最終的に`Json`オブジェクトとして返すという感じです。

```rust
// dbの定義
type DbPdfList = Arc<RwLock<Vec<PdfOverview>>>;

/// ## get_list
/// pdfの一覧を返す
async fn get_list(State(db): State<DbPdfList>) -> Json<Vec<PdfOverview>> {
    let list = db // DbPdfList = Arc<RwLock<Vec<PdfOverview>>>
      .read()     // → Result<RwLockReadGuard<Vec<PdfOverview>>>
      .unwrap()   // → RwLockReadGuard<Vec<PdfOverview>>
      .deref()    // → &Vec<PdfOverview>
      .clone();   // → Vec<PdfOverview>

    Json(list)
}
```

## デプロイ

[Shuttle](https://www.shuttle.rs/)という、Rust 製の web アプリを無料でデプロイすることができるサービスを利用しました。
サーバ側の設定などは必要なく、コマンドのみでデプロイできるのは非常に快適でした。

```shell コマンドの概要
$ cargo shuttle init         # shuttleの初期化
$ cargo shuttle project new  # プロジェクトの作成
$ cargo shuttle deploy       # デプロイ🚀
```

今回は試しませんでしたが、データベースへの接続なども無料でできるみたいです 👍

詳細は公式のドキュメントや[example](https://github.com/shuttle-hq/examples)を参照してみてください。

## 終わりに

今回初めて、Rust で web アプリをデプロイするところまで挑戦してみました。
よく言われる通り、Rust でのアプリ開発はかなり難しいですが
厳格な型による安心感と保守のしやすさというメリットは圧倒的だなと感じました。

また、shuttle はまだ発展途上だなと思うような場面にも何度か遭遇しましたが、
開発コミュニティの Discord がかなり活発で、質問を投げるとすぐに返してくれたことも
非常にありがたかったです。

この記事を読んで Rust による web 開発に興味を持ってくださった皆さん、
一緒に Rust を勉強してみませんか？
