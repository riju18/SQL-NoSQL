# How to use MongoDB

+ [**Syntax**](#syntax)
+ [**Data Type**](#data_type)
+ [**DB list**](#db)
+ [**Create**](#create)
+ [**Read**](#read)
+ [**Update**](#update)
+ [**Delete**](#delete)
+ [**Condition**](#condition)
+ [**Projection**](#projection)
+ [**Aggregation**](#aggregation)

# syntax

+ **$and**              : and
+ **$or**               : or
+ **$gt**               : greater than
+ **$gte**              : greater than equal
+ **$lt**               : less than
+ **$lte**              : less than equal
+ **$eq**               : equal
+ **$ne**               : not equal
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

# condition

+ and

    ```mongojs
    db.collectionName.find({$and:[
        {key:{$gte:val}},
        {key:{$lte:val}}
        ]})
    // upper limit included
    ```

+ search by array element

    ```mongojs
    db.collectionName.find({arrayKey:"val"})
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
