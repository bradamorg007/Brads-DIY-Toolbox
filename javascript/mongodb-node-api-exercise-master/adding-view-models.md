# DTOs, View Models and Projections

As discussed in clean code and also when learnin about APIs, you should never expose more data from anything than is needed, or accept more than you want. Because you could be allowing a

In any programming if we expose more than we mean to there is a good chance that a developer (using your library or function) or a client (using a service like and API) will start using it. That results in restricting your ability to refactor as you need to take that exposed data into acount. But also it can mean expose implementation details as well.

Because JavaScript is not type safe it can be even more important to be explicit on what you expose, as it can be hard to see.

## Terms

Here are a couple of related terms we will use below.

### Projection (or Mapping)
To take data of one particular shape and change it into another shape. Otherwise known as mapping E.g.

```js
function change(person) {
	return {
		"name": person.firstName + ' ' + person.lastName;
	};
}

console.log(
	change({ "firstName": "Joe", "lastName": "Bloggs" })
);
// { "name": "Joe Bloggs" }
```

### POJO
A `Plain old Java Object` a POJO is basically an object in Java that only contains related data, this also applies to JavaScript so we can steal POJO to mean 'Plain Old JavaScript Object' ;) They are also present in C# as POCOs.

```js
const myPojo = {
	"firstName": "Joe",
	"lastName": "Bloggs"
};
```

### DTO
`Data Transfer Object` a DTO is essentially just POJOs used to transfer data around an application.

As we dont want to expose more than we need too, we map (project) an object into a new object that only has properties that you want to expose before passing it to another part of the application.

A good example of this is your database models, quite often the data structure in the database is not the same as what is needed in the application. So we create layer in between that will first do the database process and then map the return object to whatever called that layer.

It creates a dependancy on the domain logic and often implementation, so that your code isnt as decoupled as you think. Plus remember single responsibility, if you are simply using your database models and you change them, code throughout the program changes behavior, or you expose sensitive data without meaning too.

## View models

A View Model is a term thats used to describe the data models/structure exposed by API endpoints. But they are effectively just DTOs.

(It is also used in MVC when passing the data needed to a view layer, after all the logic is calculated).

If the data returned was just the same structure as the database model and we expose it via the API we can't change the name of a database property before the API version has gone up, otherwise we can break our user contract unintentionally.

So we use View Models/DTOs with Projection to avoid that.

## In Practice

Examples of sensitive information we dont want to expose are user details, such as a password or date of birth. Or underlying information about how the application is built.

Currently we dont have much senstive information in our application, however as we are using MongoDb and Mongoose we need to map our data to remove the **_v** (version property) and rename **_id** to id, as it is giving away how our application is built, along with an implementation detail and also its a little unexpected.

We have 2 choices, we can use projection from MongoDb or we can project the data after we get it from the database.

### Use the Database for Projection

If we are wanting to return a lot less fields than is in the document in Mongo I would recommend using projection.

Example:
```js
Book.find().select('name author').exec()
```

See [here](https://mongoosejs.com/docs/api.html#query_Query-select) for more information.

### Use Application Projection

However we are restricted when doing that with Mongoose, more than with Mongo directly, so it can be easier to project your data once its returned from Mongoose, we then have the complete ability to rename to anything we want.

```js
Book.find()
	.exec()
	.then(data =>
		data.map(book => ({
			id: book._id,
			name: book.name,
			author: book.author,
			catgory: book.category,
			price: book.price
		}))
	);
```

### Best Practice

Best Practice is a hard one to judge here, try to limit the amount of data thats being sent over the network, but if its only a small change then do it in the application.

You are also not restricted from combining them:
```js
Book.find()
    .select('name author')
	.exec()
	.then(data =>
		data.map(book => ({
			id: book._id,
			...book
		}))
	);

// result:
// {
//	id: '56657J324234234',
//	name: 'Terry Pratchett',
//	author: 'Guards Guards'
//}
```

### Incoming as well as Outgoing.

**IMPORTANT:** We should **not** only do this with outgoing data, but also incoming data as well.

Imagine we had renamed `minimumPlayers` to `minPlayers` in the API? so that the edited version coming in would be wrong as well. Or if we hadnt used Mongoose? Then any additional fields we didnt want would be added to our document in our DB, which raises the risk of injection attacks.


## Todo

- Add projection to remove the `__v` field and rename the `_id` field to `id` to the different repo functions for both books and also board games.
