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


