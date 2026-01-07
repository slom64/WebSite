- All Rows `"objects"` `{name: john}` in MongoDB called `document`, Documents lives in `collections` `tables`'users', Collections lives in `Databases` 'My_database'. 
- Keep in mind `object = {thing: "value"}`.
- Field equivalent to columns.
- When you deal with field "column" you should start with `$` like `$password`.

| Command                                                                                                                                                                                                                  | Description                                                                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `show dbs`<br>`show collections`                                                                                                                                                                                         | Show databases<br>show tables "collection" in database                                                                                                                                                         |
| `use <database-name>`                                                                                                                                                                                                    | Switch to database                                                                                                                                                                                             |
| `db.<collection>.insertOne(object)`<br>`db.<collection>.insertMany([object, object])`                                                                                                                                    | insert One object.<br>insert multiple objects.                                                                                                                                                                 |
| `db.<collection>.find()`<br>`db.<collection>.find(object)`<br>`db.<collection>.find(object1, object2)`<br>`db.<collection>.find().limit(1)`<br>`db.<collection>.find().sort(object)`<br>`db.<collection>.find().skip(1)` | works as select without where.<br>works as select with where.<br>Object2 is for specifying Columns <br>Limit number of retrived data.<br>Sort based on specific object.<br>Skip number of entries in database. |
| `db.<collection>.countDocuments(object`)                                                                                                                                                                                 | Object is same as find query.                                                                                                                                                                                  |
| `db.<collection>.updateOne()`<br>`db.<collection>.updateMany(Object1, Object2)`                                                                                                                                          | Update First occurrence document<br>Update All based Condition inObject1                                                                                                                                       |
| `db.<collection>.replaceOne(Object, Object2)`                                                                                                                                                                            | replace document with object2                                                                                                                                                                                  |
| `db.<collection>.deleteOne(Object)`<br>`db.<collection>.deleteMany(Object)`                                                                                                                                              | `Object as find, filter based on it.`                                                                                                                                                                          |

## Insertion
```mongodb
db.users.insertOne({name: "John"})
db.users.insertOne({name: "sally", age: 26 , address:{ street: "str1", country: "Earth" }, hoppies:[ "Running", "Swimming" ]})

```

## Selection
```mongodb
db.users.find()  // retrive all data from specific collection.
db.users.find({name: "ali"}) // select * from users where name = ali;
db.users.find({age: 26})
db.users.find({age:26}, {name: 1, age: 1, _id: 0}) // 1 --> we want to show this filed, 0 --> hide this fild
db.user.find({age: 26}, {age: 0}) // get all fields except age. 


db.users.find().limit(2) // limit number of retrived data.
db.users.find().sort({name: -1}) // -1 --> descending, 1 --> ascending
db.users.find().sort({age: 1, name: -1}) // sort based multiple fields.
```

### complex queries

| Expression | Description                       |
| ---------- | --------------------------------- |
| `$eq`      | equal                             |
| `$ne`      | Not equal                         |
| `$gt`      | greater than                      |
| `$gte`     | greater than or equal             |
| `$lt`      | less than                         |
| `$lte`     | less than or equal                |
| `$in`      | Used to equal in array            |
| `$nin`     | Not in array                      |
| `$exists`  | Check if field exists in document |

```
// Complex query = {$exp: Value}

db.users.find({field: {$eq: Value}})

db.users.find({name: {$eq: "Sally"}})
db.users.find({age: {$gt: 36}})
db.users.find({name: {$in:["ali", "hamada"]  }})
db.users.find({name: {$exists: true}}) // return documents that only contain name field.

// And
db.users.find( {age: { $gt: 26, $lt: 50 }} ) // greater than 26 and less than 50 
db.users.find ({age: {$gt: 26} , name: {$eq:"ali"}},  {name: 1}) // select name from users where age > 26 and name = "ali".
db.users.find({$and: [{age: 26},{name: {$eq:"ali"}}]},{name: 1}) // select name from users where age > 26 and name = "ali". Not too common syntax


// Or
db.users.find({$or: [{age: 26},{name: {$eq:"ali"}}]},{name: 1}) // select name from users where age > 26 or name = "ali". Not too common syntax


// Not
db.users.find({age: {$not:{$eq: 26 }}})

// Use fields
db.users.find({$expr: {$gt: ["$debt", "$balance"] }}) // select * from users where debt > balance;

// find based on attribute
db.users.find({"address.street": "Earth"})
```

## Update

```mongodb
// This update the field, it doesn't mean that it has deleted other fields and kept age field that equal 27.
db.users.updateOne({age: 26}, {$set: { age: 27 }}) // Update(age) from users set values(age=27) where age = 26
db.user.updateOne({_id: ObjectId("asdfz456")},{$set:{age: 27} })

// Change field name
db.user.updateOne({_id: ObjectId("asdfz456")},{$rename:{age: "YearsOnEarth"}})

// Remove field
db.user.updateOne({_id: ObjectId("asdfz456")},{$unset: {age: ""}})
```