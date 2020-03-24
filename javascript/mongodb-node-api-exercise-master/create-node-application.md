# Create Node Application

We are going to make a (very basic) Bookstore API! We will start by creating a new application.

A JavaScript script can be run raw (not compiled) with Node. However NodeJS can miss some features, such as import/export of modules, so it can be better to use compilation such as Babel to enable those features.

We will be using **Babel** with this project.

## First Step

*We need to create an empty node application in a new folder*

- Create empty folder
- Navigate to the folder
- Open Terminal
- Create node application `npm init`
  - Complete each question, pressing enter to stick to the default

*Now we need to add some packages to use babel.*

- Add packages:
  ```
  npm i @babel/core @babel/cli @babel/node @babel/preset-env
  ```

*The Babel packages will enable us to compile and build our application.*

*We need to add some scripts to make it possible to develop, build and test   our application more easily.*

- Add following to package.json scripts:
  ```json
  "scripts": {
	"prestart": "npm run build",
	"start": "node dist/index.js",
	"start:dev": "babel-node src/index.js",
	"build": "babel src -d dist -s"
  },
  ```

***NB:** If you want to use `nodemon` during development to automatically reload the application when editing files, install nodemon globally and change the `"start:dev": "nodemon --exec babel-node src/index.js",` instead of the `start:dev` above.*

*To explain what is happening in these scripts, `build` takes in all the code you are creating in the `src/` folder and compiles it, placing the output into the `dist/` folder. This can then be used by `start`, which runs the compiled application directly.*

*To ensure the code is built prior to running the start script, we use `prestart` to build the code before running.*

*Whereas `start` is designed for efficiency and production, `start:dev` is used for development. Instead of Node it uses the Babel Node package that compiles code on the fly, which is faster to handle code changes.*


*We need to instruct babel on what we are targeting, in this case Node, rather than a browser. This means we can reduce the amount of polyfill code that is generated, making the code more efficient.*


- Create a `.babelrc` file
  ```json
  {
  	"presets": [
  		[
  			"@babel/preset-env",
  			{
  				"targets": {
  					"node": "current"
  				}
  			}
  		]
  	],
  	"env": {
  		"debug": {
  			"sourceMaps": "inline",
  			"retainLines": true
  		}
  	}
  }
  ```

**If you are using VS Code, please follow [adding debugging to VS Code](add-vscode-debugging.md)**.

*We need to create an `index.js` in a `src` folder, which will be the entry point for the application.*

- Create `src/index.js` file:
  ```js
  console.log('Helloworld');
  ```

- Run the application:
  ```bash
  npm start
  ```