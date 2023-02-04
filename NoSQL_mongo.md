# How to use MongoDB

+ [**Syntax**](#syntax)
+ [**DB list**](#db)
+ [**Create**](#create)
+ [**Read**](#read)
+ [**Update**](#update)
+ [**Delete**](#delete)
+ [**Condition**](#condition)
+ [**Projection**](#projection)

# syntax

+ **$and**          : and
+ **$or**           : or
+ **$gt**           : greater than
+ **$gte**          : greater than equal
+ **$lt**           : less than
+ **$lte**          : less than equal
+ **$eq**           : equal
+ **$ne**           : not equal
+ **$inc**          : increment by
+ **$min**          : new value < existing value then update
+ **$max**          : new value > existing value then update
+ **$mul**          : multiply by
+ **$rename**       : rename new col
+ **$push**         : add element into array key
+ **$addToSet**     : like $push but only keeps unique val 
+ **$pull**         : remove element from array key
+ **$pop**          : remove element from array key by pos
+ **$expr**         : for comparison
+ **$cond**         : custom condition
+ **$size**         : no of array elements
+ **$elemMatch**    : it applies all cond in same doc
+ **$slice**        : it skips some array elements

# db

```mongojs
show dbs  // DB list
use DBName  // activate DB to run query
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

# read

+ get single data from collection

    ```mongojs
    db.collectionName.findOne()
    ```

+ get all data from collection

    ```mongojs
    db.collectionName.find()
    ```

+ sorting data

    ```mongojs
    // key: 1   --> asc
    // key: -1  --> desc

    db.collectionName.find().sort({key:1})
    ```

+ skip & limit data

    ```mongojs
    // skip & limt 10 data 

    db.collectionName.find().skip(10).limit(10)
    ```

+ condition

  + and / or

    ```mongojs
    db.collectionName.find({$and:
        [
            {key:{$gte:val}}, 
            {key:{$lte:val}}
        ]}
    )
    ```

  + elemMatch

    > It applies all condition in same doc. If all condition is true in same doc then it returns those doc.

    ```
    db.collectionName.find({key:{
        $elemMatch:
            {"nestedKey1": "val1"}, 
            {"nestedKey2": "val2"}}}, 
        {key:1, _id:0}
    )
    ```

+ search by array element

    ```mongojs
    db.collectionName.find({arrayKey:"val"})
    ```

+ search by array/nested doc

    ```mongojs
    db.collectionName.find(
        {arrayKey.nestedKey:"val"}
    )
    ```

  + search by exact value but it follows the order

        ```mongojs
        // it always returns those doc which has order val1 then val2

        db.collectionName.find(
            {arrayKey.nestedKey:["val1", "val2"]}
        )
        ```

  + search by exact value but it follows no order

        ```mongojs
        // it always return those doc which has val1 & val2 in array

        db.collectionName.find({arrayKey:
            {$all:["val1", "val2"]}},
            {"arrayKey":1})
        ```

+ search by number of array elements

    ```mongojs
    // return the array of size 5

    db.collectionName.find({arrayKey:{$size: 5}}, {"arrayKey":1})
    ```

+ slicing array elements

    ```mongojs
    // return only 1st 2 array elements

    db.collectionName.find({}, 
        {"genres":{$slice:2}, name:1}
    )


    // it skips 1st array element then returns next 2 array elements

    db.collectionName.find({}, 
        {"genres":{$slice:[1,2]}, name:1}
    )
    ```

# update

+ update one doc

    ```mongojs
    db.collectionName.updateOne(
        {key:val},
        {$set:{key: "val"}})
    ```

+ update multiple doc

    ```mongojs
    // insert embedded/nested doc

    db.collectionName.updateMany({}, 
    {$set:
        {key:
            {nestedKey1:"val", 
            nestedKey2:"val"
        }
    }})
    ```

+ update multiple doc with cond

    ```mongojs
    db.collectionName.updateMany({key:{$gt:val}}, {$set:key:"val"}})
    ```

+ update multiple doc without cond.

    ```mongojs
    db.collectionName.updateMany({}, {$set:{key:"val"}})
    ```

+ update & increment/multiply

    > can't use $inc/$mul & $set on same field on single operation

    ```mongojs
    // must be numeric key

    db.collectionName.updateOne(
        {key:"value"}, 
        {$inc: {"key": value}}
        )
    
    db.collectionName.updateOne(
        {key:"value"}, 
        {$inc: {"key": value},
        $set:{key: "value"}}
        )
    
    db.collectionName.updateOne(
        {key:"value"}, 
        {$mul: {"key": value}}
        )
    ```

+ remove a key

    ```mongojs
    // this will remove the key by condition
    
    db.collectionName.updateOne(
        {key:"value"}, 
        {$unset: {"key": ""}}
        )
    ```

+ rename

    ```mongojs
    db.collectionName.updateOne(
        {}, 
        {$rename: {"oldKey": "newKey"}}
        )
    ```

+ upsert

> if data not found then insert

```mongojs
db.collectionName.updateOne(
    {key:val}, 
    {$set:
        {key1: val1, 
        key2 : val2, 
        key3 : val3}}, 
    {upsert: true})
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
    1st arg: null => all data (no condition), 
    2nd arg: projection => [1: show, 0:hide]
    ** By default _id=1
    */

    db.collectionName.find({}, {key: 1, _id: 0})
    ```
