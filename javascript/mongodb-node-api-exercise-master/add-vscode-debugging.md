# Debugging a Node Application in VS Code

To enable debugging in VS Code you can add the following 2 files to the root of the application:

- Create `.vscode/tasks.json` file
```json
{
	"version": "2.0.0",
	"tasks": [
		{
			"label": "build",
			"type": "shell",
			"command": "npm run build",
		}
	]
}
```
- Create `.vscode/launch.json` file
```json
{
	"version": "0.2.0",
	"configurations": [
		{
			"type": "node",
			"request": "launch",
			"name": "Debug",
			"program": "${workspaceRoot}/src/index.js",
			"cwd": "${workspaceRoot}",
			"preLaunchTask": "build",
			"outFiles": ["${workspaceRoot}/dist/**.js"]
		},
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
				"NODE_ENV": "debug"
			}
		}
	]
}
```

You can then run the code by selecting which configuration you want to use on the debug sidebar and pressing play.