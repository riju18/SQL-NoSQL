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

+ **$and**      : and
+ **$or**       : or
+ **$gt**       : greater than
+ **$gte**      : greater than equal
+ **$lt**       : less than
+ **$lte**      : less than equal
+ **$eq**       : equal
+ **$ne**       : not equal
+ **$expr**     : for comparison
+ **$cond**     : custom condition
+ **$size**     : no of array elements
+ **$elemMatch** : it applies all cond in same doc

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

+ get all data from collection

    ```mongojs
    db.collectionName.find()
    ```

+ get single data from collection

    ```mongojs
    db.collectionName.findOne()
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

  > It applies condition in same doc. If all condition is true in same doc then it returns those doc.

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

+ update multiple doc without cond.

    ```mongojs
    db.collectionName.updateMany({}, {$set:{key:"val"}})
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
