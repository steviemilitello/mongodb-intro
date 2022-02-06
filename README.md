![General Assembly Logo](https://ga-dash.s3.amazonaws.com/production/assets/logo-9f88ae6c9c3871690e33280fcf557f33.png)

# An Introduction to MongoDB

## Objectives

By the end of this, developers should be able to:

- Interact with MongoDB databases and collections using the MongoDB shell.
- Create, Read, Update, and Delete documents in MongoDB collections using the
  MongoDB shell.

## Preparation

1. Fork and clone this repository.
2. Create a new branch, `training`, for your work.
3. Checkout to the `training` branch.


## Introduction

MongoDB is a schema-less document-store that organizes documents in collections.
What does this mean?

### Terminology

| **MongoDB Term** | **MongoDB Definition**           |
|:-----------------|:---------------------------------|
| [document](https://docs.mongodb.com/manual/reference/glossary/#term-document)         | A BSON element in a collection. Similar to JSON objects. Made of name-value pairs. |
| [field](https://docs.mongodb.com/manual/core/document/#document-structure)            | The name (key) in a document name-value pair.  |
| [collection](https://docs.mongodb.com/manual/reference/glossary/#term-collection)       | A grouping of MongoDB documents. |
| [database](https://docs.mongodb.com/manual/reference/glossary/#term-database)         | A container for collections.     |

#### Document

> MongoDB stores data records as BSON documents. BSON is a binary representation of JSON documents, though it contains more data types than JSON.
> [mongodb](https://docs.mongodb.com/manual/core/document/#documents)

![document](https://docs.mongodb.com/manual/images/crud-annotated-document.bakedsvg.svg)

#### Collection

> MongoDB stores documents in collections.
> [mongodb](https://docs.mongodb.com/manual/core/databases-and-collections/#collections)

![collections](https://docs.mongodb.com/manual/images/crud-annotated-collection.bakedsvg.svg)

#### Demo:  Data Types

Let's look at some of the [MongoDB data types](https://docs.mongodb.com/manual/reference/bson-types/)
that we can store.

## Create a Database

### Code Along

We'll use `mongo-crud` as the database to hold our tables and
[mongo](https://docs.mongodb.org/manual/reference/program/mongo/) to interact
with it.  `mongo` is MongoDB's command line client which lets us execute
commands interactively and from scripts.

First let's fire up our server:

- Mac OS only:

```bash
brew services start mongodb-community
```

- Ubuntu Only:

```bash
sudo systemctl start mongodb
```

- WSL Only:

```bash
sudo service mongodb start
```

The command to enter into the MongoDB shell is `mongo <name-of-database>`:

```bash
$ mongo mongo-crud
MongoDB shell version v4.x.x
connecting to: mongo-crud
>
```

> Note: On Windows, this might not look the same, and could even look like
> it's hanging! Don't worry, this is to be expected. Follow along with the
> next step to confirm you're all set.

The command to display the current database in use is `db`:

```bash
> db
mongo-crud
>
```

The command to list databases is `show databases` (or `show dbs`):

```bash
> show databases
local  0.000GB # or local  0.078GB
>
```

MongoDB lets us select a database that hasn't been created.  When we add a
collection, the database will be created.

For instance, if we didn't specify the database on the command line, we can
connect to a database with `use <database-name>`:

```bash
$ mongo
MongoDB shell version v4.x.x
connecting to: test
> db
test
> use mongo-crud
switched to db mongo-crud
> db
mongo-crud
> show databases
local  0.000GB # or local  0.078GB
>
```

## Importing Data

To help us practice interacting with a Mongo database, we'll want some data to
work with. MongoDB's [`mongoimport`](https://docs.mongodb.org/manual/reference/program/mongoimport/)
command will let us load bulk data from a `JSON` or `CSV` file.

Our first collection will be called `people`. It has no entries.

```js
> show collections
> db.people.count()
0
```

This is a common pattern in MongoDB: you can refer to things that don't yet
exist, and it will cooperate.  MongoDB won't create them until you give it
something to remember.

### Code Along: Bulk Load People

First, we'll load data in bulk from `data/people.csv`.  We'll save the
command in `scripts/import/people.sh`.

```bash
mongoimport --db=mongo-crud --collection=people --type=csv --headerline --file=data/people.csv
```

If we want to clear the collection before the import, we pass the `--drop` flag.

Run this script by typing:

 ```bash
 sh path_to_file.sh
 ```

Now that we've inserted data into it, the `mongo-crud` database and the `people`
collection both exist.

```js
$ mongo mongo-crud
MongoDB shell version v4.x.x
connecting to: mongo-crud
> show dbs
local       0.000GB
mongo-crud  0.000GB
> show collections
people
> db.people.count()
2438
```

### Lab: Import Ingredients and Doctors

On your own, use `mongoimport` to bulk load from `data/doctors.csv` and
`data/ingredients.csv`. Save the commands as `.sh` files and run them from the
terminal (not the Mongo shell!).

## Retrieving Documents from a Collection

- [Querying](https://docs.mongodb.com/manual/tutorial/query-documents/)
  - Overview of retrieving data from MongoDB.
- [Query an Array of Embedded Documents](https://docs.mongodb.com/manual/tutorial/query-array-of-documents/)
  - More detailed overview on retrieving data.
- [Query Operators](https://docs.mongodb.com/manual/tutorial/query-documents/#specify-conditions-using-query-operators)
  - Documentation on operators for retrieving data (ex. greater than is `$gt`)
- [`find`](https://docs.mongodb.org/manual/reference/method/db.collection.find/)
  - Detailed documentation on the `find` collection method.
- [`findOne`](https://docs.mongodb.org/manual/reference/method/db.collection.findOne/)
  - Detailed documentation on the `findOne` collection method.

MongoDB uses JSON natively (technically
[BSON](https://docs.mongodb.org/manual/reference/glossary/#term-bson)), which
makes it well suited for JavaScript applications.  Conveniently, MongoDB lets us
specify the JSON as a JavaScript object.

### Code Along: Read People and Doctors

Together, we'll build a query for our people collection. We'll first find all of
the people in our database.

```js
> db.people.find().pretty()
{
	"_id" : ObjectId("5f8506e2f563f37f60f25d7e"),
	"familyName" : "Rivers",
	"givenName" : "Jonah",
	"height" : 69,
	"weight" : 167,
	"bornOn" : "2000-11-21"
}
{
	"_id" : ObjectId("5f8506e2f563f37f60f25d7f"),
	"familyName" : "Rodriguez",
	"givenName" : "Palmer",
	"height" : 61,
	"weight" : 112,
	"bornOn" : "2012-04-09"
}
...
Type "it" for more
```

> Note: By default `find` only shows the first 20 documents it finds. `find`
  returns a [cursor](https://docs.mongodb.com/manual/reference/glossary/#term-cursor)
  which allows us to type `it` to see the next 20 documents.

Let's see if we can find all people that include "John" in their family name.

```js
> db.people.find({ familyName: /John/ }).pretty()
{
  "_id" : ObjectId("5d0a44038745fb251e970a26"),
  "familyName" : "Johns",
  "givenName" : "Myron",
  "height" : 59,
  "weight" : 106,
  "bornOn" : "2009-02-16"
}
{
  "_id" : ObjectId("5d0a44038745fb251e970cb5"),
  "familyName" : "Johns",
  "givenName" : "Robbie",
  "height" : 59,
  "weight" : 118,
  "bornOn" : "2013-11-02"
}
{
  "_id" : ObjectId("5d0a44038745fb251e9711f1"),
  "familyName" : "Johnson",
  "givenName" : "Corina",
  "height" : 60,
  "weight" : 136,
  "bornOn" : "2008-06-10"
}
```

What about all the doctors who perform surgery?

```js
> db.doctors.find({ specialty: /surgery/ }).pretty()
{
	"_id" : ObjectId("5f85bc1d5db15897bcadcea6"),
	"specialty" : "Oral and maxillofacial surgery",
	"familyName" : "Malone",
	"givenName" : "Scottie"
}
{
	"_id" : ObjectId("5f85bc1d5db15897bcadceaa"),
	"specialty" : "General surgery",
	"familyName" : "Johnston",
	"givenName" : "Shon"
}
```

How about the number of people under 5 feet tall?

```js
> db.people.find({ height: { $lt: 60 } }).count()
368
```

How about finding at most 2 people under 5 feet tall? Lets sort them by the date
they were born on in *descending* order.

```js
> db.people.find({ height: { $lt: 60 } }).sort({ bornOn: -1 }).limit(2).pretty()
{
	"_id" : ObjectId("5f8506e2f563f37f60f25f99"),
	"familyName" : "Marshall",
	"givenName" : "Gaylord",
	"height" : 58,
	"weight" : 116,
	"bornOn" : "2014-04-17"
}
{
	"_id" : ObjectId("5f8506e2f563f37f60f26588"),
	"familyName" : "Rose",
	"givenName" : "Tessie",
	"height" : 59,
	"weight" : 103,
	"bornOn" : "2014-01-24"
}
```

**Note:**   When using the REPL, the `.pretty()` method can be quite helpful.

What do we see?

- MongoDB gave each of our documents a *unique* ID field, called `_id`.
- MongoDB doesn't care that some documents have fewer or more attributes.

### Lab: Read Ingredients

Write a query to get all the ingredients with a unit of `tbsp`.

## Deleting Documents

- [Removing Data](https://docs.mongodb.com/manual/tutorial/remove-documents/)
  - Overview of removing documents from a collection.
- [`deleteOne`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteOne/#mongodb-method-db.collection.deleteOne)
  - Detailed documentation of MongoDB's `deleteOne` collection method.
- [`deleteMany`](https://docs.mongodb.com/manual/reference/method/db.collection.deleteMany/)
  - Detailed documentation of MongoDB's `deleteMany` collection method.
- [`remove`](https://docs.mongodb.org/manual/reference/method/db.collection.remove/)
  - Detailed documentation of MongoDB's `remove` collection method.

If we want to clean up, `db.<collection>.drop()` drops the specified collection
and `db.dropDatabase()` drops the current database.

### Code Along: Delete People and Doctors

There are methods for removing one entry and multiple entries.

Let's remove a person with a specific `bornOn` date.

```js
> db.people.deleteOne({ bornOn: '2013-11-02' })
{ "acknowledged" : true, "deletedCount" : 1 }
```

We can also remove *all* doctors who have "medicine" in their
specialty.

```js
> db.doctors.deleteMany({ specialty: /medicine/ })
{ "acknowledged" : true, "deletedCount" : 3 }
```

### Lab: Delete Ingredients

Remove ingredients that have `ml` as their unit of measure.

## Changing the Data in Documents in a Collection

- [Updating Data](https://docs.mongodb.com/manual/tutorial/update-documents/)
  - Overview of changing documents.
- [`updateOne`](https://docs.mongodb.com/manual/reference/method/db.collection.updateOne/)
  - Documentation of MongoDB's `updateOne` collection method which updates one
      document.
- [`updateMany`](https://docs.mongodb.com/manual/reference/method/db.collection.updateMany/#db.collection.updateMany)
  - Documentation of MongoDB's `updateMany` collection method which updates
      multiple documents.
- [Update Operators](https://docs.mongodb.org/manual/reference/operator/update/)
  - The different modifications we can make during an update.
- [`update`](https://docs.mongodb.org/manual/reference/method/db.collection.update/)
  - Detailed documentation of MongoDB's `update` collection method.

### Code along: Update People and Doctors

MongoDB makes it easy to add an array of items to a document. We'll update
a single person born on a specific date.

```js
> db.people.updateOne({ bornOn: '2013-11-02' }, { $set: { bornOn: '1999-06-08' } })
{ "acknowledged" : true, "matchedCount" : 1, "modifiedCount" : 1 }
```

Now, let's update all doctors whose specialties include *surgery*.

```js
> db.doctors.updateMany({ specialty: /surgery/ }, { $set: { specialty: 'Surgery', hobby: 'Operation the game' } })
{ "acknowledged" : true, "matchedCount" : 2, "modifiedCount" : 2 }
```

What happens if we run an `updateOne` or `updateMany` command without the `$set`
option?

### Lab: Update Ingredients

Update a couple of ingredients' units.

## Adding a Document to a Collection

- [Inserting data](https://docs.mongodb.com/manual/tutorial/insert-documents/)
  - Overview of adding documents to a collection.
- [`insertOne`](https://docs.mongodb.com/manual/reference/method/db.collection.insertOne/#mongodb-method-db.collection.insertOne)
  - Documentation of MongoDB's `insertOne` collection method which inserts one
      document.
- [`insertMany`](https://docs.mongodb.com/manual/reference/method/db.collection.insertMany/#mongodb-method-db.collection.insertMany)
  - Documentation of MongoDB's `insertMany` collection method which inserts
      multiple documents.
- [`insert`](https://docs.mongodb.com/manual/reference/method/db.collection.insert/)
  - Detailed documentation of MongoDB's `insert` collection method.

Next, we'll use the `insertOne` collection method to add another person.  We'll
save our invocations in `scripts/insert/people.js`.  We'll execute that script
using the `mongo` `load` method.  Let's give these people a `middleInitial` or a
`nickName`. Note that the attributes we choose for these people need not match
those from the data we loaded in bulk.

```js
> load('scripts/insert/people.js')
```

### Code along: Insert Doctors

Together we'll add a few doctors with `insertMany`.

### Lab: Insert Ingredients

Add a few ingredients to the `ingredients` collection using `insert`.

## Bonus: Relationships

### Sub-Document

![subdoc](https://docs.mongodb.com/manual/images/data-model-denormalized.bakedsvg.svg)

[Read about sub documents here](https://docs.mongodb.com/manual/core/data-modeling-introduction/#embedded-data)

### Reference

![reference](https://docs.mongodb.com/manual/images/data-model-normalized.bakedsvg.svg)

[Read about references here](https://docs.mongodb.com/manual/core/data-modeling-introduction/#references)

## Additional resources

- [BSON Types](https://docs.mongodb.org/manual/reference/bson-types/)
- [Data Model Examples and Patterns](https://docs.mongodb.com/manual/applications/data-models/)
- [optional GUI client - MongoDB Compass](https://www.mongodb.com/products/compass)
- [Data aggregation](https://docs.mongodb.com/spark-connector/master/python/aggregation/)
  - Overview of summarizing documents.
- [`aggregate`](https://docs.mongodb.org/manual/reference/method/db.collection.aggregate/)
  - Detailed documentation on the `aggregate` collection method.

## [License](LICENSE)

1. All content is licensed under a CC­BY­NC­SA 4.0 license.
1. All software code is licensed under GNU GPLv3. For commercial use or
    alternative licensing, please contact legal@ga.co.
