# Debugging Tests using VS Code

This extends the guide for [Adding Debugging to VS Code](./add-vscode-debugging.md) so follow that first.

- Add the following configuration to
  ```json
  {
  	"name": "Debug Using Jest",
  	"type": "node",
  	"request": "launch",
  	"program": "${workspaceRoot}/node_modules/jest/bin/jest.js",
  	"args": ["-i"],
  	"cwd": "${workspaceRoot}",
  	"runtimeExecutable": null,
  	"sourceMaps": true,
  	"protocol": "inspector",
  	"console": "integratedTerminal",
  	"env": {
   	  "NODE_ENV": "debug",
   	  "DB_NAME": "testBookstoreDb",
   	  "LOG_LEVEL": "warn"
    }
  }
  ```

You can then run the code by selecting which configuration you want to use on the "Debug Using Jest" sidebar and pressing `play`.