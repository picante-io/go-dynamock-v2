[![GoDoc](https://godoc.org/github.com/groovili/go-dynamock-v2?status.png)](https://godoc.org/github.com/groovili/go-dynamock-v2) [![Go Report Card](https://goreportcard.com/badge/github.com/groovili/go-dynamock-v2)](https://goreportcard.com/report/github.com/groovili/go-dynamock-v2) [![Build Status](https://travis-ci.com/gusaul/go-dynamock.svg?branch=master)](https://travis-ci.com/gusaul/go-dynamock)

# go-dynamock-v2
Dynamo DB Mock based on AWS SDK for Go V2

## Examples Usage
Visit [godoc](https://godoc.org/github.com/groovili/go-dynamock-v2) for general examples and public API reference.

### DynamoDB configuration
First of all, change the dynamodb configuration to use the ***dynamodb interface***. see code below:
``` go
package main

import (
    "github.com/aws/aws-sdk-go-v2/aws"
    "github.com/aws/aws-sdk-go-v2/service/dynamodb"
    "github.com/aws/aws-sdk-go-v2/service/dynamodb/dynamodbiface"
)

type MyDynamo struct {
    DB dynamodbiface.DynamoDBAPI
}

var Dyna *MyDynamo

func ConfigureDynamoDB() {
    Dyna = new(MyDynamo)
    Dyna.DB, Mock = dynamock.New()
}
```
the purpose of code above is to make your dynamoDB object can be mocked by ***dynamock*** through the dynamodbiface.

### Something you may wanna test
``` go
package main

import (
    "context"
    "strconv"
    
    "github.com/aws/aws-sdk-go-v2/aws"
    "github.com/aws/aws-sdk-go-v2/service/dynamodb"
    "github.com/aws/aws-sdk-go-v2/service/dynamodb/dynamodbattribute"
)

func GetNameByID(ID int) (*string, error) {
	param := &dynamodb.GetItemInput{
		Key: map[string]dynamodb.AttributeValue{
			"id": {
				N: aws.String(strconv.Itoa(ID)),
			},
		},
		TableName: aws.String("employee"),
	}

	req := Fake.DB.GetItemRequest(param)
	if req.Error != nil {
		return nil, req.Error
	}

	var value *string
	output, err := req.Send(context.Background())
	if err != nil {
		return nil, err
	}

	if v, ok := output.Item["name"]; ok {
		err := dynamodbattribute.Unmarshal(&v, &value)
		if err != nil {
			return value, err
		}
	}

	return value, nil
}
```

### Test with DynaMock
``` go
package examples

import (
	"strconv"
	"testing"

	dynamock "github.com/gusaul/go-dynamock"

	"github.com/aws/aws-sdk-go-v2/aws"
	"github.com/aws/aws-sdk-go-v2/service/dynamodb"
)

func init() {
	Fake = new(FakeDynamo)
	Fake.DB, Mock = dynamock.New()
}

func TestGetItem(t *testing.T) {
	ID := 123
	expectKey := map[string]dynamodb.AttributeValue{
		"id": {
			N: aws.String(strconv.Itoa(ID)),
		},
	}

	expectedResult := "rick sanchez"
	result := dynamodb.GetItemResponse{
		GetItemOutput: &dynamodb.GetItemOutput{
			Item: map[string]dynamodb.AttributeValue{
				"id": {
					N: aws.String(strconv.Itoa(ID)),
				},
				"name": {
					S: aws.String(expectedResult),
				},
			},
		},
	}

	Mock.ExpectGetItem().Table("employee").WithKeys(expectKey).WillReturn(result)

	actualResult, err := GetNameByID(ID)
	if err != nil {
		t.Fatal(err)
	}

	if aws.StringValue(actualResult) != expectedResult {
		t.Fatalf("Fail: expected: %s, got: %s", expectedResult, aws.StringValue(actualResult))
	}
}
```

### Currently Supported Functions
``` go
GetItemRequest(*dynamodb.GetItemInput) dynamodb.GetItemRequest
PutItemRequest(*dynamodb.PutItemInput) dynamodb.PutItemRequest
UpdateItemRequest(*dynamodb.UpdateItemInput) dynamodb.UpdateItemRequest
DeleteItemRequest(*dynamodb.DeleteItemInput) dynamodb.DeleteItemRequest
BatchGetItemRequest(*dynamodb.BatchGetItemInput) dynamodb.BatchGetItemRequest
BatchWriteItemRequest(*dynamodb.BatchWriteItemInput) dynamodb.BatchWriteItemRequest
ScanRequest(*dynamodb.ScanInput) dynamodb.ScanRequest
QueryRequest(*dynamodb.QueryInput) dynamodb.QueryRequest
CreateTableRequest(*dynamodb.CreateTableInput) dynamodb.CreateTableRequest
DescribeTableRequest(*dynamodb.DescribeTableInput) dynamodb.DescribeTableRequest
WaitUntilTableExists(context.Context, *dynamodb.DescribeTableInput, ...aws.WaiterOption) error
```

## Contributions

Feel free to open a pull request. Note, if you wish to contribute an extension to public (exported methods or types) -
please open an issue before, to discuss whether these changes can be accepted.

## License

The [MIT License](https://github.com/groovili/go-dynamock-v2/blob/master/LICENSE)
