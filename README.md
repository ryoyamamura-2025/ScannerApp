# ScannerApp
UWP App の開発  

## 環境構築 (最初のサンプルアプリ)
0. Visual Studio で UWP 開発用のリソースをインストールする  
   - デスクトップアプリ開発にて UWP にチェックを入れる必要がある（デフォルトではチェックが外されている）
2. Windows の開発者モードをオンにする  
   - 設定 > システム > 開発者向けから設定
3. Visual Studio で「新しいプロジェクトの作成」  
   以下の設定で検索し空のUWPアプリ選択。適当なプロジェクト名をつけて新規作成する
   ```
   言語: C#
   プラットフォーム: Windows
   プロジェクトの種類: UWP
   ```
4. 空のアプリの動作確認  
   - 画面上部の緑色のボタン▶️を押下（Debug, x86, App名になっていることを確認）
   - 白い画面が表示されればOK

## WebView2と連携したスキャンアプリ
### 事前準備
0. 適当な名前でプロジェクトを作成
1. `Package.appxmanifest` を開き、「機能」タブで『Webカメラ』、『インターネット』、『プライベートネットワーク』にチェックを入れる
2. [MS Learn](https://learn.microsoft.com/ja-jp/microsoft-edge/webview2/get-started/winui2) にしたがって、WebView2用のライブラリをインストール
3. 使用可能なスキャナを用意する（Windowsのプリンターとスキャナに表示されていればOKのはず）

### アプリの作成概要
1. `MainPage.xaml` は基本的に [MS Learn](https://learn.microsoft.com/ja-jp/microsoft-edge/webview2/get-started/winui2) に沿って WebView2 を読み込むタグと URL を追記  
   ```
   <Grid>
      <controls:WebView2 Grid.Column="1" x:Name="MyWebView" Source="https:/hogehoge.hoge"/>
   </Grid>
   ```
2. `MainPage.xaml.cs` では、WebView2ｎビューで取得した JavaScript の関数を実行する（以下は画像データを渡して `displayScannedImage` という JavaScript の関数を実行する例）  
   ```
    // 1. ファイルをバイト配列に読み込む
    IBuffer buffer = await FileIO.ReadBufferAsync(imageFile);
    byte[] bytes = buffer.ToArray();

    // 2. バイト配列をBase64文字列に変換
    string base64String = Convert.ToBase64String(bytes);

    // 3. JavaScriptで使えるようにデータURI形式の文字列を作成
    //    'data:image/jpeg;base64,' の部分はファイルの形式に合わせて変更可能
    string dataUriString = $"data:image/jpeg;base64,{base64String}";

    // 4. 呼び出すJavaScriptコードを生成
    //    文字列はシングルクォートで囲む
    string script = $"displayScannedImage('{dataUriString}')";

    // 5. WebView2上でJavaScriptを実行
    await MyWebView.CoreWebView2.ExecuteScriptAsync(script);

    System.Diagnostics.Debug.WriteLine("WebView2に画像を送信しました。");
   ```
3. JavaScript 内で適切なエンドポイントを呼び出して API を叩けばサーバー上でのスキャンデータ等の処理をサーバーに渡すことが可能
