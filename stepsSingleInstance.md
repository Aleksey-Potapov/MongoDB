# MongoDB. Single server 


- Download community edition 4.2 (zip) from off site 
- Unpack and add Bin to Paths
- Download and install Client Robo 3T only


### Run mongod by command

	mkdir C:\mongodb\single\instance
	mongod --dbpath C:\mongodb\single\instance

### Using the config file

	cd C:\mongodb\scripts 
	mkdir C:\mongodb\single\logs
	mongod -f singledb\singleShard.conf


## Create collection

Open Robo 3T
Create Connection to localhost on port 27017 and connect
Open Shell by right click on the Connection

	use singledb
	db.createCollection("users")
	db.users.renameCollection("user")
	db.user.insertOne( { name: "Workshop" } )
```javascript
db.content.insertOne(
   { 
       type:            "Journal",
       title:           "Exploring how and why young people use social networking sites", 
       authors:         ["Gray, L"],
       publicationDate: new Date("2018-04-03"),
       parentName:     "Educational Psychology in Practice",
       volume:           34 ,
       issue:            2 ,
       pages:           {
           start:NumberInt(175),
           end:NumberInt(194)},
       ids:             {doi: "10.1080/02667363.2018.1425829"},
       publisher:       "Routledge",
       abstracts:       "Upcoming statutory UK government guidance for keeping children safe in education reflects the use of social media, which is one of the most common activities undertaken by young people.",
       createDate:      new Date()
})
``` 
  
  
 Insert many documents from Journals10
 
	load("C:/MongoDB/Scripts2/journalsInsert.js")


## Search and Filters
 ```javascript
	db.content.find({ 
	 type: "Journal",
	 publisher : "Routledge"})
```

**Limit**
 ```javascript
	db.content.find({
	 type: "Journal",
	 publisher : "Routledge"}
	 ).limit(2) 
```
**Greater**
 ```javascript
	db.content.find({
	 type: "Journal",
	 publisher : "Routledge",
	 "pages.start":  {$gte: 454}
	}) 
```
**OR** 
 ```javascript
	db.content.find({
	 type: "Journal",
	 publisher : "Routledge",
	 $or : [
		{"pages.start":{$gte: 454}},
		{"pages.start": {$lt:36}}
		]
	})
```
**Take less fields**
 ```javascript
	db.content.find(
	 { parentName: "Educational Psychology in Practice" },
	 {title: true, _id: false})
 ```

**Regexp**
 ```javascript
	db.content.find({abstracts: {$regex:"work"}}) 
	db.content.find({abstracts:/work/}) 
 ```
## Update existing records
 ```javascript
	db.content.updateMany(
		{
	     publicationDate: { $exists: true },
	     publicationDate: { $type: "date" }
		},  
		[
		 { $set: { "publicationDates.print": "$publicationDate" }},
		 { $unset: "publicationDate"}
		]  
	)
```

## Aggregation
 ```javascript
	db.content.distinct("publisher")

	db.content.aggregate([
	  { $match: {"pages.end": { $exists: true }}},
	  { 
	    $group: {
	      _id: "$publisher",
	      sampleTitle: { $first : "$title" },
          count: { $sum: 1 },
          pages: { $sum :  { $subtract: [ "$pages.end", "$pages.start"  ]}} 
	    }
	  },
	  { 
	   $sort:{ _id : 1 }
	  }
	])
 ```

## Indexes
  ```javascript
db.content.find({title:/teacher/i})
db.content.find({title:/teacher/i}).explain()
 
db.content.createIndex({"title" : 1}, {"unique" : false}) 
  
db.content.find({title:/teacher/i}).explain()
  
   
db.content.getIndexes()
db.content.dropIndex("titile_1")
```

**Text index**

 ```javascript
db.content.createIndex({"abstracts" : "text"} )
db.content.find( { $text: { $search: "adults or and teacher’s" } } ).explain()

//Exact Phrase in quotas
db.content.find( { $text: { $search: "\"an active listening\"" } } )

//Term Exclusion
db.content.find( { $text: { $search: "an active listening -work" } } ).count()

//Sorting
//To sort the results in order of relevance score, you must explicitly project the $meta textScore //field and sort on it:

db.content.find(
   { $text: { $search: "article learn" } },
   { score: { $meta: "textScore" } }
).sort( { score: { $meta: "textScore" } } )

//Take only best match
db.content.aggregate(
   [
     { $match: { $text: { $search: "article learn" } } },
     { $project: { score: { $meta: "textScore" } } },
     { $match: { score: { $gt: 1.0 } } }
   ]
)

//Check the indexes usage.
db.content.aggregate( [ { $indexStats: { } } ] )
```

### Capped Collections
 ```javascript
db.createCollection("logs", {capped:true, size:9500, max: 150})
 
 
for (var i = 1; i <= 1000; ++i) {
  db.logs.insert({ 
      creationDate: new Date(),
      uid: i
  });
}
```


## Map/Reduce Functions 
 ```javascript
//Store and Load functions
function sqrt(n) { return n*n; }
  sqrt(5)
 
//Add function to the database
//Name: checkPublisher
function() { return this.publisher == "Routledge"; }
db.content.find(checkPublisher)

//To load stored functions
db.loadServerScripts();

//Add some data
db.usersessions.insertMany([
   { userid: "a", start: ISODate('2020-03-03 14:17:00'), length: 95 },
   { userid: "b", start: ISODate('2020-03-03 14:23:00'), length: 110 },
   { userid: "c", start: ISODate('2020-03-03 15:02:00'), length: 120 },
   { userid: "d", start: ISODate('2020-03-03 16:45:00'), length: 45 },
   { userid: "a", start: ISODate('2020-03-04 11:05:00'), length: 105 },
   { userid: "b", start: ISODate('2020-03-04 13:14:00'), length: 120 },
   { userid: "c", start: ISODate('2020-03-04 17:00:00'), length: 130 },
   { userid: "d", start: ISODate('2020-03-04 15:37:00'), length: 65 }
])

//Map
var mapFunction = function() {
    var key = this.userid;
    var value = { total_time: this.length, count: 1, avg_time: 0 };
    emit( key, value );
};

//Reduce
var reduceFunction = function(key, values) {
   var reducedObject = { total_time: 0, count:0, avg_time:0 };
   values.forEach(function(value) {
      reducedObject.total_time += value.total_time;
      reducedObject.count += value.count;
   });
   return reducedObject;
};

var finalizeFunction = function(key, reducedValue) {
   if (reducedValue.count > 0)
      reducedValue.avg_time = reducedValue.total_time / reducedValue.count;
   return reducedValue;
};

db.usersessions.mapReduce(
   mapFunction,
   reduceFunction,
   {
     out: "session_stats",
     finalize: finalizeFunction
   }
)

db.session_stats.find().sort( { _id: 1 } )

db.usersessions.insertMany([
   { userid: "a", ts: ISODate('2020-03-05 14:17:00'), length: 130 },
   { userid: "b", ts: ISODate('2020-03-05 14:23:00'), length: 40 },
   { userid: "c", ts: ISODate('2020-03-05 15:02:00'), length: 110 },
   { userid: "d", ts: ISODate('2020-03-05 16:45:00'), length: 100 }
])

db.usersessions.mapReduce(
   mapFunction,
   reduceFunction,
   {
     query: { ts: { $gte: ISODate('2020-03-05 00:00:00') } },
     out: { reduce: "session_stats" },
     finalize: finalizeFunction
   }
);

db.usersessions.aggregate([
  // { $match: { ts: { $gte: ISODate('2020-03-05 00:00:00') } } },
   { $group: { _id: "$userid", total_time: { $sum: "$length" }, count: { $sum: 1 }, avg_time: { $avg: "$length" } } },
   { $project: { value: { total_time: "$total_time", count: "$count", avg_time: "$avg_time" } } },
   { $merge: {
      into: "session_stats_agg",
      whenMatched: [ { $set: {
         "value.total_time": { $add: [ "$value.total_time", "$$new.value.total_time" ] },
         "value.count": { $add: [ "$value.count", "$$new.value.count" ] },
         "value.avg_time": { $divide: [ { $add: [ "$value.total_time", "$$new.value.total_time" ] },  { $add: [ "$value.count", "$$new.value.count" ] } ] }
      } } ],
      whenNotMatched: "insert"
   }}
])
```


 
## Utils

	mongostat --port 27017

	mkdir c:\mongodb\backup
	cd c:\mongodb\backup
	mongodump --port 27017 --db singledb --collection content
	mongorestore --drop --port 27017 dump/


	mongoexport --port 27017 --db singledb --collection content -o content.json
	mongoimport --port 27017 content.json

	 

