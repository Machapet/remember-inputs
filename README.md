# Remember Inputs

This extension is a based on very useful [Memento Inputs](https://github.com/joelspadin/vscode-memento-inputs) extension by Joel Spadin. But the project seems to be dead, its author does not response to my pull requests for years so that I decided to create this project. It extends **Memento Inputs** commands by `recallString`. See below. All commands are prefixed by `remember` instead of the original `memento` but the usage is the same.

The extension lets you create [task input variables](https://code.visualstudio.com/docs/editor/variables-reference#_input-variables)
which act like `pickString` and `promptString` but remember the value you
previously entered and default to it the next time they are used. In addition the remembered string can be used as input without prompt or picking using `recallString`.

## Usage

In your `tasks.json`, create an input variable with type `command` and set the
`command` and `args` as follows:

`promptString`:

* **command**: `remember.promptString`
* **args**:
  * **id**: Name to use for storing the last-used value.
  * **description**: Shown in the quick input to provide context for the input.
  * **default**: Default value that will be used if no last-used value has been set.
  * **placeholder**: Optional text to display if the input box is empty.

`pickString`:

* **command**: `remember.pickString`
* **args**:
  * **id**: Name to use for storing the last-used value.
  * **description**: Shown in the quick input to provide context for the input.
  * **options**: An array of options for the user to pick from.
  * **default**: Default value that will be used if no last-used value has been set.

`recallString`:

* **command**: `remember.recallString`
* **args**:
  * **id**: Id of input to remember.
  * **default**: Default value that will be used if no last-used value has been set.


Below is an example of a `tasks.json` that defines tasks to build an application
and deploy it to some other device.

When starting a build, it will prompt for the build mode. Where the regular
`pickString` would always default to `production`, this will default to
whichever mode you last picked.

The `Deploy` task depends on `MySSHTask`. Both requires the IP address of the target
device. To supress the input prompt to be shown twice the `MySSHTask` reads `input:address` that uses `remember.promptString` while
`Deploy` reads `input:storedAddress` that uses `remember.recallString` which takes the string from `input:adress` entered by user in `MySSHTask`.
The benefit is that user will be prompted only once to enter the IP address.


```JSON
{
    "version": "2.0.0",
    "tasks": [
        {
            "label": "Build",
            "group": { "kind": "build", "isDefault": true },
            "type": "shell",
            "command": "npm",
            "args": ["run", "build", "--mode", "${input:buildMode}"]
        },
        {
            "label": "Deploy",
            "type": "shell",
            "command": "npm",
            "args": ["run", "deploy", "--address", "${input:storedAddress}"],
			"depependsOn" : ["MySSHTask"]
        },
		{
			"label": "MySSHTask",
			"type": "shell",
			"command": "ssh",
			"args": [
				"root@${input:address}",
				"<ie. some shell commands before Deploy is executed...>"
			]
		}
    ],
    "inputs": [
        {
            "id": "buildMode",
            "type": "command",
            "command": "remember.pickString",
            "args": {
                "id": "buildMode",
                "description": "Build mode",
                "options": ["production", "dev"],
                "default": "production"
            }
        },
        {
            "id": "address",
            "type": "command",
            "command": "remember.promptString",
            "args": {
                "id": "address",
                "description": "Target IP address",
                "default": "192.168.1.42",
                "placeholder": "IP address"
            }
        },
		{
            "id": "storedAddress",
            "type": "command",
            "command": "remember.recallString",
            "args": {
                "id": "address",
                "default": "192.168.1.42"
            }
        }
    ]
}
```
