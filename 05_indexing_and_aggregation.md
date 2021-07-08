## Aggregation
Here we'll briefly discuss mongodb's aggragation framework.
Basically aggragation framework is a superset of MQL, that weuse to query data from mongodb, that means it can perform all the tasks of MQL along with a lot of other operations like grouping etc

Q What room types are present in the sample_airbnb.listingsAndReviews collection?
```
db.listingsAndReviews.aggregate([{ $group: { _id: "$room_type"  }  }])
```

## sort and limit
There are cursor methods, similar to pretty and count.
A cursor method is not applied to the data in the database, but it is applied to the data in the cursor.

Ideally sort should be placed before limit, but if that is not the case mongodb flips the order of the MQL query itself, during execution


## Indexing
there is a separate course to deal with indexing, here we just study the basics

Jameela often queries the sample_training.routes collection by the src_airport field like this:
```
db.routes.find({ "src_airport": "MUC" }).pretty()
```
Which command will create an index that will support this query?
```
db.routes.createIndex({ "src_airport": -1 })
```

## Upsert

There is an upsert flag that we can set while running our update MQL commands, what it does is, it tries to find a document that matches the query, if there is no such document found, it creates the a new document with the provided fields

```
db.iot.updateOne({ "sensor": r.sensor, "date": r.date,
                   "valcount": { "$lt": 24 } },
                         { "$push": { "readings": { "v": r.value, "t": r.time } },
                        "$inc": { "valcount": 1, "total": r.value } },
                 { "upsert": true })
```
Here if the valcount of a document is 24. then it will not be among the queried documents and hence a new document will be created to store the next 24 readings