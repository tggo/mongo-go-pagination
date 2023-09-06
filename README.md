# Golang Mongo Pagination For Package mongo-go-driver
![Workflow](https://github.com/gobeam/mongo-go-pagination/actions/workflows/ci.yml/badge.svg) [![Build][Build-Status-Image]][Build-Status-Url] [![Go Report Card](https://goreportcard.com/badge/github.com/gobeam/mongo-go-pagination?branch=master&kill_cache=1)](https://goreportcard.com/report/github.com/gobeam/mongo-go-pagination) [![GoDoc][godoc-image]][godoc-url]
[![Coverage Status](https://coveralls.io/repos/github/gobeam/mongo-go-pagination/badge.png?branch=master)](https://coveralls.io/github/gobeam/mongo-go-pagination?branch=master)

For all your simple query to aggregation pipeline this is simple and easy to use Pagination driver with information like Total, Page, PerPage, Prev, Next, TotalPage and your actual mongo result. View examples from [here](https://github.com/gobeam/mongo-go-pagination/tree/master/example)

:speaker: :speaker: 
***For normal queries new feature have been added to directly pass struct and decode data without manual unmarshalling later. Only normal queries support this feature for now. Sort chaining is also added as new feature***

Example api response of Normal Query [click here](https://mongo-go-pagination.herokuapp.com/normal-pagination?page=1&limit=10).<br>
Example api response of Aggregate Query [click here](https://mongo-go-pagination.herokuapp.com/aggregate-pagination?page=1&limit=10).<br>
View code used in this example from [here](https://github.com/gobeam/mongo-go-pagination/tree/master/example)

## Install

``` bash
$ go get -u -v github.com/tggo/mongo-go-pagination
```

or with dep

``` bash
$ dep ensure -add github.com/tggo/mongo-go-pagination
```


## For Aggregation Pipelines Query

``` go
package main

import (
	"context"
	"fmt"
	. "github.com/gobeam/mongo-go-pagination"
	"go.mongodb.org/mongo-driver/bson"
	"go.mongodb.org/mongo-driver/bson/primitive"
	"go.mongodb.org/mongo-driver/mongo"
	"go.mongodb.org/mongo-driver/mongo/options"
	"time"
)

type Product struct {
	Id       primitive.ObjectID `json:"_id" bson:"_id"`
	Name     string             `json:"name" bson:"name"`
	Quantity float64            `json:"qty" bson:"qty"`
	Price    float64            `json:"price" bson:"price"`
}

func main() {
	// Establishing mongo db connection
	ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)
	client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
	if err != nil {
		panic(err)
	}
	
	var limit int64 = 10
	var page int64 = 1
	collection := client.Database("myaggregate").Collection("stocks")

	//Example for Aggregation

	//match query
	match := bson.M{"$match": bson.M{"qty": bson.M{"$gt": 10}}}

	//group query
	projectQuery := bson.M{"$project": bson.M{"_id": 1, "qty": 1}}

    // set collation if required
    collation := options.Collation{
		Locale:    "en",
		CaseLevel: true,
	}

	// you can easily chain function and pass multiple query like here we are passing match
	// query and projection query as params in Aggregate function you cannot use filter with Aggregate
	// because you can pass filters directly through Aggregate param
	aggPaginatedData, err := New(collection).SetCollation(&collation).Context(ctx).Limit(limit).Page(page).Sort("price", -1).Aggregate(match, projectQuery)
	if err != nil {
		panic(err)
	}

	var aggProductList []Product
	for _, raw := range aggPaginatedData.Data {
		var product *Product
		if marshallErr := bson.Unmarshal(raw, &product); marshallErr == nil {
			aggProductList = append(aggProductList, *product)
		}

	}

	// print ProductList
	fmt.Printf("Aggregate Product List: %+v\n", aggProductList)

	// print pagination data
	fmt.Printf("Aggregate Pagination Data: %+v\n", aggPaginatedData.Data)
}

```

## For Normal queries
``` go


func main() {
	// Establishing mongo db connection
	ctx, _ := context.WithTimeout(context.Background(), 10*time.Second)
	client, err := mongo.Connect(ctx, options.Client().ApplyURI("mongodb://localhost:27017"))
	if err != nil {
		panic(err)
	}

	// Example for Normal Find query
    	filter := bson.M{}
    	var limit int64 = 10
    	var page int64 = 1
    	collection := client.Database("myaggregate").Collection("stocks")
    	projection := bson.D{
    		{"name", 1},
    		{"qty", 1},
    	}
    	// Querying paginated data
    	// Sort and select are optional
        // Multiple Sort chaining is also allowed
        // If you want to do some complex sort like sort by score(weight) for full text search fields you can do it easily
        // sortValue := bson.M{
        //		"$meta" : "textScore",
        //	}
        // aggPaginatedData, err := paginate.New(collection).Context(ctx).Limit(limit).Page(page).Sort("score", sortValue)...
        var products []Product
    	paginatedData, err := New(collection).Context(ctx).Limit(limit).Page(page).Sort("price", -1).Select(projection).Filter(filter).Decode(&products).Find()
    	if err != nil {
    		panic(err)
    	}
    
    	// paginated data or paginatedData.Data will be nil because data is already decoded on through Decode function
    	// pagination info can be accessed in  paginatedData.Pagination
    	// print ProductList
    	fmt.Printf("Normal Find Data: %+v\n", products)
    
    	// print pagination data
    	fmt.Printf("Normal find pagination info: %+v\n", paginatedData.Pagination)
}
    
```
***Notice:***
```go
paginatedData.data //it will be nil incase of  normal queries because data is already decoded on through Decode function
```

## Running the tests

``` bash
$ go test
```

## Contributing
Pull requests are welcome. For major changes, please open an issue first to discuss what you would like to change.

Please make sure to update tests as appropriate.


## Acknowledgments
<ol>
<li> https://github.com/mongodb/mongo-go-driver </li>
</ol>


## MIT License

```
Copyright (c) 2021
```

[Build-Status-Url]: https://travis-ci.com/gobeam/mongo-go-pagination
[Build-Status-Image]: https://travis-ci.com/gobeam/mongo-go-pagination.svg?branch=master
[godoc-url]: https://pkg.go.dev/github.com/gobeam/mongo-go-pagination?tab=doc
[godoc-image]: https://godoc.org/github.com/gobeam/mongo-go-pagination?status.svg
