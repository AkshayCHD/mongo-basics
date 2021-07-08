## Comparision Operators

use sample_training
 
Find all documents where the tripduration was less than or equal to 70 seconds and the usertype was not Subscriber:
```
db.trips.find({ "tripduration": { "$lte" : 70 },
                "usertype": { "$ne": "Subscriber" } }).pretty()
``` 
Find all documents where the tripduration was less than or equal to 70 seconds and the usertype was Customer using a redundant equality operator:
```
db.trips.find({ "tripduration": { "$lte" : 70 },
                "usertype": { "$eq": "Customer" }}).pretty()
``` 
Find all documents where the tripduration was less than or equal to 70 seconds and the usertype was Customer using the implicit equality operator:
```
db.trips.find({ "tripduration": { "$lte" : 70 },
                "usertype": "Customer" }).pretty()

```

## Logical Operators
Mongodb has the following logical operators `$or`, `$and`, `$nor` and `$not`

The syntax of all operators is as follows
```
{ $operator: [statement1, statement2, statement3...] }
```

The functioning of all operators is quite obvious by there name, there is a point to note for `$and`.
Like the `$eq` operator `$and` is implicitly applied to all queries, when we write

```
db.quotes.find({ writer: "caeser", language: "english" })
```
Then we want both conditions to be true simultaneously, which means `$and`.
But there are situations where `$and` has to be provided explicitly.

For eg: If we have a db of air routes and we want to find a route, where the flight started/ended at KZN airport, and the flight number is one of CR2 or A81. For this if we run the following 
```
db.routes.count({ $or: [{ dst_airport: "KZN" }, { src_airport: "KZN" }], $or: [{ airplane: "CR2" }, { airplane: "A81" }]})
```
We even get count of routes where airplane is CR2 but neither start or end is KZN. So to achieve the desired result, that is to use the same operator more than once in a query, we have to run
```
db.routes.count({ 
    $and: [
        { $or: [
            { dst_airport: "KZN" },
            { src_airport: "KZN" }
        ]}, 
        { $or: [
            { airplane: "CR2" },
            { airplane: "A81" }
        ]}
    ]
})
```
If we have 2 different expressions then we do not need to provide $and explicitly
```
db.routes.count({ 
        $or: [
            { dst_airport: "KZN" },
            { src_airport: "KZN" }
        ],
        airplane: { $not: { $eq: "A81" } }
})
```

In case of `$not` operator make sure we provide it a logical operator to negate, otherwise it will give an error

Question To complete this exercise connect to your Atlas cluster using the in-browser IDE space at the end of this chapter.

How many companies in the sample_training.companies dataset were

either founded in 2004
[and] either have the social category_code [or] web category_code,

[or] were founded in the month of October
[and] also either have the social category_code [or] web category_code?

Answer
```
db.companies.count({ $or: [{ 
    $and: [
        { founded_year: 2004 },
        { $or: [{ category_code: "social" }, { category_code: "web" } ]} 
    ]},
    {$and: [
        { founded_month: 10 },
        { $or: [{ category_code: "social" }, { category_code: "web" } ]} 
    ]
}]})
```

## Expressive query operator
denoted by `$expr` allows the use of aggregation expressions within the query language
It also allows us to use variables and conditional statements

Find all documents where the trip started and ended at the same station:
```
db.trips.find({ "$expr": { "$eq": [ "$end station id", "$start station id"] }
              }).count()
```
Find all documents where the trip lasted longer than 1200 seconds, and started and ended at the same station:
```
db.trips.find({ "$expr": { "$and": [ { "$gt": [ "$tripduration", 1200 ]},
                         { "$eq": [ "$end station id", "$start station id" ]}
                       ]}}).count()
```
We can use $ to access document field values

Question: Operators like $eq, $gt take input in the form of array inside $expr, unlike there usage without expr, why?

Answer, Inside the expr operator aggregation syntax is used instead of normal mql syntax, we'll be comming to aggregation syntax later


## Array Operators

`$push` pushes that value to the array field
`$size` used to get documents where the size of an array field is equal to provided size
`$all` all documents where the array field contains all the values provided

```
db.listingsAndReviews.find({ amenities: { $size: 20, $all: ["Internet", "Wifi"] }  }).count()
```
Find all documents with amenities size 20 and containing, Internet and Wifi as items in it.

## Projection
The second parameter in find query represents projection.
The important thing to note is that in projection section, you can either list items to include, or list items to exclude. You cannot mix the items (except for _id)

for example if we write
```
db.listingsAndReviews.find({ "amenities": "Wifi" },
                           { "price": 1, "address": 1,
                             "_id": 0, "maximum_nights":0 }).pretty()
```

we get the following error

```
Error: error: {
        "operationTime" : Timestamp(1625294018, 20),
        "ok" : 0,
        "errmsg" : "Cannot do exclusion on field maximum_nights in inclusion projection",
        "code" : 31254,
        "codeName" : "Location31254",
        "$clusterTime" : {
                "clusterTime" : Timestamp(1625294022, 1),
                "signature" : {
                        "hash" : BinData(0,"S62itNty/uJaZ/GtbexdjFlgTpw="),
                        "keyId" : NumberLong("6964943813026512899")
                }
        }
}
```

But this is valid
```
db.listingsAndReviews.find({ "amenities": "Wifi" },
                           { "price": 1, "address": 1, "_id": 0 }).pretty()
```

So basically, either all values on projection should be 1 or 0, except for _id

## $elemMatch
Match documents that contain an array field with atleast one element that matches the specified query criteria.
Can be used inside query part or projection part.

```
db.grades.find({}, { scores: { $elemMatch: { score: { $gt: 90 }  } } }).limit(5).pretty()
```
where scores is an array of objects have score as an an attribute

If used in project part, the documents where none of the scores are above 90 are also taken part as a result, but there scores variable is not projected

```
{ "_id" : ObjectId("56d5f7eb604eb380b0d8d8ce") }
{
        "_id" : ObjectId("56d5f7eb604eb380b0d8d8cf"),
        "scores" : [
                {
                        "type" : "exam",
                        "score" : 91.97520018439039
                }
        ]
}
{ "_id" : ObjectId("56d5f7eb604eb380b0d8d8d0") }
{ "_id" : ObjectId("56d5f7eb604eb380b0d8d8d1") }
{
        "_id" : ObjectId("56d5f7eb604eb380b0d8d8d2"),
        "scores" : [
                {
                        "type" : "quiz",
                        "score" : 91.7351500084582
                }
        ]
}
```

## Array Operators and sub documents

To query inside documents mql uses dot notation. For looking into array object at an index use `"<field>.0": "Seattle"`

```
use sample_training

db.trips.findOne({ "start station location.type": "Point" })

db.companies.find({ "relationships.0.person.last_name": "Zuckerberg" },
                  { "name": 1 }).pretty()

db.companies.find({ "relationships.0.person.first_name": "Mark",
                    "relationships.0.title": { "$regex": "CEO" } },
                  { "name": 1 }).count()


db.companies.find({ "relationships.0.person.first_name": "Mark",
                    "relationships.0.title": {"$regex": "CEO" } },
                  { "name": 1 }).pretty()

db.companies.find({ "relationships":
                      { "$elemMatch": { "is_past": true,
                                        "person.first_name": "Mark" } } },
                  { "name": 1 }).pretty()

db.companies.find({ "relationships":
                      { "$elemMatch": { "is_past": true,
                                        "person.first_name": "Mark" } } },
                  { "name": 1 }).count()
```

Q How many trips in the sample_training.trips collection started at stations that are to the west of the -74 longitude coordinate?

Longitude decreases in value as you move west.

Trip Object
```
{
        "_id" : ObjectId("572bb8222b288919b68abf5a"),
        "tripduration" : 379,
        "start station id" : 476,
        "start station name" : "E 31 St & 3 Ave",
        "end station id" : 498,
        "end station name" : "Broadway & W 32 St",
        "bikeid" : 17827,
        "usertype" : "Subscriber",
        "birth year" : 1969,
        "gender" : 1,
        "start station location" : {
                "type" : "Point",
                "coordinates" : [
                        -73.97966069,
                        40.74394314
                ]
        },
        "end station location" : {
                "type" : "Point",
                "coordinates" : [
                        -73.98808416,
                        40.74854862
                ]
        },
        "start time" : ISODate("2016-01-01T00:00:45Z"),
        "stop time" : ISODate("2016-01-01T00:07:04Z")
}
```

Answer
```
db.trips.countDocuments({ "start station location.coordinates.0": { $lt: -74 } })
```