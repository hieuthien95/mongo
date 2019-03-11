# mongo.exe
## Cấp database
### Show all database
```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
### Tạo database
```
> use mydemo
switched to db mydemo
```
### Xóa database
```
> db.dropDatabase()
{ "dropped" : "mydemo", "ok" : 1 }
```

## Cấp collection
### Tạo collection
```
> db.createCollection("persons")
{ "ok" : 1 }
```
### Show all collection
```
> show collections
persons
books
```
### Xóa collection
```
> db.persons.drop()
true
```

## Cấp document
### Thêm 1 document
```
> db.persons.insert({name:"loan", age: 57})
WriteResult({ "nInserted" : 1 })
```
```
> var per = {}
> per.name = "Bui Hieu Thien"
Bui Hieu Thien
> per.age = 24
24
> per.exp = ["PHP", "JAVA", "Golang"]
[ "PHP", "JAVA", "Golang" ]

> db.persons.insert(per)
> db.persons.save(per)
WriteResult({ "nInserted" : 1 })
```
### Thêm nhiều document
```
> db.persons.insertMany([{name:"thien", age: 24}, {name:"tram"}])
{
        "acknowledged" : true,
        "insertedIds" : [
                ObjectId("5c85e9f3da2c0d6f87b95a90"),
                ObjectId("5c85e9f3da2c0d6f87b95a91")
        ]
}
```
### Show all document
```
> db.persons.find({})

> db.persons.find()

> db.persons.find().pretty()
{ "_id" : ObjectId("5c85ea1eda2c0d6f87b95a90"), "name" : "loan", "age" : 57 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a91"), "name" : "thien", "age" : 24 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a92"), "name" : "tram" }
```
### Delete doccument: db.persons.deleteMany({ // dieu kien })
```
> db.persons.deleteMany({})
{ "acknowledged" : true, "deletedCount" : 3 }
```


# mongoexport.exe
### Export dạng json
```
C:\Windows\system32> mongoexport --db mydemo --collection persons --out "C:\Program Files\MongoDB\Server\4.0\data\export\persons.json"
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```
### Export dạng csv
```
C:\Windows\system32> mongoexport -d mydemo -c persons --type=csv  -o "C:\Program Files\MongoDB\Server\4.0\data\export\persons.csv" -f id,name,exp
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```

# mongoimport.exe
### Import dạng json
```
C:\Windows\system32> mongoimport --db mydemo --collection persons --file "C:\Program Files\MongoDB\Server\4.0\data\export\persons.json"
2019-03-11T13:14:47.830+0700    connected to: localhost
2019-03-11T13:14:47.849+0700    imported 1 document
```
### Import dạng csv
```
C:\Windows\system32> mongoimport -d mydemo -c persons --type=csv --file "C:\Program Files\MongoDB\Server\4.0\data\export\persons.csv" --headerline
2019-03-11T13:14:47.830+0700    connected to: localhost
2019-03-11T13:14:47.849+0700    imported 1 document
```





