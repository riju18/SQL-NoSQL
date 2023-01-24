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

<<<<<<< HEAD
+ **$and**          : and
+ **$or**           : or
+ **$gt**           : greater than
+ **$gte**          : greater than equal
+ **$lt**           : less than
+ **$lte**          : less than equal
+ **$eq**           : equal
+ **$ne**           : not equal
+ **$expr**         : for comparison
+ **$cond**         : custom condition
+ **$size**         : no of array elements
+ **$elemMatch**    : it applies all cond in same doc
+ **$slice**        : it skips some array elements
=======
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
>>>>>>> 4461196f3b3b1dc97af9728918f9efe0370c4ecf

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

<<<<<<< HEAD
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

=======
>>>>>>> 4461196f3b3b1dc97af9728918f9efe0370c4ecf
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

<<<<<<< HEAD
    > It applies all condition in same doc. If all condition is true in same doc then it returns those doc.
=======
  > It applies condition in same doc. If all condition is true in same doc then it returns those doc.
>>>>>>> 4461196f3b3b1dc97af9728918f9efe0370c4ecf

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
<<<<<<< HEAD

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
=======

    db.collectionName.find({arrayKey:{$size: 5}}, {"arrayKey":1})
>>>>>>> 4461196f3b3b1dc97af9728918f9efe0370c4ecf
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
