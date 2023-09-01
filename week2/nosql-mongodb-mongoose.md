# NoSQL MongoDB database and mongoose
#### Patrick Ausderau / Ilkka Kylmäniemi

### Study before the classes

* https://youtu.be/VOLeKvNz-Zo
* https://youtu.be/ofme2o29ngU
* https://youtu.be/TbT7eO1fxu
* https://youtu.be/72_5_YuDCNA

---

# Contents

1. NoSQL
1. MongoDB
1. mongoose

---

# NoSQL

* NoSQL = "non SQL", "non relational" or "not only SQL"
* Contains about anything that is not only relational databases
  * Document databases like MongoDB
  * Key-value storages like Redis and Memcached
  * Graph databases like OrientDB and Neo4j
  * Object databases like ObjectDB
  * Tabular databases like Hbase and BigTable

---

# MongoDB

* [Free & open source](https://github.com/mongodb/mongo) document-oriented database
  * First released in February 2009
  * Currently v5.x

__Task 1__
* [Create a free MongoDB Atlas cluster](https://youtu.be/YfyKoMNavs4) (or scroll down to the end of this page)
* Create database 'sssf-labs' and add user with readWrite privileges to it
* Add collections: animal, species and category
  * Add new objects to the collections.
    * category:
    ```JSON
    {
      "category_name" : "Mammal"
    }
  
    ```
    * species:
    ```JSON
    {
     "species_name" : "Zebra",
     "category" : category id from previous insert,
     "image" : "https://upload.wikimedia.org/wikipedia/commons/thumb/e/e3/Plains_Zebra_Equus_quagga.jpg/640px-Plains_Zebra_Equus_quagga.jpg",
     "location" : {
       "type" : "Point",
       "coordinates" : [
         1.3362915,
         24.7976236
         ]
       }
     } 
    ```
    - animal
    ```JSON
    {
      "animal_name" : "Jesse",
      "species" : species id from previous insert,
      "birthdate" : "2016-11-23T00:00:00.000+0000",
    } 
    ```
---

# Main features

* Ad hoc queries
* Indexing
* High availability (replication)
* Supports sharding (scaling horizontally)
* File storage
* JavaScript execution
* Aggregation with map-reduce

---

# Basic structure

* Server has multiple databases
* Database has multiple collections
  * Like tables in relational database
  * Usually contains many related documents
  * Do not enforce schema on documents
  * Starting from 3.2 validations available for documents (checked on inserts and updates)
* Collection has multiple documents (rows)
  * JSON-like structure for collection of key-value pairs
  * Serialized and stored as BSON (Binary JSON)
* Document has multiple fields (columns)
  * Can have different fields from each other within the collection
  * Collection-wide unique identifier (12-byte, stored usually in _<!--_-->id field) can be manually set for document. If not set, one is generated

---

# Modeling data

* Modeling the data is different from relational databases
  * Storage is cheap compared to computing time
  * Duplicating data is acceptable
* Model data by user requirements
* Joins on write, not on read
* Optimize for most usual use case
* Example: Blog post documents stores tags (strings) and comments (objects with text, author etc.) within the blog document itself (with author, text, date, etc.)

---

# Mongoose

* ["Elegant MongoDB object modeling for Node.js"](http://mongoosejs.com/)
* Provides a schema-based solution to model data
* Built-in type casting, validation, query building, business logic hooks etc.
* Methods return a promise
  * All async operations return promises
  * Queries do return object with await (or .then())
---

# Connecting

For example as separated module (utils/db.ts):
```typescript
import mongoose from 'mongoose';
import dotenv from 'dotenv';
dotenv.config();

const mongoConnect = async () => {
  try {
    const connection = await mongoose.connect(process.env.DATABASE_URL as string);
    console.log('DB connected successfully');
    return connection;
  } catch (error) {
    console.error('Connection to db failed: ', (error as Error).message);
  }
};

export default mongoConnect;
```

Then in .env file (make sure to have it in .gitignore):
```dotenv
DATABASE_URL=mongodb://dbUser:pwd1234@127.0.0.1:27017/name-of-database
PORT=3000
```
And in index.ts (or app.ts or server.ts):
```typescript
import express from 'express';
import mongoConnect from './utils/db';

const app = express();
const port = process.env.PORT || 3000;

(async () => {
  try {
    await mongoConnect();
    app.listen(port, () => {
      console.log(`Listening: http://localhost:${port}`);
    });
  } catch (error) {
    console.log('Server error', (error as Error).message);
  }
})();
```
* In case connection is lost, Mongoose automatically reconnects and executes all commands in buffer
* Format for URL is: mongodb://[username:password@]host[:port][/databasename]

__Task 2__
* [Download starter files](https://github.com/ilkkamtk/SSSF_ts_mongo_starter)
* Connect to the `sssf-labs` database

---

# Schema

* maps to a MongoDB collection and defines the shape of the documents

E.g. as a module (models/blog.ts):
```typescript
import mongoose from 'mongoose';
import SomeType from '../path/to/interface'

const Schema = mongoose.Schema;

const blogSchema = new Schema<SomeType>({
  title:  String,
  author: String,
  body:   String,
  comments: [{ body: String, date: Date, author: String }],
  date: { type: Date, default: Date.now },
  hidden: Boolean,
  meta: {
    votes: Number,
    favs:  Number
  }
});

export default mongoose.model<SomeType>('Blog', blogSchema);
```
---

# Schema types

* [Schema](http://mongoosejs.com/docs/guide.html) supports basic [types](http://mongoosejs.com/docs/schematypes.html):
  * String
  * Number
  * Date
  * Buffer
  * Boolean
  * Mixed
  * ObjectId
  * Array
  * ...
* And [validation](https://mongoosejs.com/docs/validation.html):
  * all types have the built-in `required` validator.
  * Numbers have the built-in `min` and `max` validators.
  * Strings have the built-in `enum`, `match`, `minLength`, and `maxLength` validators.
  * and an option to create custom [validators](https://mongoosejs.com/docs/validation.html#custom-validators)

---
# TypeScript support
* [Mongoose Docs](https://mongoosejs.com/docs/typescript.html)

---

# Task 3

Modify existing interfaces (from last week) and then define schemas with following attributes:

Category
* category_name (string, at least 2 characters long)

Species
* species_name (string, at least 2 characters long)
* image (Url), hint: [mongoose-type-url](https://www.npmjs.com/package/mongoose-type-url)
* category (id of the category)
* location (array of [geoJson points](https://mongoosejs.com/docs/geojson.html))
  * idea of this is to save the locations where the species can be found

Animal
* name (string, at least 2 characters long)
* birthdate (Date, is in the past (maximum is `Date.now`))
  * Check ES6 [Date](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date)
* gender (string, male or female)
* species (id of the species)

---

# Model

* To use the schema, we need to create a model associated with it:

```javascript
export default mongoose.model('Blog', blogSchema);
```

* Each instance of model is a document
* Only way to store and retrieve data to/from database

---

# CRUD

* To create a new document, call the [model](http://mongoosejs.com/docs/models.html) `create` or `save` methods
* To modify existing documents, call the `updateOne` method and the `remove` or `deleteOne` to delete
* Documents can be retrieved using models' `find`, `findById`, `findOne`, or `where`
* Specifies which document fields to include or exclude with `select` (or projection)
* and [more](https://mongoosejs.com/docs/api/query.html)

E.g. as a router (routes/blogRoute.ts):
```typescript
import express, {Request, Response} from 'express';
import blog from '../models/blog';

const router = express.Router();

router.route('/')
  .post(async (req: Request, res: Response) => {
    const post = await blog.create({
      title: req.body.title,
      body: req.body.body,
      hidden: false
    });
    res.send(`blog post ${post.title} created with id: ${post._id}`);
  })
  .get(async (req: Request, res: Response) => {
    res.send(await blog.find({ hidden: false }).where('date').gt(new Date(new Date().setFullYear(new Date().getFullYear() - 1))));
  });

router.route('/:id')
  .get(async (req: Request, res: Response) => {
    res.send(await blog.findById(req.params.id));
  })
  .patch(async (req: Request, res: Response) => {
    const mod = await blog.updateOne({ _id: req.params.id }, { title: req.body.title });
    res.status(200).send(`updated sucessfully ${mod.modifiedCount} blog post`);
  })
  .delete(async (req: Request, res: Response) => {
    const del = await blog.deleteOne({ _id: req.params.id });
    res.send(`deleted ${del.deletedCount} blog post`);
  });

export default router;
```

(and in your index.ts)
```typescript
import blogRoute from './routes/blogRoute';
//...
app.use(express.urlencoded({ extended: false })); // for parsing html form x-www-form-urlencoded
app.use(express.json()); // for parsing application/json
app.use('/blog', blogRoute);
```
---

# Task 4

* Expose your models through controller + router:
* Create endpoints `categories`, `species` and `animals`
  * On `POST` create a new category, species or animal
  * Respond to `GET /` with JSON containing all data from category, species or animal
  * On `GET /:id`, find the category, species or animal by id.
  * On `PUT /:id` update the category, species or animal
  * On `DELETE /:id`, delete the category, species or animal.
  * Respond to `GET /species/location` with species located in a area specified by area. E.g  `GET /species/location?topRight=58.05,17.22&lowLeft=49.10,10.32` will return species inside Europe.
    * [Here is a function](https://gist.github.com/ilkkamtk/6f31488eb36868fd2d787eeebfa78729) to covert corners of a rectangle to [geoJSON Polygon](http://thecodebarbarian.com/a-practical-introduction-to-geojson-with-node-js.html#polygons)
* Test with postman.
* Create unit tests.


---

# Instance methods

* Models have some built-in instance methods
* Models can also have custom [instance methods](http://mongoosejs.com/docs/guide.html#methods):

```javascript
// Do not declare methods using ES6 arrow functions (=>).
// Arrow functions explicitly prevent binding this
blogSchema.methods.findFromSameDate = function(cb) {
  return this.model('Blog').find({ date: this.date }, cb);
};
const post1 = new Blog({ date: Date.now(), ... });

const posts = await post1.findFromSameDate();
```

* Built-in instance methods should not be overwritten

---

# Static methods

* [Static methods](http://mongoosejs.com/docs/guide.html#statics) can be used for example to perform more sophisticated lookups:

```javascript
blogSchema.statics.findByTitle = function(title, cb) {
  return this.find({ title: new RegExp(title, 'i') }, cb);
};

const posts = await blog.findByTitle('My title');
```

---

# Query helpers

* [Query helpers](http://mongoosejs.com/docs/guide.html#query-helpers) allow chained query building:

```javascript
blogSchema.query.byTitle = function(title) {
  return this.find({ title: new RegExp(title, 'i') });
};

try {
  const posts = blog.find().byTitle('My title');
} catch(err) {
  console.error('query failed', err);
}
```

---

# Sources

* [NoSQL (wikipedia)](https://en.wikipedia.org/wiki/NoSQL)
* [MongoDB](https://www.mongodb.com/)
* [mongoose](http://mongoosejs.com/index.html)
* [mongoose NPM package](https://www.npmjs.com/package/mongoose)

---

#### Extra: Local installation:
* [Install](https://docs.mongodb.com/manual/administration/install-community/)
* [Shell access](https://docs.mongodb.com/mongodb-shell/) (eventually, some UI like [Studio 3T](https://studio3t.com/download-studio3t-free))
* [Enable authentication](https://blog.tericcabrel.com/enable-authentication-and-authorization-on-mongodb/)
