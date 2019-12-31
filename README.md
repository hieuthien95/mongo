# (I) mongo.exe
```
> use mydemo
switched to db mydemo
> db.createUser(
...    {
...      user: "root",
...      pwd: "123456789",
...      roles: [ "readWrite", "dbAdmin" ]
...    }
... )
```
___
## 1. Cấp database
#### a. Show all database
```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
#### b. Create database
```
> use mydemo
switched to db mydemo
```
#### c. Delete database
```
> db.dropDatabase()
{ "dropped" : "mydemo", "ok" : 1 }
```

___
## 1. Cấp collection
#### a. Show all collection
```
> show collections
persons
books
```
#### b. Create collection
```
> db.createCollection("persons")
{ "ok" : 1 }
```
#### c. Delete collection
```
> db.persons.drop()
true
```

___
## 1. Cấp document
### a. Đánh chỉ số index
> **single-multi option** 

> **sparse option** loại bỏ null tại field được index

> **ttl option** Time to live
```
> db.collection.createIndex( 
        { 
                field1>: <type>, <field2>: <type2>, ... 
        }, 
        { 
                // unique option: {unique: true},
                // sparse option: {sparse: true},
                // TTL option: {expireAfterSeconds: 3600},
                // Partial option: partialFilterExpression: { salary: { $gt: 5 } } 
        }  
)

PS: > db.players.createIndex( { score: 1 } , { sparse: true } )
    > db.players.find().sort( { score: -1 } ).hint( { score: 1 } )
```
```
> db.zipcode.createIndex({pop:-1})
{
        "createdCollectionAutomatically" : false,
        "numIndexesBefore" : 1,
        "numIndexesAfter" : 2,
        "ok" : 1
}

> db.zipcode.getIndexes()
[
        {
                "v" : 2,
                "key" : {
                        "_id" : 1
                },
                "name" : "_id_",
                "ns" : "mydemo.zipcode"
        },
        {
                "v" : 2,
                "key" : {
                        "pop" : -1
                },
                "name" : "pop_-1",
                "ns" : "mydemo.zipcode"
        }
]

> db.zipcode.dropIndex("pop_-1")
{ "nIndexesWas" : 2, "ok" : 1 }
```
**After**
```
> db.zipcode.find({pop: {$eq: 4546}}).explain("executionStats")
"executionStats" : {
        "executionTimeMillis" : 17,
        "totalDocsExamined" : 29353,
        ...
}
```
**Before**
```
> db.zipcode.find({pop: {$eq: 4546}}).explain("executionStats")
"executionStats" : {
        "executionTimeMillis" : 0,
        "totalDocsExamined" : 2,
        ...
}
```


> **Text indexes** search text input có trong các field được liệt kê - full text search
```
> db.adminCommand({ setParameter: true, textSearchEnabled: true})

> db.collection.createIndex(
   { <field1>: "text", <field2>: "text"... },    // các field muốn search được liệt kê
   { default_language: "spanish" }              // docs.mongodb.com/manual/reference/text-search-languages/#text-search-languages
)

> db.collection.find({ 
        $text: { 
                $search: "text noi dung",       // "noi dung": "noi"->match, "dung"->match
                                                // "\"noi dung\"": "noi dung"->match
                                                // "noidung -noidungkhac": "noidung"->match, "noidungkhac"->fail
                $caseSensitive: <boolean>,    
                $language: <string>,
                $diacriticSensitive: <boolean>
         } 
})
```
```
> db.persons.createIndex( 
    { name: "text", exp: "text" }, 
    {default_language: "none"} 
)

> db.persons.find({ $text: { $search: "Thien" } })
{
    "_id" : ObjectId("5c864c0fda2c0d6f87b95a9c"),
    "name" : "Bui Hieu Thien",
    "age" : 25.0,
    "exp" : [ 
        "PHP", 
        "JAVA", 
        "Golang"
    ],
    "class" : "mongodb"
}

> db.persons.find({ $text: { $search: "php -golang"} })
{
    "_id" : ObjectId("5c864c0fda2c0d6f87b95a9d"),
    "name" : "Tran Van A",
    "age" : 20.0,
    "exp" : [ 
        "PHP"
    ]
}
```

> Kiểm tra dung lượng các index
```
> db.stats().indexSize
253952
```

```
> db.runCommand({ dbStats: 1, scale: 1 })

{
    "db" : "payroll_stg_pms",
    "collections" : 6,
    "views" : 0,
    "objects" : 32,
    "avgObjSize" : 288.21875,
    "dataSize" : 9223.0,
    "storageSize" : 221184.0,
    "numExtents" : 0,
    "indexes" : 8,
    "indexSize" : 253952.0,
    "fsUsedSize" : 3630948352.0,
    "fsTotalSize" : 214734073856.0,
    "ok" : 1.0
}
```

```
> db.policy.aggregate([ { $indexStats: {} } ]);

db.users.aggregate([ { $indexStats: {} } ]);
[{
    "name": "idx_users_by_created_at_user_status",
    "key": {
        "created_at": 1,
        "user_status": 1
    },
    "host": "6b1b716ae456:27017",
    "accesses": {
        "ops": NumberLong(456550227),
        "since": ISODate("2018-05-31T15:31:09.020Z")
    }
} {
    "name": "idx_users_by_user_status",
    "key": {
        "user_status": 1
    },
    "host": "6b1b716ae456:27017",
    "accesses": {
        "ops": NumberLong(277641131),
        "since": ISODate("2018-05-31T15:05:39.148Z")
    }
} {
    "name": "_id_",
    "key": {
        "_id": 1
    },
    "host": "6b1b716ae456:27017",
    "accesses": {
        "ops": NumberLong(0),
        "since": ISODate("2018-05-31T15:03:12.197Z")
    }
}]
```

### b. Show document
```
// https://docs.mongodb.com/manual/reference/operator/query/regex/index.html
> db.collection.find({ 
    <field>: { 
        $regex: /pattern/, 
        $options: '<options>' 
    } 
})

> db.collection.find(
        query,                  // câu lệnh where: { name:"Bui Hieu Thien", age: { $gt: 24 } }
        projection              // những record cần select: { "name": 1, "age": 1, _id: 0 }
)

> db.collection.find().sort( { name: 1 } ).limit(n).skip(n)
```
```
> db.products.find( { sku: { $regex: /^ABC/i } } )

> db.persons.find({})
> db.persons.find()

> db.persons.find().pretty()
{ "_id" : ObjectId("5c85ea1eda2c0d6f87b95a90"), "name" : "loan", "age" : 57 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a91"), "name" : "thien", "age" : 24 }
{ "_id" : ObjectId("5c85e9f3da2c0d6f87b95a92"), "name" : "tram" }

> db.restaurant.find({       
    grades: {
        $elemMatch: { 
            grade: "A", 
            score: {$gte: 13}, 
            date: { 
                $gt: new Date(2014,1,1), 
                $lt: new Date(2014,1,2) 
            } 
        }
    }
})
{
    "_id" : ObjectId("5c872bb1c0841c2215236763"),
    "address" : {
        "building" : "9718",
        "coord" : [ 
            -73.8887152, 
            40.6337296
        ],
    },
    "borough" : "Brooklyn",
    "grades" : [ 
        {
            "date" : ISODate("2014-02-01T00:00:00.000Z"),
            "grade" : "A",
            "score" : 13
        }, 
        {
            "date" : ISODate("2013-01-09T00:00:00.000Z"),
            "grade" : "A",
            "score" : 12
        }, 
    ],
    "name" : "Win Hing Chinese Restaurant",
    "restaurant_id" : "41152860"
}
```

### c. Insert 1 document
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
### d. Insert nhiều document
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
### e. Update
```
> db.collection.update(
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
### d. Delete doccument: 
```
> db.collection.deleteMany(
   <query>
)
```
```
> db.persons.deleteMany({name: "thien"})
{ "acknowledged" : true, "deletedCount" : 3 }
```
### e. Remove doccument
```
> db.collection.remove(
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


### f. Insert 1 document bằng javascript
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
### g. Insert nhiều document bằng javascript
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
### h. Auto increate id
```
db.counters.insertMany([{_id:"productid",sequence_value:0},{_id:"articleid",sequence_value:0}])
function getNextSequenceValue(sequenceName){
   var sequenceDocument = db.counters.findAndModify({
      query:{_id: sequenceName },
      update: {$inc:{sequence_value:1}},
      new:true
   });
   return sequenceDocument.sequence_value;
}
var seqProducrId = getNextSequenceValue("productid")
db.products.insert({
   "_id": seqProducrId,
   "product_name":"Apple iPhone",
   "category":"mobiles"
})
seqProducrId = getNextSequenceValue("productid")
db.products.insert({
   "_id": seqProducrId,
   "product_name":"Samsung S3",
   "category":"mobiles"
})

seqProducrId = getNextSequenceValue("articleid")
db.articles.insert({
   "_id": seqProducrId,
   "product_name":"Apple iPhone",
   "category":"mobiles"
})
seqProducrId = getNextSequenceValue("articleid")
db.articles.insert({
   "_id": seqProducrId,
   "product_name":"Samsung S3",
   "category":"mobiles"
})
```
___
___
# (II) mongoexport.exe
### a. Export dạng json
```
C:\Windows\system32> mongoexport --db mydemo --collection persons --out "C:\Program Files\MongoDB\Server\4.0\data\export\persons.json"
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```
### b. Export dạng csv
```
C:\Windows\system32> mongoexport -d mydemo -c persons --type=csv  -o "C:\Program Files\MongoDB\Server\4.0\data\export\persons.csv" -f id,name,exp
2019-03-11T13:11:34.076+0700    connected to: localhost
2019-03-11T13:11:34.077+0700    exported 1 record
```

___
___
# (III) mongoimport.exe
### a. Import dạng json
```
C:\Windows\system32> mongoimport --db mydemo --collection persons --file "C:\Program Files\MongoDB\Server\4.0\data\export\persons.json"
2019-03-11T13:14:47.830+0700    connected to: localhost
2019-03-11T13:14:47.849+0700    imported 1 document
```
### b. Import dạng csv
```
C:\Windows\system32> mongoimport -d mydemo -c persons --type=csv --file "C:\Program Files\MongoDB\Server\4.0\data\export\persons.csv" --headerline
2019-03-11T13:14:47.830+0700    connected to: localhost
2019-03-11T13:14:47.849+0700    imported 1 document
```

___
___
# (IV) mongodump.exe
### a. Export DB normal
```
C:\Windows\system32> mongodump -d mydemo -o "C:\Program Files\MongoDB\Server\4.0\data\export"
2019-03-11T18:11:45.189+0700    writing mydemo.persons to
2019-03-11T18:11:45.201+0700    done dumping mydemo.persons (1 document)
```
### b. Export DB gzip
```
C:\Windows\system32> mongodump -d mydemo -o "C:\Program Files\MongoDB\Server\4.0\data\export" --gzip
2019-03-11T18:14:51.702+0700    writing mydemo.persons to
2019-03-11T18:14:51.710+0700    done dumping mydemo.persons (1 document)
```
### c. Export DB *.gz
```
C:\Windows\system32> mongodump -d mydemo --gzip --archive=tenfile.gz
2019-03-11T18:27:16.917+0700    writing mydemo.persons to archive 'tenfile.gz'
2019-03-11T18:27:16.922+0700    done dumping mydemo.persons (1 document)
```

___
___
# (V) mongorestore.exe
### a. Import DB normal
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
### b. Import DB gzip
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
### c. Import DB *.gz
```
C:\Windows\system32> mongorestore --db mydemo --gzip --archive=tenfile.gz
```


=====
# Tham khảo Cursor Method
## Max/Min
```
> db.products.insertMany( [
    { "_id" : 5, "item" : "mango", "type" : "cortland", "cost" : 1.29, "warrantyYears" : 1, "available" : false },
    { "_id" : 9, "item" : "mango", "type" : "fuji", "cost" : 1.99 },
    { "_id" : 7, "item" : "mango", "type" : "honey crisp", "cost" : 1.99, "warrantyYears" : 1, "available" : true },
    { "_id" : 10, "item" : "mango", "type" : "jonagold", "cost" : 1.29, "warrantyYears" : 1, "available" : false },
    { "_id" : 1, "item" : "mango", "type" : "jonathan", "cost" : 1.29 },
    { "_id" : 6, "item" : "mango", "type" : "mcintosh", "cost" : 1.29 },
    { "_id" : 8, "item" : "orange", "type" : "cara", "cost" : 2.99 },
    { "_id" : 4, "item" : "orange", "type" : "navel", "cost" : 1.39, "warrantyYears" : 1, "available" : true },
    { "_id" : 3, "item" : "orange", "type" : "satsuma", "cost" : 1.99 },
    { "_id" : 2, "item" : "orange", "type" : "valencia", "cost" : 0.99, "warrantyYears" : 1, "available" : true }
] )
```

```
> db.products.createIndex({item: 1, type: 1})
> db.products.createIndex({item: 1, type: -1})
> db.products.createIndex({cost: 1})

> db.products.getIndexKeys()
[
    {
        "_id" : 1
    },
    {
        "item" : 1.0,
        "type" : 1.0
    },
    {
        "item" : 1.0,
        "type" : -1.0
    },
    {
        "cost" : 1.0
    }
]

> db.products.find().max( { item: 'mango', type: 'jonagold' } ).hint( { item: 1, type: 1 } )
...

> db.products.find().min( { item: 'mango', type: 'jonagold' } ).hint( { item: 1, type: 1 } )
...

> db.products.find().min( { cost: 1.39 } ).max( { cost: 1.99 } ).hint( { cost: 1 } )
...
```

# Check index
## explain
```
db.getCollection('user_role').find({
    org_code: "ghnexpress",
    user_id: "1874659",
    app_code: "payroll"
    })
.explain()
```

## indexStats
```
db.getCollection('payroll_detail_2019_11').aggregate( [ { $indexStats: { } } ] )
```

