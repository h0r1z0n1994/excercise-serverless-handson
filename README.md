# excercise-serverless-handson

## データ格納先のDynamoDBを設定

テーブル名：[姓]_[名]
プライマリキー：timestamp(数値)

# Lambda Function作成

2つのLambda関数をpython3.6で作ってみましょう！！

## dataGetFunc_[姓]_[名]　：　GETリクエスト受信時にDynamoDBからデータをScanして返す関数

```
import datetime
import boto3
from boto3.session import Session

def lambda_handler(event, context):
    region = "ap-northeast-1"
    session = Session(
        region_name=region
    )
    dynamodb = session.resource('dynamodb')

    table = dynamodb.Table('テーブル名')

    scan_response = table.scan()
    scan_response['Items'] = sorted(scan_response['Items'], key=lambda x:x['timestamp'], reverse=True) # scanしたデータをtimestampで降順にソートしたデータを返すようにしています。
    return scan_response
```

## dataPostFunc_[姓]_[名]　：　POSTリクエスト受信時にDynamoDBにデータをPutItemして、登録後にScanして返却する

```
import datetime
import boto3
from boto3.session import Session

def lambda_handler(event, context):
    region = "ap-northeast-1"
    session = Session(
        region_name=region
    )
    dynamodb = session.resource('dynamodb')

    table = dynamodb.Table('テーブル名')

    put_response = table.put_item(
        Item = {
                'timestamp': int(datetime.datetime.now().timestamp()),
                'content': event['content'],
        }
    )
    scan_response = table.scan()
    scan_response['Items'] = sorted(scan_response['Items'], key=lambda x:x['timestamp'], reverse=True)
    return scan_response
```

# API GatewayでAPI定義

## /v1/[姓][名]という階層でリソースを作成

## リソースに対して、GETとPOSTメソッドを作成

## 統合タイプをLambda関数を選択し、先程作成したLambda関数を紐付ける

## マッピングテンプレートを使ったHTML生成

GETメソッドの設定の中で、「統合レスポンス」の項目を選択します。
この中で「本文マッピングテンプレート」を設定します。

Lambda関数からJsonデータが来た場合、統合レスポンスが指定した形式に変換してレスポンスを返す設定をします。

Content-Typeの[application/json]を選択
エディタに下記を貼り付ける

```
#set($inputRoot = $input.path('$'))

<html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
        <script type="text/javascript">
        <!--javascriptのコード-->
        </script>
        <style type="text/css">
        <!--cssのコード-->
        </style>
    </head>
<body>
    #foreach($elem in $inputRoot['Items'])
    <div class="issue">
        "$elem.content"
        <div class="timestamp">
            "$elem.timestamp"
        </div>
    </div>
    #end
</body>
</html>
```

## メソッドレスポンスの設定

+ レスポンス本文のコンテンツタイプを「application/json」から「text/html」に変更する


## POSTデータを処理する設定

+ POSTメソッドの「統合リクエスト」を選択

+ [マッピングテンプレート]を選択

+ [Content-Type]に「application/x-www-form-urlencoded」を受信した時の受信データ処理の内容を記載します

```
{
#set( $tmpstr = $input.body )
#foreach( $keyandvaluestr in $tmpstr.split( '&' ) )
#set( $keyandvaluearray = $keyandvaluestr.split( '=' ) )
        "$keyandvaluearray[0]" : "$keyandvaluearray[1]"
#end
}
```

# S3のエンドポイントにアクセスしてフォームが動作することを確認する
