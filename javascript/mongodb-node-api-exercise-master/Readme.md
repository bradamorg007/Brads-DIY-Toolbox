# Mongo/NodeJs Exercise

We are going to make a (very basic) Bookstore API!

It will be built using NodeJS and MongoDB. We will also be using the **Express** and **Mongoose** libraries that are very popular in the industry.

## Stage 1 - Create basic NodeJs application

Follow this [guide](./create-node-application.md) to create a basic NodeJS application that can be run from the command line and VS Code.

## Stage 2 - Add Mongoose library

Use this [guide](./add-mongoose.md) to create the initial setup code for using the popular Object Data Modelling (ODM) library that allows NodeJS applications to access Mongo Databases in a structured way.

## Stage 3 - Adding Environment Variables

Follow this [guide](./adding-environment-variables.md) to switch to using Environment Variables using dotenv.

## Stage 4 - Convert to an API application using Express

Convert the application to a Express API application using the popular Express library with this [guide](./convert-to-api.md).

## Stage 5 - Add REST Resource

Follow this [guide](./add-rest-resource.md) to add a book resource to our Bookstore API.

## Stage 6 - Search by name

The product owner has asked that as well as being able to return all books, they also need to be able to do a case insentive search on book names.

*__NB:__ I have a mini video explaining how this can be done for you to watch after you have attempted it yourself*

## Stage 7 - New resource

The product owner tells us as well as selling books, the book store will now start selling board games as well. The properties that need to be stored for board games are similar to books. So name, price, category and also the minimum and maximum no of players they board game is designed for. Also allow for searching by name as well.

*__NB:__ I have a mini video explaining how this can be done for you to watch after you have attempted it yourself*

## Stage 8 - Search by player count

Add the ability to search for games that are suitable for people of a particular number of players.

## Stage 9 - Logging and Middleware

Follow the [guide](adding-logging.md) to include logging in your API and learn about Middleware.

## Stage 10 - Error Handling

Follow the [guide](adding-error-handling.md) to include catch all error handling to your API.

## Stage 11 - Validating API Endpoints

Follow the [guide](adding-api-validation.md) to restrict your API from incorrect data.

## Stage 12 - DTOs, View Models and Projections

Follow the [guide](adding-view-models.md) to prevent exposing data you shouldnt.

## Stage 13 - Documentation

Follow the [guide](adding-api-documentation.md) to give your consumers documentation via swagger.

## Stage 14 - Authorisation

Follow the [guide](./add-authorisation.md) to make your endpoints more secure.

## Stage 15 - E2E Testing

Follow the [guide](add-api-testing.md) to add API endpoint tests to your api.

## Stage 16 - Docker Integration - IN PROGRESS

Developing in containers has many benefits but can be a little tricky to setup with so many options. Follow the [guide](add-docker-integration.md) to implement it in your bookstore.


## Extra

- [What is Middleware?](./what-is-middleware.md)