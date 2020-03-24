# Add REST Resource - Books

We have an API that can connect to a Mongo database containing records of Books and an API we can ping to see if the API is working, however we currently dont have a way of combining them.

We will be creating RESTful end points based around resources, so using HTTP Methods such as GET, POST, PUT and DELETE to manage a resource, in our case books.

When doing REST we group resources by URL, e.g. for books we would use the `/books` url. We could add each **get/post/put/delete** call directly on to the app object, and that would work fine, however we can group them together with a `Router` object and add them to the app all at once.

Hopefully it should make more sense when we add the code.

### Create Router

- Create `src/routes/books.js` file which will contain our **book** endpoints:
  ```js
  import express from 'express';
  import {
      getBookById,
      getAllBooks,
      createBook,
      updateBook,
      deleteBook
  } from "../services/books/bookRepo";

  // Create a new router to handle the books resource
  const router = express.Router();

  // NB: This is where we will add end points

  // Export the router ready to be imported into an app.
  export default router;
  ```

*We will add the end points in a minute, but this shows the simple process of creating and exporting a new **Router** object.*

- Replace the code in `src/app.js` with:
  ```js
  import express from 'express';
  import healthCheck from './routes/healthCheck';
  import books from './routes/books';

  const app = express();

  app.get("/health-check", healthCheck);
  app.use('/books', books);

  export default app;
  ```

*Instead of using the `get` function like we did before, we use the `use` function to combine it with the app. This is because a Router is **middleware** rather than a route (we will mention middleware more in the logging stage).*

*By giving our books a first parameter of `"/books"` any endpoints in the Router will automatically be prefixed with `/books` making it very portable.*

### Get All Books

- Add the following endpoint to `src/routes/books.js`:
  ```js
  router.get("/", async (req, res) => {
      const books = await getAllBooks();

      res.status(200).send(books);
  });
  ```

- Run the code
  ```
  npm run start:dev
  ```

- Use Curl to test the end point
  ```
  curl -i http://localhost:36789/books
  ```

*You should now be able see a list of books returned from the API... cool*

*Because the end point has an async action, in this case `getAllBooks()` we either use a Promise and then or we make the whole router handler async. In this case we made it async. So it will wait for the books to be returned before sending a response.*

### Get By Id

- Add another endpoint to `src/routes/books.js`:
  ```js
  router.get("/:id", async (req, res) => {
      const { id } = req.params;

      const book = await getBookById(id);

      if (!book) {
          res.status(404).send();
      } else {
          res.send(book);
      }
  });
  ```

*For this end point we only want to get a single book by its id. This end point accepts a paramater called `id` that is stated in the url by `:id`. When the route handler runs it passes in the value found at id to the params object of the request (req in the code above).*

***E.g.** `http://localhost:36789/5e21c32d468398f3db63a97b` would mean `req.params.id` would equal `5e21c32d468398f3db63a97b`.*

We can then use that to get the book we want. However if it isnt found, then we should return a `404` response (using res in the code above) to inform the client of why it failed.

***Important:** For which urls you can use in express, like with React Router, express uses the excellent library [path-to-regexp](https://www.npmjs.com/package/path-to-regexp) so worth checking their docs. Also there is funky tool someone created for testing urls [express-route-tester](http://forbeslindesay.github.io/express-route-tester/).*

### Delete

- Add another endpoint to `src/routes/books.js`:
  ```js
  router.delete("/:id", async (req, res) => {

  	const { id } = req.params;

  	await deleteBook(id);

  	res.status(204).send();
  });
  ```

*Almost identical to the getById endpoint, although it can be argued that a 404 is not needed, but to the action being performed. Therefore if it doesnt exist do you care whether it was deleted by you or another request?*

### Put and Post

*Both the Put and Post endpoints require an extra step, this is because instead of accepting a value on the url, they are expecting a value in the body of the request.*

- Replace the `src/app.js` file with:
  ```js
  import express from 'express';
  import healthCheck from './routes/healthCheck';
  import books from './routes/books';

  const app = express();

  // Enable parsing of request bodies to json
  app.use(express.json());

  app.get("/health-check", healthCheck);
  app.use('/books', books);

  export default app;
  ```

*By adding the middleware `app.use(express.json());` we are telling express that when it handles any request with the content type set to `json` that it should parse that body into an object and make it available to the route handler.*

- Add another endpoint to `src/routes/books.js`:
  ```js
  router.post("/", async (req, res) => {
      const newBook = req.body;

      try {
          const book = await createBook(newBook);
          res.status(201).send(book);

      } catch (e) {
        console.log(e);
        res.status(500).send({
            error: 'Internal Server Error'
        });
      }
  });
  ```

*Here we are accepting a post request where the body is the new book that someone wants to add to our database. Because we added the middleware above, we can already assume that `req.body` will contain our book... accept it might not, it might incorrect/invalid data, but we handle that in the stage about validation.*

- Add another endpoint to `src/routes/books.js`:
  ```js
  router.put("/:id", async (req, res) => {

  	const book = req.body;
  	const { id } = req.params;

      try {
          await updateBook(id, book);
          res.status(204).send();

      } catch (e) {
          console.log(e);
          res.status(500).send({
            error: 'Internal Server Error'
        });
      }
  });
  ```

*The put request is almost identical to the post, except it accepts a `id` in the url as well, to idenitfy it. Also we return 204 instead of 201 in response. But the behaviour is the same.*

*Your resource should now have 5 end points for handling books via an API.*

- Run the code
  ```
  npm run start:dev
  ```

*NB: To test endpoints such as put and post you will need to use a tool like [postman](https://www.getpostman.com/) or Curl rather than the browser.*

