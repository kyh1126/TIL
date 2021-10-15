# DynamoDB with curl

## 테스트 환경

- 레퍼런스: [https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html](https://docs.aws.amazon.com/ko_kr/amazondynamodb/latest/developerguide/DynamoDBLocal.DownloadingAndRunning.html)

```bash
docker pull amazon/dynamodb-local
docker run -p 8000:8000 amazon/dynamodb-local -jar DynamoDBLocal.jar -inMemory -sharedDb
```

- 포트 8000번에 로컬 DynamoDB를 띄운다.
- `-inMemory` 옵션을 주면 DB 를 종료할 때 DB 의 모든 데이터가 사라지고,
- `-sharedDb` 옵션을 주면 request 의 region 이 다르더라도 같은 공간을 사용하게 된다.


## CURL 명령어

### 1. create-table

---

- 레퍼런스: [https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_CreateTable.html](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_CreateTable.html)

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Content-Type: application/json' \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'X-Amz-Target: DynamoDB_20120810.CreateTable' \
  -d '{
    "AttributeDefinitions": [
        {
            "AttributeName": "id",
            "AttributeType": "S"
        },
        {
            "AttributeName": "invoiceId",
            "AttributeType": "N"
        }
    ],
    "TableName": "InvoiceExcel",
    "KeySchema": [
        {
            "AttributeName": "id",
            "KeyType": "HASH"
        }
    ],
    "GlobalSecondaryIndexes": [
        {
            "IndexName": "InvoiceIdIndex",
            "KeySchema": [
                {
                    "AttributeName": "invoiceId",
                    "KeyType": "HASH"
                }
            ],
            "Projection": {
                "ProjectionType": "ALL"
            },
            "ProvisionedThroughput": {
                "ReadCapacityUnits": 1,
                "WriteCapacityUnits": 1
            }
        }
    ],
    "ProvisionedThroughput": {
        "ReadCapacityUnits": 1,
        "WriteCapacityUnits": 1
    }
}'
```

### 2. update-table

---

- 레퍼런스: [https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateTable.html](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_UpdateTable.html)

### 3. describe-table

---

- 레퍼런스: [https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_DescribeTable.html)

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'Content-Type: application/json' \
  -H 'X-Amz-Target: DynamoDB_20120810.DescribeTable' \
  -d '{
    "TableName":"InvoiceExcel"
}'
```

### 4. put-item

---

- 레퍼런스: [https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PutItem.html](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_PutItem.html)

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'Content-Type: application/json' \
  -H 'X-Amz-Target: DynamoDB_20120810.PutItem' \
  -d '{
    "TableName":"InvoiceExcel",
    "Item":{"id": {"S": "3ea8e671-1e64-4cde-bd78-5980049a772b"}, "invoiceId": {"N": "1"}, "snapshot_json": {"S": "{\"invoiceContents\": {\"bodies\": [{\"row\": [{\"key\": \"serviceContent\", \"order\": 0, \"value\": null, \"keyName\": \"서비스내역\", \"children\": [{\"key\": \"level1\", \"order\": 0, \"value\": \"운영\", \"keyName\": \"대분류\", \"children\": null}, {\"key\": \"level2\", \"order\": 1, \"value\": null, \"keyName\": \"중분류\", \"children\": null}, {\"key\": \"level3\", \"order\": 2, \"value\": null, \"keyName\": \"소분류\", \"children\": null}]}, {\"key\": \"count\", \"order\": 1, \"value\": \"90000\", \"keyName\": \"수량\", \"children\": null}, {\"key\": \"unit\", \"order\": 2, \"value\": \"퍼센트\", \"keyName\": \"단위\", \"children\": null}, {\"key\": \"unitPrice\", \"order\": 3, \"value\": \"5\", \"keyName\": \"단가\", \"children\": null}, {\"key\": \"price\", \"order\": 4, \"value\": \"3004500\", \"keyName\": \"금액\", \"children\": null}, {\"key\": \"remark\", \"order\": 5, \"value\": \"okok\", \"keyName\": \"비고\", \"children\": null}], \"rowId\": 0}, {\"row\": [{\"key\": \"serviceContent\", \"order\": 0, \"value\": null, \"keyName\": \"서비스내역\", \"children\": [{\"key\": \"level1\", \"order\": 0, \"value\": \"보관\", \"keyName\": \"대분류\", \"children\": null}, {\"key\": \"level2\", \"order\": 1, \"value\": \"상온\", \"keyName\": \"중분류\", \"children\": null}, {\"key\": \"level3\", \"order\": 2, \"value\": \"PLT\", \"keyName\": \"소분류\", \"children\": null}]}, {\"key\": \"count\", \"order\": 1, \"value\": \"6.0\", \"keyName\": \"수량\", \"children\": null}, {\"key\": \"unit\", \"order\": 2, \"value\": \"EA\", \"keyName\": \"단위\", \"children\": null}, {\"key\": \"unitPrice\", \"order\": 3, \"value\": \"100\", \"keyName\": \"단가\", \"children\": null}, {\"key\": \"price\", \"order\": 4, \"value\": \"600.0\", \"keyName\": \"금액\", \"children\": null}, {\"key\": \"remark\", \"order\": 5, \"value\": \"test1\", \"keyName\": \"비고\", \"children\": null}], \"rowId\": 1}, {\"row\": [{\"key\": \"serviceContent\", \"order\": 0, \"value\": null, \"keyName\": \"서비스내역\", \"children\": [{\"key\": \"level1\", \"order\": 0, \"value\": \"보관\", \"keyName\": \"대분류\", \"children\": null}, {\"key\": \"level2\", \"order\": 1, \"value\": \"저온\", \"keyName\": \"중분류\", \"children\": null}, {\"key\": \"level3\", \"order\": 2, \"value\": \"PLT\", \"keyName\": \"소분류\", \"children\": null}]}, {\"key\": \"count\", \"order\": 1, \"value\": \"8.0\", \"keyName\": \"수량\", \"children\": null}, {\"key\": \"unit\", \"order\": 2, \"value\": \"EA\", \"keyName\": \"단위\", \"children\": null}, {\"key\": \"unitPrice\", \"order\": 3, \"value\": \"200\", \"keyName\": \"단가\", \"children\": null}, {\"key\": \"price\", \"order\": 4, \"value\": \"1600.0\", \"keyName\": \"금액\", \"children\": null}, {\"key\": \"remark\", \"order\": 5, \"value\": \"test2\", \"keyName\": \"비고\", \"children\": null}], \"rowId\": 2}], \"footers\": [{\"key\": \"chargeSurtaxExcluded\", \"order\": 0, \"value\": \"1980.0\", \"keyName\": \"요금 (부가세 제외)\", \"children\": null}, {\"key\": \"surtax\", \"order\": 1, \"value\": \"220.0\", \"keyName\": \"부가세\", \"children\": null}, {\"key\": \"chargeSurtaxIncluded\", \"order\": 2, \"value\": \"2200.0\", \"keyName\": \"요금 (부가세 포함)\", \"children\": null}], \"headers\": [{\"key\": \"serviceContent\", \"order\": 0, \"value\": null, \"keyName\": \"서비스내역\", \"children\": [{\"key\": \"level1\", \"order\": 0, \"value\": null, \"keyName\": \"대분류\", \"children\": null}, {\"key\": \"level2\", \"order\": 1, \"value\": null, \"keyName\": \"중분류\", \"children\": null}, {\"key\": \"level3\", \"order\": 2, \"value\": null, \"keyName\": \"소분류\", \"children\": null}]}, {\"key\": \"count\", \"order\": 1, \"value\": null, \"keyName\": \"수량\", \"children\": null}, {\"key\": \"unit\", \"order\": 2, \"value\": null, \"keyName\": \"단위\", \"children\": null}, {\"key\": \"unitPrice\", \"order\": 3, \"value\": null, \"keyName\": \"단가\", \"children\": null}, {\"key\": \"price\", \"order\": 4, \"value\": null, \"keyName\": \"금액\", \"children\": null}, {\"key\": \"remark\", \"order\": 5, \"value\": null, \"keyName\": \"비고\", \"children\": null}]}, \"footerComponents\": [{\"key\": \"footer_0\", \"order\": 0, \"value\": null, \"keyName\": \"아래와 같이 물류보관 및 대행비용에 대한 내역서를 제출합니다.\", \"children\": null}, {\"key\": \"footer_1\", \"order\": 1, \"value\": \"2021-08-16\", \"keyName\": \"청구일자\", \"children\": null}, {\"key\": \"footer_2\", \"order\": 2, \"value\": null, \"keyName\": \"스마트푸드네트웍스\n사업자등록번호 719 - 87 - 01744 / 대표자 조성수\n서울시 강남구 논현로 26길, 18-4, 1층 (도곡동)\n010-3323-9694\nsunghyun.park@smartfoodnet.com\", \"children\": null}], \"headerComponents\": [{\"key\": \"receiver\", \"order\": 0, \"value\": \"(주)테스트\", \"keyName\": \"수신자\", \"children\": null}, {\"key\": \"companyRegistrationNumber\", \"order\": 1, \"value\": \"604-81-15788\", \"keyName\": \"사업자등록번호\", \"children\": null}, {\"key\": \"representative\", \"order\": 2, \"value\": \"테스트\", \"keyName\": \"대표자\", \"children\": null}, {\"key\": \"address\", \"order\": 3, \"value\": \"서울특별시 서초구 서초동\", \"keyName\": \"주소\", \"children\": null}, {\"key\": \"settlementDate\", \"order\": 4, \"value\": \"2021-08-01 ~ 2021-08-02\", \"keyName\": \"정산기간\", \"children\": null}, {\"key\": \"accountNo\", \"order\": 5, \"value\": \"(국민) 519701-01-348547  / 예금주 : 스마트푸드네트웍스 주식회사\", \"keyName\": \"계좌번호\", \"children\": null}]}"}}
}'
```

### 5. get-item

---

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'Content-Type: application/json' \
  -H 'X-Amz-Target: DynamoDB_20120810.GetItem' \
  -d '{
    "TableName": "InvoiceExcel",
    "Key": {
        "id": {"S": "3ea8e671-1e64-4cde-bd78-5980049a772b"}
    }
}'
```

- `--consistent-read`: strongly consistent reads 옵션
    
    ```bash
    aws dynamodb get-item --consistent-read \
    		--table-name Music \
    		--key '{ "Artist": {"S": "Acme Band"}, "SongTitle": {"S": "Happy Day"}}'
    ```
    

### 6. Global secondary index 에서 query

---

- 레퍼런스: [https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html](https://docs.aws.amazon.com/amazondynamodb/latest/APIReference/API_Query.html)

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'Content-Type: application/json' \
  -H 'X-Amz-Target: DynamoDB_20120810.Query' \
  -d '{
    "TableName": "InvoiceExcel",
    "IndexName": "InvoiceIdIndex",
    "Limit": 1,
    "KeyConditionExpression": "invoiceId = :invoiceId",
    "ExpressionAttributeValues": {
        ":invoiceId":{"N":"1"}
    },
    "Key": {
        "id": {"S": "3ea8e671-1e64-4cde-bd78-5980049a772b"}
    }
}'
```

### 7. delete-table

---

```bash
curl -X POST \
  http://localhost:8000/ \
  -H 'Authorization: AWS4-HMAC-SHA256 Credential=key1/20210830/ap-northeast-2' \
  -H 'Content-Type: application/json' \
  -H 'X-Amz-Target: DynamoDB_20120810.DeleteTable' \
  -d '{
    "TableName":"InvoiceExcel"
}'
```

- 참고
    - [https://techblog.woowahan.com/2633/](https://techblog.woowahan.com/2633/)


- [Notion link](https://jennyuni.notion.site/DynamoDB-with-curl-880e13f2815b4364bfb34ab6f0cf8b0a)
