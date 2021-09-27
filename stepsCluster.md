# MongoDB. Cluster

**Create folders to store the data**

	mkdir c:\mongodb\Cluster\db\csrs1
	mkdir c:\mongodb\Cluster\db\csrs2
	mkdir c:\mongodb\Cluster\db\node1
	mkdir c:\mongodb\Cluster\db\node2
	mkdir c:\mongodb\Cluster\db\node3
	mkdir c:\mongodb\Cluster\db\mongoserver
	mkdir c:\mongodb\Cluster\logs


**Run nodes for Config replica set**

	cd c:\mongodb\scripts
	START /B mongod -f  cluster\csrs1.conf
	START /B mongod -f  cluster\csrs2.conf

**Initiate Replica and add one node**

	mongo --port 26001 -host localhost
 ```javascript
rs.initiate() 
rs.add("localhost:26002")
rs.status()
exit
 ```

**Run working nodes and Mongos server for Sharded Cluster**

	START /B mongod -f   cluster\node1.conf
	START /B mongod -f   cluster\node2.conf
	START /B mongod -f   cluster\node3.conf
	START /B mongos -f  cluster\mongoserver.conf

**Add shards**

	mongo --port 26000 -host localhost 
 ```javascript
sh.status()
sh.addShard("localhost:27011")
sh.addShard("localhost:27012")
sh.status()
```
**Adjust chunk size**
 ```javascript
use config 
db.settings.insertOne( { _id:"chunksize", value:  5 } )
exit
```

## Work with cluster
 - Run Robomongo Shell
 - Connect to localhost:26000

 ```javascript
use students
sh.enableSharding("students")
sh.shardCollection("students.grades", {"student_id" : 1})

sh.status()

use students
db.grades.createIndex( { student_id: 1 } );

use students
for ( i = 0; i < 10000; i++ ) {
  db.grades.insert({student_id: i, type: "exam", score : Math.random() * 100 }); 
  db.grades.insert({student_id: i, type: "quiz", score : Math.random() * 100 }); 
  db.grades.insert({student_id: i, type: "homework", score : Math.random() * 100 });
}
```

**Check the speed in the new window**

	mongostat --port 26000 
	
 ```javascript
db.grades.getShardDistribution()
sh.startBalancer()
sh.status()
```

**Add third shard and run the script again**

```javascript
sh.addShard("localhost:27013")
sh.status()
```

**Check whenever it sharded**
	
	mongo --host localhost --port 27011
 ```javascript
use students
db.grades.find().sort({student_id:1}).limit(1).pretty()
db.grades.find().sort({student_id:-1}).limit(1).pretty()
```

	mongo --port 26000 --host localhost
 ```javascript
use students
db.grades.find().sort({student_id:1}).limit(1).pretty()
db.grades.find().sort({student_id:-1}).limit(1).pretty() 
```


## Write Concern
 ```javascript
use config
db.getCollection('actionlog').find({}).readConcern("linearizable").maxTimeMS(10000)

//0, 1, majority

db.products.insert(
   { item: "envelopes", qty : 100, type: "Clasp" },
   { writeConcern: { w: "majority" , wtimeout: 5000 } }
)

db.products.insert(
   { item: "envelopes", qty : 100, type: "Clasp" },
   { writeConcern: { w: 2, wtimeout: 5000 } }
)
 ```
 
 
### Turn off

1. Stop mongos routers. 
2. Stop each shard replica set. 
3. Stop config servers.
 ```javascript
use admin
db.shutdownServer()
exit
```
 
 