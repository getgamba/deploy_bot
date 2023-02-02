
# AWS Lambda Slack CircleCI Deploy Bot

A Slack bot for AWS Lambda that automate deployment with Circle CI.

## Install

### Slackの設定
1. Slashのワークスペース管理より、Appカスタムインテグレーションへ進み、Slash Commandsを追加します。
1. Slash Commandsのインテグレーションを追加し、トークンに表示された内容を取得します。

![Slack管理画面](https://user-images.githubusercontent.com/1058058/121304694-a3988c80-c937-11eb-978d-722643179505.png)

### AWS Lambdaの準備(gamba_deploy_bot)
このLambda関数は、AWS API Gatewayからコマンドを受け取って、別のLambda関数(gamba_deploy_runner)へリクエストを引き継ぎます。
Githubのプルリクエストの処理は若干時間がかかるため、1つのLambda関数内で一度に処理しようとするとタイムアウトが発生してしまいます。

1. AWS Lambdaに用意されているBluePrintに用意されている、`Slack-echo-command`を元に、lambda関数を作成します。
1. 以下のコマンドにより、`lambda_function.zip`を作成します
    ```
    $ ./deploy.sh
    ```
1. AWS Lambdaの関数コードに`lambda_function.zip`をアップロードします
1. ハンドラーを`index.slack_handler`に設定します
1. 以下の環境変数を登録します
    * `slackToken`: Slackの設定で表示されたトークン

### AWS Lambdaの準備(gamba_deploy_runner)
gamba_deploy_botからリクエストを引き継ぎ、Githubのプルリクエストの処理を実際に行うLambda関数です。

1. 上記で作成した`lambda_function.zip`をアップロードします
1. ハンドラーを`index.deploy_handler`に設定します
1. 以下の環境変数を登録します
    * `githubToken`: Githubのアクセストークン

### API Gatewayの設定
1.  `/gamba_deploy_bot - ANY - 統合リクエスト`を開き、マッピングテンプレートを追加します
    * Content-TYpe: `application/x-www-form-urlencoded`
    * テンプレート: `{ "body": $input.json("$") }`
2. APIをデプロイします
3. このとき表示されたURLをメモします

### Slackの設定(続き)
1. API Gatewayで取得したURLをSlash CommandsのURLに設定します
1. ユーザの名前には`deploy_bot`、アイコンには適切な画像を設定します

## Usage
### デプロイ
Slackのチャネルに対して、以下のコマンドを送ると、デプロイが始まります。

```
/deploy branch_name to environment
```
* `branch_name`: デプロイするブランチ名を指定します。省略時は`master`がはいります。
* `environment`: 以下のいずれかを指定します。
  * `production`: 本番環境にデプロイします
  * `staging`: ステージング環境にデプロイします
  * `app`: iOS/Androidのアプリのビルド＆AppStore/GooglePlayへのアップロード
  * `android`: Androidのアプリのビルド＆GooglePlayへのアップロード 
  * `ios`: iOSのアプリのビルド＆AppStoreへのアップロード 
  * `codepush`: CodePushへのJSコードのアップロード
  * `web`: `www.getgamba.com/m` 配下にベータ版Webクライアントのみデプロイします
* `branch_name`: デプロイするブランチ名を指定します。省略時は`master`がはいります。

### メンテナンス
Slackのチャネルに対して、以下のコマンドを送ることで、各種メンテナンスを行うことができます。

```
/deploy command
```
* `maintenance start`: gamba!をメンテナンスモードに切り替えます。これによって、APIリクエストはすべてステータスコード503を応答するようになります。
* `maintenance stop`: メンテナンスモードを解除します。APIリクエストは正常応答を返すようになります。
* `rollback production`: 直前のデプロイリリースの状態にロールバックします。

## 参考
* [2023/01/24 AWS Lambdaの入れ替え作業の録画](https://drive.google.com/file/d/1SLRfoHC91MkBPbz6xLZmNbt0-fjubIQh/view)

