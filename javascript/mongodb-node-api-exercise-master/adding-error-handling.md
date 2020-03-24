# Add Global Error Handling

*In a perfect world, we would never make mistakes and we would never make errors, but in the real world thats impossible. Sometimes the error isnt even down to us.*

*When we develop any type of code we should always try and catch any errors. We can also be clever and handle different types of error in different ways, for example with the appropriate status code response.*

*One of the worst things we can do is allow errors to reach our customers unfiltered, not only is it a bad user experience, but we could inadvertently expose sensitive information about our application, or even about the users ourselves.*

*Therefore we try to catch errors on every route, this can get tedious to do in every route though and can sometimes be missed.*

*Example:*
```js
router.get("/", (req, res) => {

    try {
        // Your handler code...
    }
    catch(e) {
        log.error(e);
        res.status(500).send({ error: 'Internal Server Error' });
    }
});
```

*In the example above we have attempted to catch an errors and return a generic 500 message, after logging it, but what if you forget to add it? Or you need to change the generic message?*

__NB: Did you notice that when we added the 'get book by id' route it is missing a try/catch? 🤔__

### Create Error Middleware

*A better way to do this is with middleware, that will catch any errors in the pipeline and routes and return a generic error message from that.*

- Create a `src/utils/createErrorObject.js` file:
  ```js
  /**
   * Create an object that can be returned to a client in a consistent format.
   * Use this to create any error response objects for the user/client.
   *
   * @param {string} message The error message to give to the client
   * @param {number} statusCode The status code to match the response statusCode, for convenience
   * @param {?object} data (Optional) additional data to add to the object
   */
  export const createErrorObject = (message, statusCode = 500, data = undefined) => {

  	let output = {
  		statusCode: statusCode,
  		message: message
  	};

  	if (data) {
  		output.data = data;
  	}

  	return output;
  };

  export default createErrorObject;
  ```

*This function will allow us to create consistent error objects, in a 'shape' that is expected throughout the application, we can then use it directly in our middleware, or anywhere else it is needed.*

- Create a `src/utils/sendGenericErrorResponse.js` file:
  ```js
  import log from '../log';
  import createError from './createErrorObject';

  /**
   * Excepts an exception, logs it and then sends a generic 500 response to the caller.
   *
   * @param {Error} error The error to log
   * @param {Express.Response} response The response object to use to send the error message.
   */
  const sendGenericErrorResponse = (err, res) => {
  	log.error(err.message, err);
  	res.status(500)
  		.json(
  			createError('Internal Server Error', 500)
  		);
  };

  export default sendGenericErrorResponse;
  ```

*This function actually sends the generic error message back to the client that called the API. Extracting this function from the middleware below would be overkill normally, however we will need to use it elsewhere in a minute*


- Create a `src/utils/errorHandlerMiddlware.js` file:
  ```js
  import sendGenericErrorResponse from './sendGenericErrorResponse';

  /**
   * Handles any SYNC errors in the express stack and returns an error object   for the client to consume
   * @see https://expressjs.com/en/guide/error-handling.html
   *
   * @param {string|Error} error The error obejct caught by express
   * @param {Request} request  The request object
   * @param {Response} response  The response object
   * @param {function} next  A function to 'call the next middleware' in the   chain
   */
  const errorHandlerMiddlware = (error, request, response, next) => {
  	sendGenericErrorResponse(error, request, response);
  };

  export default errorHandlerMiddlware;
  ```

*This is the actual middleware we are going to attach to the application.*

***Important:** the middleware above actually gets **FOUR** arguments. If express see's that a middleware function has four arguments it makes it a error handling middleware and only runs if there is an error, plus passes the error as the first argument. The other arguments are the same. This behaviour is unique to error handling middleware. All other middleware recieves three values.*

### Add Middleware to App

*You then need to add the middleware in the same way as we added logging. However we need to add it 'last' after the other middleware as it doesnt call to the next, but just returns a response.*

- Replace `src/app.js` with:
  ```js
  import express from 'express';
  import healthCheck from './routes/healthCheck';
  import books from './routes/books';
  import loggerMiddleware from './utils/loggerMiddleware';
  import errorHandlerMiddlware from './utils/errorHandlerMiddlware';

  const app = express();

  // Enable parsing of request bodies to json
  app.use(express.json());

  // Add logging to each request
  app.use(loggerMiddleware);

  app.get("/health-check", healthCheck);
  app.use('/books', books);

  // Add synchronous catch all for errors, needs to be 'last' after other     middleware and routes
  app.use(errorHandlerMiddlware);

  export default app;
  ```

*Yay we've finished!... unfortunately not.*

### Async Routes

*This will capture all errors that are thrown from synchronous routes and middleware.*

*Currently express has a problem, all of the routes we have added so far have been asynchronous and will not get captured by middleware as they are out of the flow of control.*

*Despite express handling async route handlers it doesnt fire middleware against async handlers due to Promises being 'micro tasks'.*

*__NB:__ This is on the express road map to fix and hopefully will in the next version (5) of express.*

- Create `src/utils/asyncHandler.js`:
  ```js
  import sendGenericErrorResponse from './sendGenericErrorResponse';

  /**
   * Wraps an async express route handler, to catch and handle expected errors
   *
   * @param {function} handler An express route handler
   */
  export const asyncHandler = (handler) => (req, res, next) => {
  	return handler(req, res, next)
  		.catch(err => sendGenericErrorResponse(err, res));
  };
  ```

We then need to apply this to each of our async route handlers. Eg:
```js
router.get("/", asyncHandler(async (req, res) => {

    const { name } = req.query;

    const books = await search(name);

    res.status(200).send(books);
}));
```

*Although this is similar to what we had before (try/catch) it is easier to read/process and will make removing the handlers easier once express supports it natively. Also it ensures the error message is always returned in the same format.*

*This will then captch any errors in a catch and reply to the client with the same error message as sync endpoints will.*

### Todo

- Follow the steps above to add the global error handler.
- Add an `asyncHandler()` wrapper to each async route (in books and boardGames)