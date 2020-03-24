# Adding Mongo/Mongoose

Mongoose is a popular MongoDB library for NodeJS that enforces schemas to ensure consistent data.

## Prep for Development

Prior to adding Mongoose to our empty Node application we need a MongoDB and some data to work with.

### Create Database

*We want to start with a database with some dummy data to build against.*

- Install MongoDB (Preferably using Docker)
  - `docker run -d --name temp-mongodb-bookstore -e "MONGO_INITDB_ROOT_USERNAME=uname" -e "MONGO_INITDB_ROOT_PASSWORD=Passw0rd#" -p 27034:27017 mongo`
- Open MongoDb Cli
  - `docker exec -it temp-mongodb-bookstore mongo -u uname -p Passw0rd#`
- Create database: `use bookstoreDb;`
- Create books collection: `db.createCollection('books');`
- Insert dummy records:
	```js
	db.books.insertMany([
	  {'name':'Design Patterns','price':54.93,'category':'Computers',  'author':'Ralph Johnson'},
	  {'name':'Clean Code','price':43.15,'category':'Computers',  'author':'Robert C. Martin'}
	]);
	```
- Run `db.books.find({}).pretty();` to test it works

## Add Mongoose

- Install mongoose
  ```bash
  npm i mongoose
  ```


*To access mongodb we will use the popular mongoose library. We want a setup database script that will allow us to access the Mongo database easily, but also handle the application being closed down forceabilly.*

- Create `src/utils/setupdb.js` file
  ```js
  import mongoose from "mongoose";

  /**
   * Connect to a mongoose database
   *
   * @param {string} mongoDbUri the URI of the MongoDB to connect too. Including credentials
   * @param {string} databaseName the database to switch to on connection
   * @returns {Mongoose} the Mongoose API setup for this application.
   */
  export default (mongoDbUri, databaseName) => {

      const connectionString = createConnString(mongoDbUri, databaseName);

      // Create the database connection
      mongoose.connect(connectionString, { useNewUrlParser: true, useUnifiedTopology: true });

      // Log successful connection
      mongoose.connection.on("connected", function() {
          console.log(`Mongoose connection open to database: ${databaseName}`);
      });

      // If the connection throws an error, log it and then exit the application
      mongoose.connection.on("error", function(err) {
          console.error("Mongoose connection error", err);
          process.exit(0);
      });

      // Log disconnected
      mongoose.connection.on("disconnected", function() {
          console.log("Mongoose connection disconnected");
      });

      // If the user attempts to close the app by terminating it then cleanup and exit
      process.on("SIGINT", function() {
          mongoose.connection.close(function() {
              process.exit(0);
          });
      });

      return mongoose;
  };

  const createConnString = (mongoDbUri, databaseName) => `${mongoDbUri}/${databaseName}?authSource=admin`;
  ```

*We now need to create a db file to be imported throughout our application. Notice how the username and password are included within the url for mongo.*

- Create `src/db.js` file
  ```js
  import setupDb from './utils/setupdb';

  const DB_MONGO_URL = "mongodb://uname:Passw0rd#@127.0.0.1:27034";
  const DB_NAME = "bookstoreDb";

  const db = setupDb(DB_MONGO_URL, DB_NAME);

  export default db;
  ```

## Add First Entity

*We now need to start to add the code required for connecting to the database.*

*We start by adding a **Book Schema**. This is one of the main reasons for using mongoose as it enforces the use of Schemas and protects you from adding data you dont need in a document.*

- Create `src/services/books/book.js` file
  ```js
  import db from '../../db';

  const { model, Schema } = db;

  const bookSchema = new Schema({
  	'name': String,
  	'price': Number,
  	'category': String,
  	'author':String
  });

  const Book = model('book', bookSchema);

  export default Book;
  ```

*We will now introduce a layer in above the Schema. We could expose schemas directly to other areas of the application, however that is not desirable.*

*Creating a wrapper around the mongoose calls is important, yes it restricts the usage in code, but provides a consistent interface for accessing the documents.*

*This also means you also have descriptive code and also it can provide cleanup for the raw values you have.*

*You should only access mongodb through a repo like this.*

- Create `src/services/books/bookRepo.js` file
  ```js
  import Book from "./book";

  export const getBookById = id =>
      Book.findById(id).exec();

  export const getAllBooks = () => {
      return Book.find().exec();
  };

  export const createBook = book => {
      let newBook = new Book(book);

      return new Promise((success, fail) => {
          newBook
              .save()
              .then(success)
              .catch(fail);
      });
  };

  export const updateBook = (id, book) => {
      // Ideally we should get and save, but that is not atomic.
      // Model.updateOne however does partial updates on the document on the server in an atomic manner.
      // See https://masteringjs.io/tutorials/mongoose/update for more details on updating
      return Book.updateOne({ _id: id }, book).exec();
  }

  export const deleteBook = id => Book.deleteOne({ _id: id }).exec();
  ```

*We will now update the `src/index.js` file to display the books from our database.*

- Replace code in `src/index.js` with:
  ```js
  import { getAllBooks } from "./services/books/bookRepo";

  getAllBooks().then(books => console.log(books));
  ```

*You should see a list of books returned from the database.*

- Stop application with `Ctrl+C`