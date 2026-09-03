# Starlight Valkyrie

ダミーサイトです。

表示先は [StarValkyrie](https://gplayer2022.github.io/StarValkyrie/) です。

## プロジェクト作成手順

1. メニュー [ファイル] > [新規作成] > [プロジェクト] を選択
2. ウィンドウ [新しいプロジェクトの作成] で `Blazor WebAssembly スタンドアロン アプリ` を選択し、ボタン [次へ] を押下
3. ウィンドウ [新しいプロジェクトを構成します] でプロジェクト名・場所・ソリューション名を設定し、ボタン [次へ] を押下
4. ウィンドウ [追加情報] で、下記のように設定し、ボタン [作成] を押下
    - フレームワーク: `.NET 10.0`
    - 認証の種類: `なし`
    - HTTPS 用の構成: ☑
    - プログレッシブ Web アプリケーション: □
    - サンプルページを含める: □
    - 最上位レベルのステートメントを使用しない: ☑
    - .NET Aspire オーケストレーションへの傘下: □



## 設定手順

1. `.github/workflows/gh-pages.yml` を設定する
    - 必ずソリューションのルートからの相対パスで指定すること！
    - 1 字たりとても間違えないこと！
    - ただし、 `.yml` ファイル名については任意でよい
2. GitHub リポジトリで `Settings` > `Code` でブランチ切り替え用のドロップダウンから [View all branches] を選択する
3. GitHub リポジトリの、`Branches` でボタン [New branch] をクリックし、 `gh-pages` ブランチを作成する
4. GitHub リポジトリで `Settings` > `Pages` > `Branch` を `gh-pages` に設定する
5. GitHub リポジトリで `Settings` > `Actions` > `General` > `Workflow permissions` を `Read and write permissions` に設定する
6. Visual Studio でソリューションをコミットおよびプッシュする


## `.razor` ファイルの説明

ファイル名がそのままクラス名になるため、
クラス名を `Weather` にするためにファイル名を `Weather.razor` にする必要がある。
親クラスは `ComponentBase` が自動的に継承される。

```C#
public partial class Weather : ComponentBase
{
}
```

## ライフサイクル

メソッド | 呼ばれるタイミング | 主な用途
---------|--------------------|-------------
SetParametersAsync | 最初、パラメータ受け取り時 | 通常は触らない
OnInitialized(Async) | 生成時に1回だけ | 初期データ取得
OnParametersSet(Async) | パラメータ変更ごと | パラメータに基づく計算
ShouldRender | 再レンダリング前 | 再描画の抑制
OnAfterRender(Async) | 描画完了後 | JS Interop、DOM 操作
Dispose(Async) | 破棄時 | リソース解放

1. SetParametersAsync
    - 親から渡された [Parameter] をコンポーネントにセットする、一番最初に呼ばれる処理
    - 基本的にオーバーライドすることは少ない（パラメータ受け取りの挙動そのものをカスタマイズしたい特殊なケースのみ）
2. OnInitialized / OnInitializedAsync
    - コンポーネントが最初に生成されたときだけ、1回だけ呼ばれる
    - 用途: 初期データの取得、サービスからの初期値の設定など
    - 非同期版（ OnInitializedAsync ）は、 API からデータを取ってくるような処理に使う
3. OnParametersSet / OnParametersSetAsync
    - 親から渡された [Parameter] が変更されるたびに呼ばれる
    - 用途: パラメータの値を使って何らかの計算・加工をしたいとき
    - OnInitialized は 1 回だけ、 OnParametersSet は**パラメータが変わるたびに何度でも**呼ばれる
4. ShouldRender
    - 再レンダリング前に呼ばれる
    - **本当に再レンダリングする必要があるか**を bool で判定し、 false を返すと再レンダリングをスキップできる
    - パフォーマンス最適化のためのフック。頻繁な更新で不要な再描画を抑えたいときに使う
    - デフォルトは true で、通常はオーバーライドすることは少ない
5. OnAfterRender / OnAfterRenderAsync
    - 描画が完了した後に呼ばれる
    - 用途: JavaScript 相互運用（ JS Interop ）、DOM 操作（ DOM 要素へのフォーカス設定など）など、**実際に画面に描画された後でないとできない処理**に利用する
    - 非同期版（ OnAfterRenderAsync ）は、 JS Interop で非同期処理を行う場合に使う
6. Dispose / DisposeAsync
    - コンポーネントが破棄される（画面から消える、ページ遷移するなど）ときに呼ばれる
    - 用途: イベントハンドラの解除、タイマーの停止、 JS Interop で確保したリソースの解放など
    - 使うには `IDisposable` または `IAsyncDisposable` を実装する必要がある

# `.yml` ファイルの設定

```yml
name: Deploy Blazor WASM to GitHub Pages

on:
  push:
    branches: [ "master" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v4

    - name: Setup .NET
      uses: actions/setup-dotnet@v4
      with:
        dotnet-version: 10.0.x

    - name: Publish
      run: dotnet publish -c Release -o release

    - name: Fix base href
      run: |
        sed -i 's|<base href="/" />|<base href="/StarValkyrie/" />|g' release/wwwroot/index.html
        cp release/wwwroot/index.html release/wwwroot/404.html
        touch release/wwwroot/.nojekyll

    - name: Deploy
      uses: peaceiris/actions-gh-pages@v4
      with:
        github_token: ${{ secrets.GITHUB_TOKEN }}
        publish_dir: release/wwwroot
```

# ローカルで発行する

```cmd
dotnet publish -c Release -o release
```

- `publish` : 発行する
    - 他のサブコマンドの例
        - `build` : ビルドする
        - `run` : 実行する
- `-c Release` : Debug ではないく Release ビルドで発行する
- `-o release` : 発行先のフォルダを `release` にする


