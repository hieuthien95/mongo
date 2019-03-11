# mongo.exe
___
## Cấp database
#### Show all database
```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
#### Create database
```
> use mydemo
switched to db mydemo
```
#### Delete database
```
> db.dropDatabase()
{ "dropped" : "mydemo", "ok" : 1 }
```

___
## Cấp collection
#### Show all collection
```
> show collections
persons
books
```
#### Create collection
```
> db.createCollection("persons")
{ "ok" : 1 }
```
#### Delete collection
```
> db.persons.drop()
true
```

___
## Cấp document
#### Show document
```
db.collection.find(
        query,                  // câu lệnh where: {"name" : "loan", "age" : 57 }
        projection              // những record cần select: { "name": 1, "age": 1, _id: 0 }
)
```
```
> db.persons.find({})
> db.persons.find()

> db.persons.find().pretty()
{ "_id" : ObjectId("5c85ea1eda2c0d6f87b95a90"), "name" : "loan", "age" : 57 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a91"), "name" : "thien", "age" : 24 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a92"), "name" : "tram" }
```

#### Insert 1 document
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
#### Insert nhiều document
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
#### Update
```
db.collection.update(
   <query>,                                     // viết WHERE tại đây: {name:"thien", age: 24}
   <update>,                                    // viết SET(update nội dung) hay UNSET(xóa field) tại đây để cập nhật: 
                                                // . vd set: {age: 25, class: "mongodb"}
                                                // . vd unset: {age: "", ...}
                                                https://docs.mongodb.com/manual/reference/method/db.collection.update/#update-parameter
   {
     upsert: <boolean>,                         // true: nếu không tồn tại record nào => **thêm** mới record đó
                                                // false: nếu không tồn tại record nào => **không thêm** record đó
                                                
     multi: <boolean>,                          // true: **cho phép** update 1 lúc nhiều record
                                                // false: **không cho phép** update 1 lúc nhiều record
                                                
     writeConcern: <document>,
     collation: <document>,
     arrayFilters: [ <filterdocument1>, ... ]
   }
)
```
```
> db.persons.update(
   {name:"thien", age: 24},
   { $set: { age: 25, class: "mongodb" } },
   {
     upsert: true,                        
     multi: true
   }
)
```
#### Delete doccument: 
```
db.collection.deleteMany(
   <query>
)
```
```
> db.persons.deleteMany({name: "thien"})
{ "acknowledged" : true, "deletedCount" : 3 }
```
#### Remove doccument
```
db.collection.remove(
   <query>,                     // https://docs.mongodb.com/manual/reference/operator/
   {
     justOne: <boolean>,        // true: chỉ xóa 1 record
                                // false: xóa nhiều
     writeConcern: <document>,
     collation: <document>
   }
)
```
```
> db.persons.remove(
   { age: {$gt: 25} },
   {
     justOne: true
   }
)
```


#### Insert 1 document bằng javascript
```
> load("C:/Users/hieut/Desktop/persons.js")
true
```
```
Nội dung file:
1. db.persons.insert({
2. 	"name":"Bui Hieu Thien",
3. 	"age":24.0,
4. 	"exp":["PHP","JAVA","Golang"]
5. })
```
#### Insert nhiều document bằng javascript
```
> load("C:/Users/hieut/Desktop/persons.js")
true
```
```
Nội dung file:
1. var pers = [
2. 	{
3. 		"name":"Bui Hieu Thien",
4. 		"age":24.0,
5. 		"exp":["PHP","JAVA","Golang"]
6. 	},
7. 	{
8. 		"name":"Tran Van A",
10. 		"age":20,
11. 		"exp":["PHP"]
12. 	}
13. ]
14. 
15. db.persons.insert(pers)
```

___
___
# mongoexport.exe
#### Export dạng json
```
C:\Windows\system32> mongoexport --db mydemo --collection persons --out "C:\Program Files\MongoDB\Server\4.0\data\export\persons.json"
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```
#### Export dạng csv
```
C:\Windows\system32> mongoexport -d mydemo -c persons --type=csv  -o "C:\Program Files\MongoDB\Server\4.0\data\export\persons.csv" -f id,name,exp
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```

___
___
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

___
___
# mongodump.exe
#### Export DB normal
```
C:\Windows\system32> mongodump -d mydemo -o "C:\Program Files\MongoDB\Server\4.0\data\export"
2019-03-11T18:11:45.189+0700    writing mydemo.persons to
2019-03-11T18:11:45.201+0700    done dumping mydemo.persons (1 document)
```
#### Export DB gzip
```
C:\Windows\system32> mongodump -d mydemo -o "C:\Program Files\MongoDB\Server\4.0\data\export" --gzip
2019-03-11T18:14:51.702+0700    writing mydemo.persons to
2019-03-11T18:14:51.710+0700    done dumping mydemo.persons (1 document)
```
#### Export DB *.gz
```
C:\Windows\system32> mongodump -d mydemo --gzip --archive=tenfile.gz
2019-03-11T18:27:16.917+0700    writing mydemo.persons to archive 'tenfile.gz'
2019-03-11T18:27:16.922+0700    done dumping mydemo.persons (1 document)
```

___
___
# mongorestore.exe
#### Import DB normal
```
C:\Windows\system32> mongorestore -d abc_xyz "C:\Program Files\MongoDB\Server\4.0\data\export\mydemo2"
2019-03-11T18:18:48.195+0700    the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
2019-03-11T18:18:48.197+0700    building a list of collections to restore from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo2 dir
2019-03-11T18:18:48.200+0700    reading metadata for abc_xyz.persons from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo2\persons.metadata.json
2019-03-11T18:18:48.216+0700    restoring abc_xyz.persons from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo2\persons.bson
2019-03-11T18:18:48.220+0700    no indexes to restore
2019-03-11T18:18:48.221+0700    finished restoring abc_xyz.persons (1 document)
2019-03-11T18:18:48.223+0700    done
```
#### Import DB gzip
```
C:\Windows\system32> mongorestore -d abc_xyz "C:\Program Files\MongoDB\Server\4.0\data\export\mydemo" --gzip
2019-03-11T18:21:40.270+0700    the --db and --collection args should only be used when restoring from a BSON file. Other uses are deprecated and will not exist in the future; use --nsInclude instead
2019-03-11T18:21:40.272+0700    building a list of collections to restore from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo dir
2019-03-11T18:21:40.274+0700    reading metadata for abc_xyz.persons from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo\persons.metadata.json.gz
2019-03-11T18:21:40.288+0700    restoring abc_xyz.persons from C:\Program Files\MongoDB\Server\4.0\data\export\mydemo\persons.bson.gz
2019-03-11T18:21:40.291+0700    no indexes to restore
2019-03-11T18:21:40.292+0700    finished restoring abc_xyz.persons (1 document)
2019-03-11T18:21:40.294+0700    done
```
#### Import DB *.gz
```
C:\Windows\system32> mongorestore --db mydemo --gzip --archive=tenfile.gz
```




