# Adding API Documentation - Swagger JsDoc

We can also generate the OpenAPI documentation using Jsdoc comments in your code and a plugin called [swagger-jsdoc](https://github.com/Surnet/swagger-jsdoc).

You still need to be writing yaml definitions for your end points, but being co-located in your code makes it easier to remember to remove documentation or change it when you are working on on that code.

## 1. Create Specification Document

- Install swagger doc:
  ```cmd
  npm i swagger-jsdoc
  ```

On each route we add an OpenAPI definition. Below is a definition we can add to the get all books endpoint.

Preceeded by the `@swagger` tag the definition is the same format as it would be under the `paths:` node in a openapi.yml file. Defintions are explained [here](https://swagger.io/specification/#schema).

- Add to the Get All Books end point in `src/routes/books.js`:
  ```js
  /**
   * @swagger
   * /books/:
   *  get:
   *    tags:
   *      - Books
   *    summary: Get all books
   *    parameters:
   *      - name: name
   *        in: query
   *        description: >-
   *          (Optional) Filter results where the name contains this value (case
   *          insensitive)
   *        schema:
   *          type: string
   *      - name: author
   *        in: query
   *        description: >-
   *          (Optional) Filter results by where the name of the author matches
   *          this value
   *        schema:
   *          type: string
   *    responses:
   *      '200':
   *        description: Success
   *        content:
   *          application/json:
   *            schema:
   *              type: array
   *              items:
   *                $ref: '#/components/schemas/Book'
   */
   router.get("/", asyncHandler(async (req, res) => {

  		const { name } = req.query;

  		const books = await search(name);

  		res.status(200).send(books);
  	})
  );
  ```

We have stated that this end point accepts an optional parameter, name, and states it returns an array of `Book` in the format of `application/json`.

But erm... what is a `Book`?

We could have put the book definition directly in the yaml above, but as we are likely to be using it throughout the router, we can declare it once in the file and then reference it anywhere else, it will get picked up because of the `@swagger` tag.

```js
/**
 * @swagger
 * components:
 *  schemas:
 *    Book:
 *     type: object
 *     properties:
 *       id:
 *         type: string
 *       bookName:
 *         type: string
 *       price:
 *         type: number
 *         format: double
 *       category:
 *         type: string
 *       author:
 *         type: string
 *     additionalProperties: false
 */
```

You need to repeat this for all the end points in your code. That seems tedious, but its exacly the same as if you had done it manually in the swagger editor.

## 2. Add UI

Instead of the manually created OpenApi file, you can use the output from `swagger-jsdoc` directly. We need to wire it up and tell it where to find the routes we have added.

- Create file `/src/openapi.js`.
  ```js
  import swaggerJsDoc from 'swagger-jsdoc';

  const options = {
  	definition: {
  		openapi: '3.0.1',
  		info: {
  			title: 'Books R Us API',
  			version: 'v1'
  		}
  	},
  	// Base path (optional)
  	basePath: '/',
  	// apis refers to the location of swagger jsdocs and the routes (required)
  	apis: [
  		require('path').join(__dirname, '/routes/*.js')
  	]
  }

  export default swaggerJsDoc(options);
  ```

- Create file `src/routes/documentation.js`
  ```js
  import { serve, setup } from "swagger-ui-express";
  import swagger from "../swagger";

  export default [serve, setup(swagger)];
  ```

- We then need to link it with the app.
  ```js
  import app from "./app";
  import books from './routers/books';
  import boardGames from './routers/boardGames';
  import documentation from './routers/documentation';

  app.use('/books', books);
  app.use('/boardgames', boardGames);
  app.use('/', documentation);
  ```

## TODO

If using `swagger-jsdoc` then:
- Add documentation for all books and board games endpoints. ***NB:** Look at [openapi.json](./openapi.json) to help you.*
- Test endpoints interactively in Swagger UI.