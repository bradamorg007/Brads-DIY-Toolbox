# Docker

Would it not be brilliant to just run the one command from the command line and have it setup your databases and your server? And if we had one even our frontend?

With **Docker** and containers we can, but it can take a bit of extra work, but once its done its done.

*__Warning:__ Before starting I have presumed you have been using the following folder structure. We talked about it when first doing node.js as it separates your app code from the git folder meaning it cant be exposed by accident.*

```file
root/
 ├── .git/
 ├── bookstore-api/
 │		├── .vscode/
 │ 	 	├── src/
 │		├── tests/
 │		├── package.json
 │		└── ...other
 └──.gitignore
```

So far we have been working exclusively in the bookstore-api folder, but the next files need to be added from the root folder. If your structure does not look like this, change it so that is at least similar.

## Improve Scripts

The first stage is to improve the scripts in the `bookstore-api/package.json` file so that we can do more things inside a container.

*__NB:__ This could be done regardless of the containerisation, in fact build scripts can help in many scenerios, including as documentation to how to interact with the app!*

```json
{
	...
	"scripts": {
        "prestart": "npm run build",
        "start": "node dist/index.js",
        "start:dev": "nodemon --exec babel-node src/index.js",
        "test": "DB_NAME='testBookstoreDb' LOG_LEVEL='warn' npx jest",
        "build": "npm run build:clean && babel src -d dist -s && npm run build:copy",
        "build:clean": "rm -rf dist",
        "build:copy": "find src/ -type f -name '*.json' -exec cp '{}' dist/ ';'"
    },
	...
}
```

Phew we have added a lot; we have created a new `start` script and renamed the original to `start:dev`. This means that the default start task is to run the production version of the code, but we provide a dev version as well to help us when we want to do development.

We have also introduced a `prestart` hook, which is built into npm/yarn to run prior to `start`, which ensures our code is built prior to starting.

Additionally we have split `build` into 3 parts, with descriptive names. `build:clean` deletes files from the destination to ensure it is clean. `build:copy` is designed to copy any files not included with the babel build.

*__NB:__ If we used webpack we would not need to worry about the spliting of `build` as it would be handled in webpack and its web.config.js*.

`build` then runs three stages in series and if any fails it aborts.

Having multiple scripts like this is common as it enables descriptive names and single responsibility.

## Database Security

Another improvement we can make (thats not just needed for containerisation), is adding authentication to our database. We need modify our `setupdb.js` file to handle it.

In `bookstore-api/src/utils/setupdb.js` change the following:

```js
mongoose.connect(connectionString);
// to
mongoose.connect(connectionString, { useNewUrlParser: true });
```

And
```js
const createConnString = (mongoDbUri, databaseName) => `${mongoDbUri}/${databaseName}`;
// to
const createConnString = (mongoDbUri, databaseName) => `${mongoDbUri}/${databaseName}?authSource=admin`;
```

This protects our database and prepares us for adding the username and password to the DB URI.

## Cleanup Files

We have the option to clean up files that we dont need anymore, although leaving **shouldn't** cause any issues.

We have added a `bookstore-api/.env` file, *which should __not__ be in source control*, but we no longer need it.

Also the `bookstore-api/node_modules` folder is no longer needed, so safe to remove those.

## Adding Docker

We will start by adding a `bookstore-api/.dockerignore` file, which tells docker which files in our app to exclude, just like `.gitignore` does.

```INI
node_modules
npm-debug.log
.env
```

Now we need to tell Docker how to build your application.

Add a `bookstore-api/Dockerfile` to your application:
```docker
# The prepared base image that includes everything it needs to run node in a container
FROM node:12-slim

# Set the current directory
WORKDIR /app

# Globally install nodemon
RUN npm install -g nodemon

# Only copy the package.json files as we will use these to create
COPY package*.json ./

# `ci` rather than `i` (aka install) as it is more strict than i and better for pipelines
RUN npm ci

# Only now add the rest of the code, this will mean the install stage about will be cached until package.json is changed
COPY . .

# We tell docker what ports we will be exposing
EXPOSE ${SERVER_PORT}

# The command to run (by default) when this docker image is run. Can be overwritten via docker-compose
CMD ["npm", "start"]
```

I have commented on each line, so  please read. But in essense it is grabbing an image with node already in it. Then, using the `package-lock.json` file, node installs the dependencies.

The Dockerfile exposes the port and then, as a default, attempts to run the application.

You can have multiple Dockerfiles e.g. Dockerfile.prod than build in different ways, in theory thats the best method, but not always the simpliest to

*__Warning:__ Ideally you would want to run the container using a different user, however that can cause some issues using nodemon. More information [here](https://github.com/nodejs/docker-node/blob/master/docs/BestPractices.md#non-root-user) and [here](https://github.com/nodejs/docker-node/issues/971).*

It is perfect possible to now build this into an image and run it as a container, with a series of long `docker run` commands... but there is a better way.

## Docker Compose

As we talked about in the Docker presentation, instead of creating several docker run commands, one for each service, and also attaching networks and creating data volumes by hand each time, we can use a `docker-compose.yml` file instead.

Add the `docker-compose.yml` file to the root:
```yml
version: "3.4"

services:
    bookstore-api:
        build: ./bookstore-api # We tell docker about the location of the dockerfile
        volumes:
            - ./bookstore-api:/app # We link our bookstore app on the  host (your machine) and the docker container
            - node_modules:/app/node_modules # We create a node_modules volume container, so that our link above does not remove them node_modules by mistake
        ports:
            - "31023:80" # Our exposed port is 30123, but it was 80 in the container
        environment: # Env variables to pass to the container DONT NORMALLY PUT passwords etc in here
            DB_MONGO_URL: mongodb://${DB_USER}:${DB_PASSWORD}@bookstore-api-mongo
            DB_NAME: bookstoreDb
            SERVER_PORT: 80
        depends_on: # Dont start this until these have already started
            - bookstore-api-mongo
    bookstore-api-mongo:
        image: mongo
        restart: always # Restart this container automatically, unless deliberately and manually stopped
        environment:
            MONGO_INITDB_ROOT_USERNAME: ${DB_USER} # We can use environment variables in here as well to hide sensitive data
            MONGO_INITDB_ROOT_PASSWORD: ${DB_PASSWORD}
        volumes:
            - bookstore-api-mongo-data:/data/db

        ports:
            - "27021:27017"
    bookstore-api-mongo-express:
        image: mongo-express
        restart: always
        ports:
            - "8192:8081"
        environment:
            ME_CONFIG_MONGODB_SERVER: bookstore-api-mongo
            ME_CONFIG_MONGODB_PORT: 27017
            ME_CONFIG_MONGODB_ADMINUSERNAME: ${DB_USER}
            ME_CONFIG_MONGODB_ADMINPASSWORD: ${DB_PASSWORD}
        depends_on:
            - bookstore-api-mongo
volumes: # declared volumes, basically storage for files. Can be set to not disappear after a closed down swarm
  bookstore-api-mongo-data:
  node_modules:
```

I have annotated most the lines to explain what they are doing, but in essense the file is creating each of the services it needs to run. It is also creating an internal network, required for inter container connnections, and data volumes to store data after containers are stopped.

### Overrides

Currently this script would simply use the command at the end of the dockerfile we created, however as we want to use it to develop, we need to pass the command to override the command in the Dockerfile. To does this we can add a `command: npm run start:dev` to the `bookstore-api:` service above.

But that would mean we are making our compose file only useful for dev right?

So instead of adding it here we also create a `docker-compose.override.yml` file:

```Dockerfile
version: "3.4"

services:
  bookstore-api:
    # Overriding the command in this file means that we can change the initial command
    # Dependant on the need
    command: "npm run start:dev"
```

By default `docker-compose up` will use `docker-compose.yml` by default.
But if there is also a `docker-compose.override.yml` file it will use that AS WELL and
override the values with the ones there. *NB: If you explicitly tell docker-compose which files to use, it will ignore both those default files.*

So now you can use the `docker-compose.yml` file as a base and create other compose files e.g. `docker-compose.test.yml` to do other things, *NB: you'll need to explicitly pass them into `docker-compose run`*.

### .env

Docker compose also supports .env files, so we can create a new one to put in the root folder and tell docker the DB_USER and DB_PASSWORD environment variables required in the docker-compose file:

```
DB_PASSWORD=MyPassword
DB_USER=root
```

## Run

To run the application open the terminal at the root folder and run:

```cmd
docker-compose up
```
You will then get several containers spin up *(first time will be slow as it is downloading everything)*, all with arbitary names.

If you want to access the inside of a container, you can use:

```cmd
docker-compose exec bookstore-api-mongo bash
```

The advantage to that is you dont have to know the containers real name!

## Todo

- Add docker and docker-compose to your application
- While the code is running change the app, you should see it reflected in nodemon.

#### Stretch goals

- Create a compose that will enable testing in a container
- Adapt compose so that you can attach a debugger to the container (there are a couple of ways of doing it, but worth checking [this](https://blog.risingstack.com/how-to-debug-a-node-js-app-in-a-docker-container/) first)
