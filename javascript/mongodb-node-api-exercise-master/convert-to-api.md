# Convert NodeJS App to an API

*Currently we have a standard console application, but we actually want it to be a Web API.*

*Node natively offers functionality to build an API, however the Express library is really good wrapper that provides additionally functionality like middleware that makes it extremely extensible.*

- Install Express
  ```bash
  npm i express
  ```

*The first API end point we will create is a simple health check that can be pinged to see if the service is up.*

*When creating an API it requires a port number to be able to run on a computer and be accessible to other programs. But as a computer might already have something else running on that port, its good not to **hard code** a port, but provide a way to override it.*

*Start by adding a value to your `.env` file, that we will set the port we will use.*

- Add a port number to your `.env` file:
  ```js
  SERVER_PORT=36789
  ```

*We need to update your `src/config.js` file to include the reference to make it importable throughout your code.*

- Add port number to the config object in `src/config.js`:
  ```js
  SERVER_PORT: process.env.SERVER_PORT,
  ```

*To create our first end point we will create the function, known as a **route handler**, that will run when a particular url is matched.*

- Create a file `src/routes/healthCheck.js` for the route handler:
  ```js
  const healthCheck = (request, response) => {
  	response.send('Running...');
  };

  export default healthCheck;
  ```

*This route handler will do nothing until we reference it from an express application, so thats what we will create next.*

*But all a handler is, is a function that has two parameters, the first contains everything about the request made by the user and the second is the response we want to send back. In this route handler we simply return the text 'Running...' when called.*

*Now we need to create a `app.js` file that creates and builds our application.*

- Create a `src/app.js` file:
  ```js
  import express from 'express';
  import healthCheck from './routes/healthCheck';

  const app = express();

  app.get("/health-check", healthCheck);

  export default app;
  ```

*After creating the app, the `get` function adds the end point, so when the API is hit with the a url matching the first parameter (E.g. `http://localhost:36789/health-check`) then it runs the second parameter, which in this case is the healthCheck we wrote earlier.*

*To run the application, we need to tell express to listen on the port we desire.*

- Replace the `src/index.js` file with:
  ```js
  import app from "./app";
  import config from './config';

  const port = config.SERVER_PORT;

  app.listen(port, () => console.log(`Listening on port ${port}`));
  ```

***NB:** The reason we dont start the application in the `app.js` file is because in some scenerios we would want to start the application differently depending on the sitation e.g. testing. Plus it is good practice to keep your 'creating' code separate from 'running' your code.*

- Run the code
  ```
  npm run start:dev
  ```

- Use curl to test the API
  ```
  curl http://localhost:36789/health-check -i
  ```