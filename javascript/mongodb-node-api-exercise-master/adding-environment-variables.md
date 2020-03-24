# Use Environment Variables with DotEnv

A good way to use env files is to use the popular dotenv library to enable us to use .env files, as well as OS Environment Variables. Its is also in Create React App by default.

- Add dotenv package
  ```
  npm i dotenv
  ```

- Create a `.env` file in the root of the application.
  ```
  DB_MONGO_URL=mongodb://uname:Passw0rd#@127.0.0.1:27034
  DB_NAME=bookstoreDb
  ```

***Important:** Always exclude `.env` files from source control, usually with a .gitignore file.*

*Its useful to understand that we need to run the config() function on the dotenv package to ensure the values are loaded as expected.*

*We will create a wrapper file for config from environment variables, this will reduce the use of `process.env` throughout the application and also will allow you to provide default values easily and in one place.*

- Create `src/config.js` file
  ```js
  import dotenv from 'dotenv';

  dotenv.config();

  const config = {
    DB_MONGO_URL: process.env.DB_MONGO_URL,
    DB_NAME: process.env.DB_NAME
  };

  export default config;
  ```

*We now need to update the `src/db.js` file so it accepts the values from config.*

- Replace code in `src/db.js` with:
  ```js
  import setupDb from './utils/setupdb';
  import config from './config';

  const { DB_MONGO_URL, DB_NAME } = config;

  const db = setupDb(DB_MONGO_URL, DB_NAME);

  export default db;
  ```

*You can supply now change the database used though environment variables*

- Run app to ensure it still works
  ```bash
  npm start
  ```