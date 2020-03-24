# Adding Validation

*APIs are great for excepting values and returning results, but what do we do if someone sends us the wrong type of data?*

*We should always protect our APIs by validating the data being sent to us. This also enables a better user experience as well as it means we can let them know 'why' the request failed.*

*When using express there are several libraries for validation, but one of the easiest, most popular and mature is [express-validator](https://express-validator.github.io/docs/).*

### Middleware

***Please** refresh your knowlegde on Middleware with [What is Middleware?](./what-is-middleware.md), especially the section on `Using Middleware`.*

### Express Validator

*We need to start by installing `express-validator`.*

- Install express-validator into your project:
  ```cmd
  npm i express-validator
  ```

*`express-validator` takes advantage of being able to add multiple middlewares to a route.*

*Each middleware only validates one piece of data, so we add multiple validators as needed depending on what data is coming in.*

Example:
```js
import asyncHandler from '../utils/asyncHandler';
import { param, validationResult } from 'express-validator';

const validateById = [param("id").isMongoId()];

router.get('/something/:id', validateById, asyncHandler(async (req, res) => {

	const errors = validationResult(req);
	if (!errors.isEmpty()) {
		return res.status(422).json({ errors: errors.array() });
	}

    // handle request
	const { id } = req.params;

    const book = await getBookById(id);

    if (!book) {
        res.status(404).send();
    } else {
        res.send(book);
    }
}));
```

*Here we use the `param` function from express-validator. The param function allows us to validate any data that appears on the `req.params` object, which should be set from the incoming url. In this case `id` which should have the same format as a MongoDB id, we will discuss checking the incoming format later.*

***NB:** There are also functions for `body` and `headers` that can validate those parts of the request as well.*

***WARNING:** There is an additional function `check` which checks for the a value for that name in all areas of the imcoming request. **Avoid** it, as it can result in unexpected errors and behaviour, as the value could be set in an unexpected way. Always try to be explicit.*

### Handle Errors

*`express-validator` **does not** reject automatically, it places an 'report' on the incoming request object, so we have to check the values in the incoming request and reject them if invalid.*

*Putting that into almost every route would get annoying and messy, but custom middleware to the rescue!*

- Add the follow`src/utils/validateHandler.js` to your application:
  ```js
  import { validationResult } from "express-validator";
  import createErrorObject from "../utils/createErrorObject";

  /**
   * Creates a middleware by grouping any number of express-validator checks (param, body,etc)
   * and validating the incoming values against them. If the validation of the checks fail the
   * request is rejected with an error message, without running the next middleware/ handler.
   *
   * @param  {...any} checks
   */
  export const validate = (...checks) => {
      return [
          ...checks,
          (req, res, next) => {
              const errors = validationResult(req);

              // As we have no errors continue on to the next middleware or route   handler
              if (errors.isEmpty()) {
                  next();
                  return;
              }

              const statusCode = 422;
              const errorObject = createErrorObject(
                  "Unprocessable Entity",
                  statusCode,
                  errors.array()
              );

              res.status(statusCode).json(errorObject);
          }
      ];
  };
  ```

*If you read the comment for the functon you can see this is a function that accepts multiple validation checks (`param`, `body` or `header`) and then returns an array, that can be assigned to the end point.*

*It also adds a 'final' middleware to the array that will reject the request if there were any validation errors found.*

*We can now use this middleware to make our routes cleaner.*

- Replace the get by id endpoint in `src/routes/boardGames.js` with:
  ```js
  const validateById = validate( param("id").isMongoId() );

  app.get("/:id", validateById, asyncHandler(async (req, res) => {

          const { id } = req.params;

          const boardGame = await getBoardGameById(id);

          if (!boardGame) {
              res.status(404).send();
          } else {
              res.send(boardGame);
          }
      })
  );
  ```

### Using Multiple Checks

*So far we have only used a single param check, but we can also check against the body or headers as well and multiple values in each request.*

*The following is an example of the validator for boardGames being POSTed to the API.*

- Replace the post new board game endpoint in `src/routes/boardGames.js` with:
  ```js
  const validatePost = validate(
      body("name").exists(),
      body("price").isDecimal(),
      body("category").exists(),
      body("minimumPlayers").optional().isNumeric({ no_symbols: true }),
      body("maximumPlayers").optional().isNumeric({ no_symbols: true })
  );

  router.post(
      "/",
      validatePost,
      asyncHandler(async (req, res) => {
          const newBoardGame = req.body;

          const boardGame = await createBoardGame(newBoardGame);

          res.status(201).send(boardGame);
      })
  );
  ```

*Breaking down the code above we are validating each property of the object coming in, we are simply expecting `name` and `category` to exist, `price` also needs to exist, however it also has to be in the format of a decimal, min/max players are optional values, but if we do recieve them then they to be numeric.*

***IMPORTANT:** You are not restricted in the validators you can use. You can use `body()` and `param()` etc together.*

### Builtin Validators *isXXX()*

*`express-validator` uses the library `validate.js` internally to handle the format functions, e.g. `isDecimal()`, `isNumeric()`, `isMongoId()`, `isEmail()` etc.*

*See [this list](https://github.com/validatorjs/validator.js#validators) of validators for more information.*

### Todo:

- Apply validation to each end point that accepts data in your API (both in Board Games and Books.
- Think about DRYer version of creating validators for POST and PUT, as they are 'almost' identical.

### References:

- https://express-validator.github.io/docs/check-api.html
- https://github.com/validatorjs/validator.js#validators