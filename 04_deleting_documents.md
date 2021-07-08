## Deleting
```
use sample_training
```
Look at all the docs that have test field equal to 1.
```
db.inspections.find({ "test": 1 }).pretty()
``` 
Look at all the docs that have test field equal to 3.
```
db.inspections.find({ "test": 3 }).pretty()
``` 
Delete all the documents that have test field equal to 1.
```
db.inspections.deleteMany({ "test": 1 })
``` 
Delete one document that has test field equal to 3.
```
db.inspections.deleteOne({ "test": 3 })
``` 
Inspect what is left of the inspection collection.
```
db.inspection.find().pretty()
``` 
View what collections are present in the sample_training collection.
```
show collections
``` 
Drop the inspection collection.
```
db.inspection.drop()
```

To drop a database

```
db.dropDatabase()
```
```
{
        "dropped" : "sample_trainings",
        "ok" : 1,
        "$clusterTime" : {
                "clusterTime" : Timestamp(1622377232, 12),
                "signature" : {
                        "hash" : BinData(0,"kIZN5Po10uW+GWNMA1UIBR9x3BE="),
                        "keyId" : NumberLong("6964943813026512899")
                }
        },
        "operationTime" : Timestamp(1622377232, 12)
}
```