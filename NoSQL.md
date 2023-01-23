# How to use MongoDB

+ [**Syntax**](#syntax)
+ [**Data Type**](#data_type)
+ [**DB list**](#db)
+ [**Create**](#create)
+ [**Read**](#read)
+ [**Update**](#update)
+ [**Delete**](#delete)
+ [**Projection**](#projection)
+ [**Aggregation**](#aggregation)

# syntax

+ **$and**              : and
+ **$or**               : or
+ **$nor**              : opposite of or
+ **$not**              : documents that do not match
+ **$gt**               : greater than
+ **$gte**              : greater than equal
+ **$lt**               : less than
+ **$lte**              : less than equal
+ **$eq**               : equal
+ **$ne**               : not equal
+ **$in**               : in
+ **$nin**              : not in
+ **$exists**           : check the field exists or not
+ **new Date()**        : current datetime
+ **new TimeStamp()**   : current timestamp

# data_type

+ **Text**          : string
+ **Boolean**       : true/false
+ **Number**        : int32, int64, decimal
+ **ObjectId**      : auto generated random string ID
+ **ISODate**       : date (2023-01-03)
+ **Timestamp**     : uique time (11421532)
+ **Embedded Doc**  : nested doc
+ **array**         :

# db

```mongojs
show dbs  // DB list
use DBName  // create/activate DB to run query
db.stats()
```

# create

+ insert a doc

    ```mongo
    db.collectionName.insertOne({
        "key": "val",
        }
    )
    ```

+ insert multiple doc

    ```mongojs
        db.collectionName.insertMany([
            {
            "key": "val",
            },
            {
            "key": "val",
            }
        ])
    ```

+ insert doc through cmd

    ```mongojs
    mongoimport *.json -d dbName -c collectionName jsonArray --drop
    ```

# read

+ get all data from collection

    ```mongojs
    db.collectionName.find()
    ```

+ get single data from collection

    ```mongojs
    db.collectionName.findOne()
    ```

+ count of documents

    ```mongojs
    db.collectionName.find().count()
    ```

+ condition
  + and

    ```mongojs
    db.collectionName.find({$and:[
        {key:{$gte:val}},
        {key:{$lte:val}}
        ]})
    // upper limit included
    ```

  + or

    ```text
    same as "and" operator
    ```

  + search by array element

    ```mongojs
    // it works as in
    db.collectionName.find({arrayKey:"val"})

    // it works as equal
    db.collectionName.find({arrayKey:["val"]})
    ```

  + exists

    ```mongojs
    // returns the doc with null & not null values
    db.collectionName.find({key: {$exists:true}})
    
    // check key exists and not null
    db.collectionName.find({key: {$exists:true, $ne:null}})
    ```

  + type
    > specific data type
    >
    > [data type doc](https://www.mongodb.com/docs/manual/reference/operator/query/type/#mongodb-query-op.-type)

    ```mongojs
    db.collectionName.find({"key":{$type:["string"]}})
    ```

  + regex
    > matches pattern from text
    >
    > as like SQL "**ilike/like**" operator
    >
    > [regex](https://www.mongodb.com/docs/manual/reference/operator/query/regex/#mongodb-query-op.-regex)

    ```mongojs
    db.collectionName.find({"summary": {$regex: /musical/}})
    ```

# update

+ update one doc

    ```mongojs
    db.collectionName.updateOne({key:val},{$set:{key: "val"}})
    ```

+ update multiple doc

    ```mongojs
    // insert embedded/nested doc

    db.collectionName.updateMany({}, {$set:{key:{nestedKey1:"val", nestedKey2:"val"}}})
    ```

+ update multiple doc with cond

    ```mongojs
    db.collectionName.updateMany({key:{$gt:val}}, {$set:key:"val"}})
    ```

+ update multiple doc without cond

    ```mongojs
    db.collectionName.updateMany({}, {$set:{key:"val"}})
    ```

+ update array

    ```mongojs
    db.collectionName.updateMany({"arrayKey":"arrayVal"}, 
    {$set:{"arrayKey":["arrayVal"]}})
    ```

# delete

+ delete one doc

    ```mongojs
    db.collectionName.deleteOne({key:"vale"})
    ```

+ delete multiple doc

    ```mongojs
    db.collectionName.deleteMany({key:{$lt:val}})
    ```

# projection

+ show/hide

    ```mongojs
    /*
    1st arg : condition, 
    2nd arg : projection => [1: show, 0:hide]
    By default _id=1
    */

    db.collectionName.find({}, {key: 1, _id: 0})
    ```

+ condition in nested key

    ```mongojs
    db.collectionName.find({$and:
    [
        {key:"Manu Lorenz"}, 
        {"key.nestedKey":"val"}
    ]})
    ```

# aggregation

+ lookup

    > It's used to merge two collections

    ```mongojs
    db.collectionName1.aggregate([
        {$lookup:
            {from:"collectionName2", 
            localField: "keyName", foreignField:"keyName",
            as:"alias"
            }
        }
        ])
    ```
