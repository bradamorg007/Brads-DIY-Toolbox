# Add API Endpoint Testing

Whether doing test driven development (TDD) or not, we should at the least be in the position that when we merge our code into existing code, that a) it is tested and b) it has a high test coverage.

We have built this application without tests so far, purely so we could learn the techniques for building an API in NodeJS without extra confusion. So now we will add testing in retrospectively.

## What you'll learn

- General testing topics
- What to test
- API Testing
- General best practices for testing
  - Recommended test location
- How to setup Jest and Supertest to test Express applications
- How to handle setting up and clearing databases after tests
- Mocking
- Async Testing
- Showing Code Coverage

## What To Test?

I would recommend refreshing your knowledge on Testing and TDD from [this presentation](https://drive.google.com/open?id=1k0vsDLWwYp5rJUC6A5-nPDr-zhaYAePYiiXpTb8k9Ho).


When testing the best practice is to test **all our *own* code and *only* our code**. So basically anything thats not a library/framework created externally. E.g. React.

Many of the tools that we can use to write code tests with include a tool to check code coverage. This can give us a percentage of your code that the tests 'touched' when they ran. Ideally you should push to try to get 100%, never go below 80%.

### Black Box Testing

Best practice is to write tests that test only the external parts of the code, e.g. in a function that would be the parameters passed in when called (inputs) and the returned value (output). We should not be trying to test the internal structure of that code, as it is not important to whether the code works, plus what happens we the code is refactored? So use Blackbox testing where you can.

Whether doing TDD or not, code should be written with testing in mind, in other words 'How would I test this?'. So to test well you should think about **what** the code you are writing really needs to do. What inputs do you need? and what outputs you are expecting? before you even start writing it.

Thinking about the inputs and outputs of your code (classes, functions, libraries and even APIs) prior to writing it, allows you to write better code as well as you have a clear goal. This is required for TDD, but is also helpful in general.

### APIs

For testing APIs we can use a variety of types of testing, unit, integration and e2e testing.

Unit testing is ideal if you have individual pieces of logic that could be split outside of the API into its own library, as these pieces have few external side effects, so fit well as unit tests. TDD would also be recommended for these pieces of code.

As an API is a service that accepts inputs and returns outputs we can think of just testing it from **end to end** (e2e), rather than each individual piece of code that we use to build the API. This is recommended for API as it not only tests that the system works as intended, but also that it works together in harmony, plus it is easier to identify the coverage of the tests. And can reduce the amount of redundant testing as well.

There are drawbacks to this approach is often we are hitting an external service such as a database with an API. You should never hit a live database, or even a database that isnt 'clean' (already had changes) as the results of the tests are not deterministic. **Best practice** is to recreate the database on each test, so that each test works in isolation and can be run in any order. This can be done in the setup and teardown before each test.

Another problem is that it might not be possible to run a database in the test environment, so rather than full e2e testing we would need to 'mock' the database. Mocking is basically 'faking' the functionality so that it simulates the code that is being relied on. This then becomes an integration test, rather than e2e test.

### Setup, Teardown and Scopes

Almost all test runners and frameworks have a concept of before and after. Which is code that runs before tests to set them up and other code to run after tests to clean them up.

Often you can choose the scope of the before/after code so it only effects that scope.

[Scopes](https://jestjs.io/docs/en/setup-teardown#scoping) include:
-  Per test
-  Per file of tests
-  Per describe (grouped tests, can be multiple per file)
-  The all tests

So you can write code to run before and after each test, but also write code that will only run once for a group of tests (normally a suite of tests) or even to run once for the whole application.

**Best practice** is to try to avoid using before and after either on each test or in groups of tests and only minimally using before and after all tests for the application. The better option is to use helper methods that do setup in the test itself, to show there is no magic going on, with no unexpected side effects.

But its ok to use them, we will infact use afterEach and afterAll later on in this tutorial.

### General Best Practice

Remember:
- Use consistent and descriptive test names.
- Test just one thing (try to avoid too many assertions).
- Structure your tests `Arrange, Act and Assert`.
- Avoid testing internal logic directly (i.e. avoid white box testing).
- Make tests work without dependencies on other tests.

## Getting Started

We are going to test our API using e2e tests, with a real Mongo blank database, plus using mocks to simulate authentication.

### Jest and Supertest

We will be using a combination of [Jest](https://jestjs.io/docs/en/getting-started) and [Supertest](https://github.com/visionmedia/supertest) for this project.

Jest is used to run tests, but also has an inbuilt framework of assertion commands, that can test an expected value. Perfect for unit testing logic. It is also used with React and other frontend libraries, so using it means continuation between JavaScript teams.

Supertest is a framework designed specifically for testing a Node application built with Express. It requires a test runner to run, such as Jest.

### Install

- Install
  ```cmd
  npm i babel-jest jest supertest
  ```

### Configure Jest and the Test Environment

- Add the following at the top level of the json document to the `package.json` file:
  ```json
  {
      ...
      "jest": {
          "testPathIgnorePatterns": [
              "dist",
              "build"
          ],
          "testEnvironment": "node"
      },
      ...
  }
  ```

This tells jest to ignore anything that is 'built' by babel (i.e. in the `dist/` folder) as we are testing our source code. _Jest is already applying the settings from the .babelrc file, so it will work with imports/exports etc._

By default jest uses a library called jsdom to handle browser testing which simulates some of quirks of browser based javascript that are not needed for a NodeJS application, so we tell jest to use node directly and avoid jsdom.

***NB:** If using this guide as a reference to setup a way of testing using Jest, then dont forget the section [Fix Async Warning](#fix-async-warning) below to avoid errors.*

### Environment Vars

When we created our application we made sure that we could change some important settings using environment variables, that was partly for production environments but also testing environments.

As well as using a .env file, we can pass values in the command line as well. ***NB:** Environment variables set on the command line override values in an `.env` file*

- Add the 'test' script to the scripts section of the `package.json` file:
  ```json
  "test": "DB_NAME='testBookstoreDb' LOG_LEVEL='warn' npx jest",
  ```

***NB:** If you are using VS Code to debug our code follow this guide to add [debugging to VS Code](./add-vscode-test-debugging.md) which includes the environment variables.*

### Location for Tests

After setting up Jest we can start to create tests. Any file that has the `.test.js` extension will be picked up, but also any file the folder `__tests__` in the root of the application. Its **best practice** to create files with the `.test.js` extension, regardless of whether they are in `__tests__` folder or not.

***Tip:** use a `tests/` folder rather than a `__tests__/` folder, so that you can have any helper or utilitiy functions next to your tests. In a `__tests__/` folder all `.js` files will be considered a test file otherwise.*

### Example Tests

Here are a couple of basic test examples with explanations.

Example:
```js
test("1+1 equals 2", () => {
    // Arrange and Act
    const result = 1+1;
    // Assert
    expect(result).toBe(2);
});
```

This is one of the most basic tests we can create. It declares a test and gives it a name that will be shown along with the test result, best practice is to be be descriptive, and then we create a function to run when the test is run.

In the test above we **assert** that when 1 is added to 1 the result will be 2, which will obviously pass.

In Jest we use `expect(value)` to assert the value is correct or not, `toBe(2);` is described as a [matcher](https://jestjs.io/docs/en/using-matchers). There are many matchers for many different things, like checking the length of an array/string E.g. `.toHaveLength(7);`. Although for most assertions we could just used `toBe();` often its better to use the explicit matcher to make it more obvious what is being checked.

***NB:** To 'negate' the result (anything 'but' 2) you can use `.not` E.g. `expect(result).not.toBe(2);`.*

Example:
```js
import getWeatherReport from './weatherReport';

describe('Weather Reports', () => {

    test("Returns 7 daily reports", async () => {

        // Arrange and Act
        const report = await getWeatherReport();
        // Assert
        expect(report).toHaveLength(7);
    });

    test("First report is today", async () => {

        // Arrange
        const today = new Date();

        // Act
        const report = await getWeatherReport();
        const firstReport = report[0];
        const reportDate = new Date(firstReport.date);

        // Assert
        expect(reportDate.getDate()).toBe(today.getDate());
        expect(reportDate.getMonth()).toBe(today.getMonth());
        expect(reportDate.getYear()).toBe(today.getYear());
    });
});
```

The example is a little contrived but shows how to group together related tests using a `describe` which is structured the same as a test, and combines the name from the describe and the test in the output.

Both tests show that a test can be `async` and use async/await, but tests can also be used with [Promises](https://jestjs.io/docs/en/asynchronous#promises) as well. Another method of handling async behaviour is with the `done` parameter, which we use below.

### Our First Test

Its very important to consider how we want to keep our tests independant regarding the database, but for now we will just look at our first test and add that later in this guide to avoid confusion.

- Create the file `__tests__/healthCheckApi.test.js`:
  ```js
  import supertest from "supertest";
  import app from "../src/app";

  // This will setup the application, including all the middleware and run it
  const request = supertest(app);

  test("Enure health-check returns correctly", (done) => {

      // Arrange
      request
          .get("/health-check")
          // Act
          .send()
          // Assert: Check the status code is correct and the text is as expected
          .expect(200)
          .expect(response => {
              // 'response' has all the same properties that are expected from a real API request
              expect(response.text).toBe('Running...');
          })
          .end((err, response) => {
              // If there was errors, err will be an object if not it will be null. Jest checks the if argument to 'done' is 'truthy' and considers it an error if it is truthy.
              done(err);
          });
  });
  ```

There is a lot going on here, we want to test the API health check endpoint and make sure it works as expected.

In summary:
- We created a supertest object `request` to enable us to hit the API endpoints in the app and test the results.
- We have followed **Act, Arrange and Assert** for testing to keep it clean.
- We used to supertest `expect` functions to assert that the status code returned was correct.
- We tested that the response was certain a format. The assertions here used jest expects, rather than supertest, showing they can be used in combination.
- We have used `done()` to indicate to jest that the async tasks have finished. *Alternatively you can return a promise (directly or via async/await) but done gives us a lot of control.*

***NB:** The [supertest docs](https://github.com/visionmedia/supertest#readme) are very detailed and worth checking out for more options.*

- Run the test with the command:
  ```cmd
  npm test
  ```

### Fix Async Warning

During the last test you should have had the following error:
```cmd
Jest did not exit one second after the test run has completed.

This usually means that there are asynchronous operations that weren't stopped in your tests. Consider running Jest with `--detectOpenHandles` to troubleshoot this issue.
```

This happened as even though you wrote the test correctly and we didnt use the database, the app opened a connection and then the underlying Mongoose connection did not close correctly when supertest finished, so we will handle that now.

As described above in [Setup, Teardown and Scopes](#setup-teardown-and-scopes) we can add code that will run once, after all tests are complete, we will use that to close the connection.

- Create `tests/utils/jest.setup.js` file:
  ```js
  import db from "../../src/db";

  afterAll(async () => {
      await db.connection.close();
  });
  ```

- Add the following to the configuration object for Jest in `package.json`:
  ```json
  "jest": {
      "setupFilesAfterEnv": [
          "./tests/utils/jest.setup.js"
      ],
      ... other settings
  }
  ```

We are telling Jest about the existence of a setup file here, without it Jest will just load each file independantly.

- Run the test with the command:
  ```cmd
  npm test
  ```

The error message should now have disappeared.


### Data

Now we have a small problem, how do we test whether certain requests will return data when we have an empty database? Or how do we know whats going to be returned to assert?

**Best practice** is that tests should run without a reliance on any other test running before it, so we need to tell each test about everything it needs to be able to be successful.

So you should add the data into the database in the 'arrange' part of the test (i.e. at the beginning of the test) and then clear the database as the test finishes.

But that might not always be possible, due to database size etc, so start with the preference of 'before/after each test' and if need be change to 'before/after each group of tests' (or file) and then as a last resort setup and empty a database 'before/after all tests', in that order.

Adding dummy data in a database can be done in a couple of ways:
- Using supertest and inserting the dummy data in via the API
- Using the database schema directly and inserting them via that.

Both have their benefits, the first meaning you are only able to test the API as if you were a client. It can have some limitations, as in bigger systems some processes, like overnight tasks are sometimes not accessible to the API Endpoints. But as we are only testing API Endpoints in this test however we will use the first method.


### Clearing the Dummy Data

With with either option we still have the problem with us adding data into a database and then it remaining and polluting other tests in future.

- Create a `tests/utils/clearDb.js` file:
  ```js
  import db from "../../src/db";

  export default async () =>
      await Promise.all(
          Object.keys(db.connection.collections)
              .map(name => db.connection.collections[name])
              .map(collection => collection.deleteMany())
      );
  ```

This is a generic function that uses mongoose to find all the schema collections (not necessarily all the collections in the DB! just the ones Mongoose knows about) and loops them, emptying each collection of its records.

*__NB:__ Promise.all means wait for all of them to run, and they will run in parallel so it is faster.*

### Insert Tests

Lets start with a test to create a book, as it doesnt require us putting dummy data into the database.

- Create the file `tests/booksApi.test.js`:
  ```js
  import supertest from "supertest";
  import app from "../src/app";

  // This will setup the application, including all the middleware and run it
  const request = supertest(app);

  // After each test ensure the database is clean
  afterEach(async () => {
      await clearDb();
  });

  test("BooksApi - Create a new Book", done => {

      // Arrange: by creating a new book to insert
      const book = { name: "A Book", price: 10, category: "A Category", author: "A   N Other" };

      // Act
      request
          .post("/books")
          .send(book)
          .set("Accept", "application/json")
          // Assert
          .expect("Content-Type", /json/)
          .expect(201)
          .expect(({ body }) => {
              expect(body).not.toBeNull();
              expect(body.id).toBeTruthy();
          })
          .end((err, response) => {
              done(err || undefined);
          });
  });
  ```

This is purely an expanded version of our first test, accept we also test the `Content-Type` header is correct and we are expecting the body to be a JavaScript object.

However if you run the test above it will fail...

### Mocking Authentication

When we created the API we also made several of the endpoints secure by requiring a valid access token in the form of a JWT. Even though we are only testing, the application still requires a valid JWT on each request to those endpoints.

The easiest option is to pass in a valid JWT with a stupidly long expiry date, but the **best practice** is to **mock** (fake) the auth middleware so that it is always successful and test the auth middleware separately and adhere to single responsibility for this test.

- Add the following to `tests/booksApi.test.js`:
  ```js
  jest.mock('../src/auth.js', () => {
      return (req, res, next) => {
          req.user = {
              sub: "5e380c9ba6ca9b4d87573ac9",
              name: "Joe Bloggs",
              email: "test-user@andigital.com"
          };
          next();
      }
  });
  ```

Anytime code imports the `auth.js` file in the application then it will use the returned value from the code above, rather than the actual module.

The auth file itself returns middleware, so to simulate it we just create basic middleware! In this instance we also add dummy user data, as we would expect

Mocking is available in almost every test framework for most languages. `jest.mock()` doesnt require supertest to work, it can be used for any module.

***Important:** The declared file path in jest.mock is relative to the test file.*

### Search for data

As we need to insert dummy data in the database to be test the search functionality, we will start by creating a 'insertDocuments' helper function. It will allow us to insert multiple documents, like books, via the API without too much code.

- Add file `tests/utils/insertDocuments.js`:
  ```js
  /**
   * Helper function to insert a list of objects into the database, via the API.
   */
  const insertDocuments = async (request, url, documents = []) =>
      Promise.all(
          documents.map(doc =>
              request
                  .post(url)
                  .send(doc)
                  .set("Accept", "application/json")
          )
      );

  export default insertDocuments;
  ```

- Add the following test to `tests/booksApi.test.js`:
  ```js
  test("BooksApi - Find two books with Hello in the title", async () => {

      //Arrange
      await insertDocuments(request, '/books', [
          { name: "A Hello world book", price: 10, category: "Programming", author:   "A N Other" },
          { name: "A normal book", price: 5.99, category: "A normal Category",   author: "A N Other" },
          { name: "A hello universe book: fancy", price: 10, category:   "Programming", author: "A N Other" }
      ]);

      //Act
      await request
          .get("/books?name=hello")
          //Assert
          .expect("Content-Type", /json/)
          .expect(200)
          .then(({ body }) => {
              expect(body).not.toBeNull();
              expect(body.length).toBe(2);
              expect(body[0]).toEqual(expect.objectContaining({ name: "A Hello   world book" }));
              expect(body[1]).toEqual(expect.objectContaining({ name: "A hello   universe book: fancy" }));
          });
  });
  ```

***Important:** if we hadnt of used `clearDb()` this would fail on the second run of this test as the numbers would not be correct.*

The only difference between this test and the one previously is we create the books in the database when we prepare the test, so that we can be sure of the contents prior to starting the test.

We dont need to be too concerned about the insert failing as the code will be tested in the 'Create a new book' test.

### Test Authentication

To test authetication, we create a new scope (file) and simply dont mock the auth middleware. At a minimum we then need to test a missing token and a token thats valid.

- Create file `test/authentication.test.js`:
  ```js
  import supertest from "supertest";
  import app from "../src/app";

  const request = supertest(app);

  test("Auth - Ensure that a missing token returns a 401 and message", (done) => {

      // Arrange
      const expected = expect.objectContaining({ message: "No authorization token   was found" });

      // Act
      request
          .get("/books")
          // Assert: Check the status code is correct and the text is as expected
          .expect(401)
          .expect(response => {
              expect(response.body).toEqual(expected);
          })
          .end((err, response) => {
              done(err);
          });
  });

  test("Auth - A valid token should pass authentication", (done) => {

      // Arrange
      // Token that expires in 2030
      const token = "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiI1ZTM4MGM5YmE2Y2E5YjRkODc1NzNhYzkiLCJuYW1lIjoiSm9obiBEb2UiLCJpYXQiOjE1ODA3NDE0MDcsImV4cCI6MTU4MDc0NTMwMiwiaXNzIjoiaHR0cDovL25hbWVvZmF1dGhzZXJ2ZXIiLCJhdWQiOiJib29rc3RvcmUtYXBpIiwianRpIjoiMzQ1YTg4Y2YtZGYxOC00MGU4LTkzMjMtMDM0MzIzZDUyZjRiIn0.UuxftmTq2R6Ua1bZn7a8v0UZugG0lOSGS0reIAMLdN0";
      const expected = [];

      // Act
      request
          .get("/books")
          .set('Authorization', 'Bearer ' + token)
          .expect(200)
          .expect(response => {
              expect(response.body).toEqual(expected);
          })
          .end((err, response) => {
              done(err);
          });
  });
  ```

But ideally we should also check for a token that fails because of:
- The issuer is invalid
- The token is expired
- The audience is incorrect

These are the criteria, along with the secret (password), that we use to secure our application. So we should ensure they are tested completely.

### Code Coverage

Its really simple to get a code coverage result from jest.

- Run the following in the terminal:
  ```cmd
  npm run test -- --coverage --collectCoverageFrom="src/**/*"
  ```

The collectCoverageFrom ensures we dont test anything we are not supposed too, just our code.

### Test Structure

We have split the tests into *booksApi* and *healthCheckApi* using files to seprate each, but we could have also used describes to group them together as well in the one file. Thats not best practice, but sometimes its fits the situation better.

Dont just create a `describe` or file, per test though, as that makes little sense as you can run a test without it, you can always create a 'misc' describe if really needed.

Additionally we have created our tests in a `tests/` folder for this project, there are different options to this, many like to include their tests next to what is being tested, which works well, however it can clutter the codebase and make it harder to find testing utils. But co-location makes it easier to maintain.

There is no hard rule, only best practice, but regardless which style you employ just be consistent in your team.

## Todo

- Add tests for `put`s and `delete`s
- Add tests for boardGames
- Add tests for searching different things
- Add some tests that are designed to fail, e.g. some of the parameters should not validate when creating a book. **Expected failures are as important as expected successes!**
- Read up on [expect](https://jestjs.io/docs/en/expect) in jest and look at [jest-extended](https://github.com/jest-community/jest-extended)
  - Also check out https://medium.com/@boriscoder/the-hidden-power-of-jest-matchers-f3d86d8101b0