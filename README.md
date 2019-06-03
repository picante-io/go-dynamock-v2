# go-dynamock-v2

[![GoDoc](https://godoc.org/github.com/groovili/go-dynamock-v2?status.png)](https://godoc.org/github.com/groovili/go-dynamock-v2) 
[![Go Report Card](https://goreportcard.com/badge/github.com/groovili/go-dynamock-v2)](https://goreportcard.com/report/github.com/groovili/go-dynamock-v2) 
[![Build Status](https://api.travis-ci.org/groovili/go-dynamock-v2.svg?branch=master)](https://travis-ci.org/groovili/go-dynamock-v2)
[![Codecov](https://codecov.io/gh/groovili/go-dynamock-v2/branch/master/graphs/badge.svg?branch=master)](https://codecov.io/gh/groovili/go-dynamock-v2)

Amazon DynamoDB mock for unit testing, fully compatible with [SDK](https://github.com/aws/aws-sdk-go-v2).

Visit [GoDoc](https://godoc.org/github.com/groovili/go-dynamock-v2) for public API documentation.

Thanks to [gusaul](https://github.com/gusaul)  for the first version of package [go-dynamock](https://github.com/gusaul/go-dynamock).

## Requirements

- Go >= 1.11.x
- [AWS SDK GO V2](https://github.com/aws/aws-sdk-go-v2) >= v0.9.0


## Usage
To use mock you should depend in your code on [ClientAPI interface](https://github.com/aws/aws-sdk-go-v2/tree/master/service/dynamodb/dynamodbiface), instead of dependency on specific DynamoDB instance.

``` go
package main

import (
    "github.com/aws/aws-sdk-go-v2/service/dynamodb/dynamodbiface"
)

type Service struct {
    DynamoDB dynamodbiface.ClientAPI
}

func NewService (dynamo dynamodbiface.ClientAPI) *Service {
    return &Service{
        DynamoDB: dynamo,
    }
}
```

#### Function you want to test
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

#### Test
``` go
package examples

import (
	"strconv"
	"testing"

	dynamock "github.com/groovili/go-dynamock-v2"

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

## Currently Supported Functions
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

Feel free to open a pull request.

## License

The [MIT License](https://github.com/groovili/go-dynamock-v2/blob/master/LICENSE)
