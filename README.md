# Basic Syntax of MongoDB

## Structure
*Assume we have a DB `blog` and collections `posts` and `comments` data structure look like:*

``` sql
posts {
    "_id" : ObjectId("5f5f3f52e9522dfdab884f6c"),
        "title" : "My first post!",
        "text" : "This is my first post, hope you will enjoy it.",
        "tags" : [
                "new",
                "tech"
        ],
        "create": 2020-07-29T13:17:08.868+00:00,
        "update": 2020-07-29T13:17:08.868+00:00,
        "like": 12, 
        "creator" : ObjectId("5f5f3ea1e9522dfdab884f6b"),
        "comments" : [
                {
                        "text" : "I like this post",
                        "author" : ObjectId("5f5f3ea1e9522dfdab884f6a"),
                        "like":1,
                        "create": 2020-07-29T13:17:40.868+00:00,
                        "update": 2020-07-29T13:17:40.868+00:00,
                },
                {
                        "text" : "Thank you",
                        "author" : ObjectId("5f5f3f52e9522dfdab884f6c"),
                        "like":2,
                        "create": 2020-07-29T13:17:49.868+00:00,
                        "update": 2020-07-29T13:17:49.868+00:00,
                }
        ]
}
```
## Create method
* *Create many `posts`:*

1. With `{ ordered: true }`:
```sql
// By default ordered = true 
db.posts.insertMany([
        {"title" : "My second post!", ...},
        {"title" : "My second post!",...}, 
        {"title" : "My third post!",...},
        {"title" : "My fourth post!",...},
        ...
])
```
and an *duplicate error* will be raised and others posts below that duplicate object (third post, fourth post,...) won't be inserted.
2. With `{ ordered: false }`:
```sql
db.posts.insertMany([
        {"title" : "My second post!", ...},
        {"title" : "My second post!",...}, 
        {"title" : "My third post!",...},
        {"title" : "My fourth post!",...},
        ...
], { ordered: false })
```
and an *duplicate error* will still be raised but all posts which aren't duplicate are inserted. 

* *Options for inserting a `post`:*
```sql
db.posts.insertOne(
        {"title" : "My second post!"},
        {writeConcern: {
                w: 1, 
                j: true, 
                wtimeout: 200
        }
})
```
Name | default value | Purpose
-----|---------------|---------
`w`    | `1`         |The w option requests acknowledgment that the write operation has propagated to a specified number of mongod instances or to mongod instances with specified tags. `{w: 0}` will not request acknowledgement.
`j`    | `false` | Make sure insert task will be execute by push it into a todo list with `{j: true}`.
`wtimeout`| `undefined`| Set timeout for insert executing action in millisecond.

## Read method.

* *To find all `posts`:*
``` SQL
db.posts.find()
```

* *To find `posts` with matching field(s):*
```SQL
db.posts.find({_id: ObjectId("<id object>")})
```

* *To find with conditions:*
```SQL
db.posts.find({likes: {$gt: 5}})
```
and the same with `$lt`, `$gte`, `$lte`, `$eq`, `$ne`, `$in`, `$nin`.

* *To query embedded documents:*
```SQL
 db.posts.find({"comments.author": ObjectId("<id object>)})
```

* *To query with logical condition:*
```SQL
 db.posts.find({ $or: [ { quantity: { $lt: 20 } }, { price: 10 } ] })
```
and the same with `$and`, `$nor`, `$not`.

* *Check a condition for a field:*
```SQL
db.posts.find({likes: {$exists: true, $gt: 5}})
```
or find that field which is not null:
```SQL
db.posts.find({likes: {$exists: true, $ne: null}})
```
or find a field having a specific type 
```SQL
db.posts.find({ field: { $type: <type1>} })
```
*or even an array of types of data accepted for checking a field's type is one in array*
```SQL
db.posts.find({ field: { $type: [ <BSON type1>; , <BSON type2>, ... ] } })
```

* *Query with evaluation:*
        *1. Find posts which haven't been update (Create equal to update date)*
        ```       
        db.posts.find( { $expr: { $eq: [ "$create" , "$update" ] } } )
        ```
        *2. Find posts which having word "first" in it's title with `regex`*
        ```
        db.posts.find({title: {$regex: /first/ }})
        ```
        *3. And so on...*
<br>

* *Query with skip and limit.*
**Note:** *These ways below give **the same** result.*
``` SQL
db.posts.find().sort({likes: 1, create: -1}).skip(10).limit(5)
```
```SQL
db.posts.find().skip(10).limit(5).sort({likes: 1, create: -1})
```

* *Find `posts` having both tags:*
``` SQL
// This will find all posts having tag "tech" and "new"
db.posts.find({tags: {$all: ["tech", "new"]}})
```

* *Find `posts` having it's title include "my"? :*
``` SQL
db.posts.find({title: {$regex : /my/i}})
// i is the flag for case sensitivity.
```

* ***Projection in MongoDB:***

*Definition:*
> The positional $ operator limits the contents of an <array> to return either:
> * The first element that matches the query condition on the array.
> * The first element if no query condition is specified for the array (Starting in MongoDB 4.4).

*Syntax*
1. Case projection with `$elemMatch`:
```SQL
db.posts.find({like: {$gt: 12}}, {tags: {$elemMatch: {$eq: "tech"}}})
```
and the result will look like:
``` sql
{ "_id" : ObjectId("5f5f3f52e9522dfdab884f6c"), "tags" : [ "tech" ] }
{ "_id" : ObjectId("5f64638adff26b47940f06d6") }
```
> *"The filter and the projection can be not related to each other, it's totally normal. However, with `$` operator, it have to filter and projection on the same field."* - Some trainer of an online MongoDB course.

2. Case projection with `$`:
```sql
db.posts.find({tags: {$eq: "new"}}, {"tags.$":1})
```
and the result will look like:
``` sql
{ "_id" : ObjectId("5f5f3f52e9522dfdab884f6c"), "tags" : [ "new" ] }
{ "_id" : ObjectId("5f64638adff26b47940f06d6"), "tags" : [ "new" ] }
```
*A little note:*
```sql
db.posts.find({tags: {$all: ["tech", "new"]}}, {"tags.$":1})
```
will return the result looked like:
``` sql
{ "_id" : ObjectId("5f5f3f52e9522dfdab884f6c"), "tags" : [ "new" ] }

// object 5f64638adff26b47940f06d6 doesn't have "tech" in tags
```
> *"Because mongo will make a check through all the array ["tech","new"], the result which element has "tech" in it's tags would be compared to which element has "new" in it's tags so **the first matching element** (as definition above) is "new", not "tech"." - Some trainer of an online MongoDB course.*

:exclamation: *Concluding*: 
* *We should consider between using `elemMatch` and`$` operator, we can take a look at docs below:*
> "The `$elemMatch`projection operator takes an explicit condition argument. This allows you to project based on a condition not in the query, or if you need to project based on multiple fields in the array’s embedded documents."

> "The `$` operator projects the first matching array element from each document in a collection based on some condition from the query statement."
* *A little note above is worth to keep in mind.*

## Update method.
* *Update field's value:*
```sql
db.posts.updateOne({_id: ObjectId("<id object>")}, {$set: {like: 2}})
```
or use `$unset` to **delete** fields and it's values:
```sql
db.posts.updateOne(
        {_id: ObjectId("<id object>")}, {$unset: {title: "", text: ""}}
)
```
* *Update increase or decrease a field's value:*
```sql
db.posts.updateOne({_id: ObjectId("<id object>")}, {$inc: {like: 2}})
```
and decrease with negative value: `{$inc: {like: -2}}`
* *Other update options:*

Name | Purpose | Syntax
-----|---------|--------
`$min` |Get the lowest value and set to the field which being updated. | `db.posts.updateOne({_id: ObjectId("<id object>")}, {$min: {like: 2}})`
`$max` |Get the highest value and set to the field which being updated. | `db.posts.updateOne({_id: ObjectId("<id object>")}, {$max: {like: 2}})`
`$mul` |Multiply the value of a field by a number. If `$mul` a field doesn't exist, result would be 0 and having the same numeric type.| `db.posts.updateOne({_id: ObjectId("<id object>")}, {$mul: {like: 2}})`

* *Updating all elements in array:*
```sql
// Increase 1 to field "like" of every object "comment" in array "comments"
db.collection.updateOne(
   {"_id" : ObjectId("<id object>")}, {$inc: {"comments.$[].like:" : 1}}
)
```
* *Insert a element to an array.*
```sql
// Insert and sort by ascending create date order
db.collection.updateOne(
   {"_id" : ObjectId("<id object>")}, 
   {$push: 
        {"comments": 
                {$each: [{
                        "text" : "I really like this post",
                        "author" : ObjectId("5f5f3ea1e9522dfdab884f6a"),
                        "like":0,
                        create: 2020-07-29T13:17:50.868+00:00,
                        update: 2020-07-29T13:17:50.868+00:00
                },
                {
                        "text" : "Thank you again",
                        "author" : ObjectId("5f5f3f52e9522dfdab884f6c"),
                        "like":0,
                        create: 2020-07-29T13:17:52.868+00:00,
                        update: 2020-07-29T13:17:52.868+00:00
                }]
        }
   }},
   {$sort: {create: -1}}, 
   $slice: 3,
)
```
Options of `$push`:
Name | Purpose | Syntax
-----|---------|--------
`$slice`|Limits the number of array elements. Requires the use of the $each modifier.| `$slice: 3` (No { })
`$position`|Specific a position to when pushing element to array.| `$position:4` (No { })

***NOTE:*** *Use `$addToSet` instead of `$push`if value inserted need to be unique.*
<br>

* *Removing element(s) from array:*
```sql
// remove with text|| condition
db.collection.updateOne(
   {"_id" : ObjectId("<id object>")}, {$pop: {comments: "thank you again"}}
)

//Or remove the last element (remove the first with {comments: -1})
db.collection.updateOne(
   {"_id" : ObjectId("<id object>")}, {$pop: {comments: 1}}
)
```
***NOTE:*** *If you work more on array, you should learn how to use arrayFilter*

## Delete Method.
* *Removing documents with more than 2 conditions:*
```sql
// remove many with conditions
db.collection.deleteMany(
   {likes: {$eq: 0}}, 
   {create: {$lt: 2020-07-29T13:17:08.868+00:00}},
   {creator: {$exists: false}}
)
```

# Advanced things.
## Indexes in MongoDB.

***Note 0:*** A collection has it's own `_id` field as it's default `index`. A collection can have many `indexes`.

***Note 1:*** Threshold for memory in MongoDB is 32 megabytes for sorting, in case of large documents, having `index` is a smart way to low the usage of memory for sorting because Mongo take all docs in it's own memory and sort elements in there. 

***Note 2:*** Using `index` having lots of benefits such as the fastest way to query or helping to reduce the usage of memories indicated above. However, it's important to use `index` at the right time and in the right place. 
For example: Looking for a `post` with it's title, collection should be create `index` with field `title` but looking for a `post` which having `create` before 07/29/2020 and having more than 10 `like` then `index` should be create on field `create` or create with both field (*Note 6*)

***Note 3:*** Create `index` on every single field will give us super fast query but it costs much more time whenever insert/update documents due to the creation/update of `index` for every field in the document.

***Note 4:*** if `index` are set to be unique and the field which was being nominated to be `index`, null/empty/not exist is still a value, which mean if two documents both having null/empty/not exist value for *nominated field* would raise a `duplicate error`. But we can deal with this with `partialFilterExpression`.
(This is `partial index`)
```sql
// "partialFilterExpression" will check duplicate if the  indicated field 
// is exists and having the same value (not value: null/undifined)
db.post.createIndex(
        {email:1}, 
        {
                unique: true, partialFilterExpression: {
                        email: {$exists: true}
                }
        }
)
// To understand how this works, please read "note 7"
```

***Note 5:*** If query return almost everything of the collection, `executionTimeMillis` of using `index` case will be greater than not using case because instead of return elements, pointers of the indexes are returned then another step have to 

***Note 6:*** Create `index` with more than two field in a time will create `index` having all fields, not create two `index`, beside, *the order of fields **does** matter*. For example:
(This was called `compound index`)
```sql 
// This will create index base on both "title" and "create"
// and "indexName" of it is "title_1_create_1"
db.post.createIndex({create: 1, title:1})
/* Index would be like below due to the order of fields
 Create: 10/20/2020 ---- Title: "Something"
                    ---- Title: "That thing"
                    ---- Title: "Those thing?"
 Create: 10/21/2020 ---- Title: "Something 1"
                    ---- Title: "That thing 2"
                    ---- Title: "That thing 3"
*/
```

***Note 7:*** With `partial index`, create a index with condition for the smaller size of `index` compare to `compound index` is absolutely possible. For example:
```sql 
// Create "index" for posts created after 7-29-2020
// This will make lots of sense if you rarely need posts created before 7-29-2020
// Smaller index, faster query
// Below was called "partial index"
db.post.createIndex(
        {title:1}, 
        {partialFilterExpression: {
                create: {$gt: 2020-07-29T13:17:08.868+00:00}
        }}
)
```
*Remember that when `partial index` was created, the field was chosen for being `partial index` **need to be specified** in the query for index scanning (IXSCAN) unless it would be collumn scanning (COLLSCAN):*
```sql 
// find posts which title including "my" and created day after 7-29-2020
db.post.find(
        {title: {$regex : /my/i}},
        {create: {$gt: 2020-07-29T13:17:08.868+00:00 }}
)
```


***Note 8:*** Difference between `partial index` and `compound index`:
Partial index|Compound index
-------------|---------------
The overall `index` simply smaller because values which not meet the conditions are not stored in there. => faster insert, update because they don't need to create `index`. | Create a general `index` for all indicated field result in many values of field(s) which is rarely needed still have `index`, then insert and update method will be more slower than `partial index`.

***Note 9:*** Set Time To Live (TTL) with `index` is an useful thing needs to know:
```sql 
// Whenever a new post/document is inserted, TTL will be trigger
// and delete that post/document after 1h from the created moment
db.post.createIndex(
        { "create": 1 }, 
        { expireAfterSeconds: 3600 } 
)

//Insert posts, they will be delete automatic after 1h.
db.post.insertMany([
        {title: "Hey there", create: new Date()},
        {title: "Hiiiiiiii dude", create: new Date()}
])
```
***Note 10:*** With an `index` on a array field, MongoDB will pull out all elements of that array and give indexes to them as separated document. This is `multi-key index`. 
Then *create a multi-key index with two array will raise error:*
> That mean you create index with every document in  array1 and with each document in array1 you will create index with all documents of array2, like O(n^2) of complexity. - Some trainer of an online MongoDB course.

## Explain() method.
* *"queryPlanner":* Show summary for the Executed Query and Winning plan 

* *"executionStats":* Show detailed summary for Executed Query, Winning Plan and Possibly Rejected Plans.

* *"allPlansExecution":* Show detailed summary for Executed Query, Winning Plan and Winning Plan Decision Process.

***Note 1:*** If *only indexes fields* returned and also filtering on those fields, `totalDocsExamined` will be `0`, only `totalKeysExamined` will equal to the returned result length, that mean this is a *efficient* querying. For example: 
```sql
// Create index on "title" with ascending order
db.posts.createIndex({title:1}) 

db.posts.explain("executionStats")
        .find({title: {$regex : /^my/i}}, {_id: 0, title: 1}
)
/*
"executionStats" : {
                "executionSuccess" : true,
                "nReturned" : 2,
                "executionTimeMillis" : 0,
                "totalKeysExamined" : 2,
                "totalDocsExamined" : 0,
                ...
}
*/
```

***Note 2:*** Create `index` will block every actions to the DB because Mongo uses the *foreground by default*. If don't want to, use option `{background: true}` and create `index` will be executed on the background but it's a *slower execution* compare with foreground. For example: 
```sql
db.posts.createIndex({title:1}, {background: true}) 
```