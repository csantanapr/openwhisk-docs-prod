---

copyright:
  years: 2016, 2018
lastupdated: "2018-01-18"

---

{:shortdesc: .shortdesc}
{:codeblock: .codeblock}
{:screen: .screen}
{:tip: .tip}
{:pre: .pre}

# Create and invoke Actions
{: #openwhisk_actions}

Actions are stateless code snippets that run on the {{site.data.keyword.openwhisk}} platform. For example, an Action can be used to detect the faces in an image, respond to a database change, aggregate a set of API calls, or post a Tweet. An Action can be written as a JavaScript, Swift, Python, PHP function, Java method, or any binary-compatible executable, including Go programs and custom executables packaged as Docker containers.
{:shortdesc}

Actions can be explicitly invoked, or run in response to an event. In either case, each run of an Action results in an activation record that is identified by a unique activation ID. The input to an Action and the result of an Action are a dictionary of key-value pairs, where the key is a string and the value a valid JSON value. Actions can also be composed of calls to other Actions or a defined sequence of Actions.

Learn how to create, invoke, and debug Actions in your preferred development environment:
* [JavaScript](#creating-and-invoking-javascript-actions)
* [Swift](#creating-swift-actions)
* [Python](#creating-python-actions)
* [Java](#creating-java-actions)
* [PHP](#creating-php-actions)
* [Docker](#creating-docker-actions)
* [Go](#creating-go-actions)
* [Native binaries](#creating-native-actions)

In addition, learn about:
* [Watching Action output](#watching-action-output)
* [Listing Actions](#listing-actions)
* [Deleting Actions](#deleting-actions)
* [Accessing Action metadata within the Action body](#accessing-action-metadata-within-the-action-body)


## Create and invoke JavaScript Actions
{: #creating-and-invoking-javascript-actions}

The following sections guide you through working with Actions in JavaScript. You begin with the creation and invocation of a simple action. Then, you move on to adding parameters to an action and invoking that action with parameters. Next, is setting default parameters and invoking them. Then, you create asynchronous Actions, and finally work with action sequences.


### Create and invoke a simple JavaScript action
{: #openwhisk_single_action_js}

Review the following steps and examples to create your first JavaScript action.

1. Create a JavaScript file with the following content. For this example, the file name is 'hello.js'.

  ```javascript
  function main() {
      return {payload: 'Hello world'};
  }
  ```
  {: codeblock}

  The JavaScript file might contain additional functions. However, by convention, a function called `main` must exist to provide the entry point for the action.

2. Create an action from the following JavaScript function. For this example, the action is called 'hello'.

  ```
  wsk action create hello hello.js
  ```
  {: pre}

  ```
  ok: created action hello
  ```
  The CLI automatically infers the type of the action by using the source file extension. For `.js` source files, the action runs by using a Node.js 6 runtime. You can also create an action that runs with Node.js 8 by explicitly specifying the parameter `--kind nodejs:8`. For more information, see the Node.js 6 vs 8 [reference](./openwhisk_reference.html#openwhisk_ref_javascript_environments).
  
3. List the Actions that you created:

  ```
  wsk action list
  ```
  {: pre}

  ```
  actions
  hello       private
  ```

  You can see the `hello` action you created.

4. After you create your action, you can run it in the cloud in OpenWhisk with the 'invoke' command. You can invoke Actions with a *blocking* invocation (that is, request/response style) or a *non-blocking* invocation by specifying a flag in the command. A blocking invocation request _waits_ for the activation result to be available. The wait period is the lesser of 60 seconds or the action's [time limit value](./openwhisk_reference.html#openwhisk_syslimits). The result of the activation is returned if it is available within the wait period. Otherwise, the activation continues processing in the system, and an activation ID is returned so that one can check for the result later, as with non-blocking requests (see [here](#watching-action-output) for tips on monitoring activations).

  This example uses the blocking parameter, `--blocking`:

  ```
  wsk action invoke --blocking hello
  ```
  {: pre}

  ```
  ok: invoked hello with id 44794bd6aab74415b4e42a308d880e5b
  ```

  ```json
  {
      "result": {
          "payload": "Hello world"
      },
      "status": "success",
      "success": true
  }
  ```

  The command outputs two important pieces of information:
  * The activation ID (`44794bd6aab74415b4e42a308d880e5b`)
  * The invocation result if it is available within the expected wait period

  The result in this case is the string `Hello world` returned by the JavaScript function. The activation ID can be used to retrieve the logs or result of the invocation at a future time.  

5. If you don't need the action result right away, you can omit the `--blocking` flag to make a non-blocking invocation. You can get the result later by using the activation ID. See the following example:

  ```
  wsk action invoke hello
  ```
  {: pre}

  ```
  ok: invoked hello with id 6bf1f670ee614a7eb5af3c9fde813043
  ```

  ```
  wsk activation result 6bf1f670ee614a7eb5af3c9fde813043
  ```
  {: pre}

  ```json
  {
      "payload": "Hello world"
  }
  ```

6. If you forget to record the activation ID, you can get a list of activations ordered from the most recent to the oldest. Run the following command to get a list of your activations:

  ```
  wsk activation list
  ```
  {: pre}

  ```
  activations
  44794bd6aab74415b4e42a308d880e5b         hello
  6bf1f670ee614a7eb5af3c9fde813043         hello
  ```

### Pass parameters to an action
{: #openwhisk_pass_params}

Parameters can be passed to the action when it is invoked.

1. Use parameters in the action. For example, update the 'hello.js' file with the following content:

  ```javascript
  function main(params) {
      return {payload:  'Hello, ' + params.name + ' from ' + params.place};
  }
  ```
  {: codeblock}

  The input parameters are passed as a JSON object parameter to the `main` function. Notice how the `name` and `place` parameters are retrieved from the `params` object in this example.

2. Update and invoke the `hello` action, while passing it the `name` and `place` parameter values. See the following example:

  ```
  wsk action update hello hello.js
  ```
  {: pre}

  If you need to modify your non-service credential parameters, be aware that doing an `action update` command with new parameters removes any parameters that currently exist, but are not specified in the `action update` command. For example, if there are two parameters aside from the `__bx_creds`, with keys named key1 and key2.  If you run an `action update` command with `-p key1 new-value -p key2 new-value` but omit the `__bx_creds` parameter, the `__bx_creds` parameter will no longer exist after the `action update` completes successfully. You then must re-bind the service credentials. This is a known limitation without a workaround.
  {: tip}  

3.  Parameters can be provided explicitly on the command line, or by supplying a file that contains the desired parameters.

  To pass parameters directly through the command line, supply a key/value pair to the `--param` flag:
  ```
  wsk action invoke --result hello --param name Bernie --param place Vermont
  ```
  {: pre}

  In order to use a file that contains parameter content, create a file that contains the parameters in JSON format. The filename must then be passed to the `param-file` flag:

  See the following example parameter file called `parameters.json`:
  ```json
  {
      "name": "Bernie",
      "place": "Vermont"
  }
  ```

  ```
  wsk action invoke --result hello --param-file parameters.json
  ```
  {: pre}

  ```json
  {
      "payload": "Hello, Bernie from Vermont"
  }
  ```

  Notice the use of the `--result` option: it implies a blocking invocation where the CLI waits for the activation to complete and then displays only the result. For convenience, this option can be used without `--blocking` which is automatically inferred.

  Additionally, if parameter values that are specified on the command line are valid JSON, then they are parsed and sent to your action as a structured object. For example, update the hello action to the following:

  ```javascript
  function main(params) {
      return {payload:  'Hello, ' + params.person.name + ' from ' + params.person.place};
  }
  ```
  {: codeblock}

  Now the action expects a single `person` parameter to have fields `name` and `place`. Next, invoke the action with a single `person` parameter that is a valid JSON, like in the following example:

  ```
  wsk action invoke --result hello -p person '{"name": "Bernie", "place": "Vermont"}'
  ```
  {: pre}

  The result is the same because the CLI automatically parses the `person` parameter value into the structured object that the action now expects:
  ```json
  {
      "payload": "Hello, Bernie from Vermont"
  }
  ```

### Setting default parameters
{: #openwhisk_binding_actions}

Actions can be invoked with multiple named parameters. Recall that the `hello` action from the previous example expects two parameters: the *name* of a person, and the *place* where they're from.

Rather than pass all the parameters to an action every time, you can bind certain parameters. The following example binds the *place* parameter so that the action defaults to the place "Vermont":

1. Update the action by using the `--param` option to bind parameter values, or by passing a file that contains the parameters to `--param-file`

  To specify default parameters explicitly on the command-line, provide a key/value pair to the `param` flag:

  ```
  wsk action update hello --param place Vermont
  ```
  {: pre}

  Passing parameters from a file requires the creation of a file that contains the desired content in JSON format. The filename must then be passed to the `-param-file` flag:

  See the following example parameter file called `parameters.json`:
  ```json
  {
      "place": "Vermont"
  }
  ```
  {: codeblock}

  ```
  wsk action update hello --param-file parameters.json
  ```
  {: pre}

2. Invoke the action, passing only the `name` parameter this time.

  ```
  wsk action invoke --result hello --param name Bernie
  ```
  {: pre}

  ```json
  {
      "payload": "Hello, Bernie from Vermont"
  }
  ```

  Notice that you did not need to specify the place parameter when you invoked the action. Bound parameters can still be overwritten by specifying the parameter value at invocation time.

3. Invoke the action, passing both `name` and `place` values. The latter overwrites the value that is bound to the action.

  Using the `--param` flag:

  ```
  wsk action invoke --result hello --param name Bernie --param place "Washington, DC"
  ```
  {: pre}

  Using the `--param-file` flag:

  File parameters.json:
  ```json
  {
    "name": "Bernie",
    "place": "Vermont"
  }
  ```
  {: codeblock}
  ```
  wsk action invoke --result hello --param-file parameters.json
  ```
  {: pre}

  ```json
  {  
      "payload": "Hello, Bernie from Washington, DC"
  }
  ```

### Get an action URL

An action can be invoked through the REST interface via an HTTPS request. To get an action URL, execute the following command:

```
wsk action get actionName --url
```
{: pre}

```
ok: got action actionName
https://${APIHOST}/api/v1/namespaces/${NAMESPACE}/actions/actionName
```

Authentication must be provided when invoking an action via an HTTPS request. For more information regarding
action invocations using the REST interface, see [Using REST APIs with OpenWhisk](./openwhisk_rest_api.html#actions).
{: tip}

### Save Action code

Code associated with an existing action is fetched and saved locally. Saving is performed on all Actions except sequences and docker Actions. When saving action code to a file, the code is saved in the current working directory, and the saved file path is displayed.

1. Save action code to a filename that corresponds with an existing action name. A file extension that corresponds to the action kind is used, or an extension of type `.zip` will be used for action code that is a zip file.
  ```
  wsk action get actionName --save
  ```
  {: pre}

  ```
  ok: saved action code to /absolutePath/currentDirectory/actionName.js
  ```

2. Instead of allowing the CLI to determine the filename and extension of the saved code, a custom filename and extension can be provided by using the `--save-as` flag.
  ```
  wsk action get actionName --save-as codeFile.js
  ```
  {: pre}

  ```
  ok: saved action code to /absolutePath/currentDirectory/codeFile.js
  ```

### Create asynchronous Actions
{: #openwhisk_asynchrony_js}

JavaScript functions that run asynchronously can return the activation result after the `main` function returns by returning a Promise in your action.

1. Save the following content in a file called `asyncAction.js`.

  ```javascript
  function main(args) {
       return new Promise(function(resolve, reject) {
         setTimeout(function() {
           resolve({ done: true });
         }, 2000);
      })
   }
  ```
  {: codeblock}

  Notice that the `main` function returns a Promise, which indicates that the activation isn't completed yet, but is expected to in the future.

  The `setTimeout()` JavaScript function in this case waits for 2 seconds prior to calling the callback function, which represents the asynchronous code, and goes inside the Promise's callback function.

  The Promise's callback takes two arguments, resolve and reject, which are both functions.  The call to `resolve()` fulfills the Promise and indicates that the activation completes normally.

  A call to `reject()` can be used to reject the Promise and signal that the activation completes abnormally.

2. Run the following commands to create the action and invoke it:

  ```
  wsk action create asyncAction asyncAction.js
  ```
  {: pre}

  ```
  wsk action invoke --result asyncAction
  ```
  {: pre}

  ```json
  {
      "done": true
  }
  ```

  Notice that you performed a blocking invocation of an asynchronous action.

3. Fetch the activation log to see how long the activation took to complete:

  ```
  wsk activation list --limit 1 asyncAction
  ```
  {: pre}
  
  ```
  activations
  b066ca51e68c4d3382df2d8033265db0             asyncAction
  ```

  ```
  wsk activation get b066ca51e68c4d3382df2d8033265db0
  ```
  {: pre}
 
  ```json
  {
      "start": 1455881628103,
      "end":   1455881648126,
      ...
  }
  ```

  Comparing the `start` and `end` time stamps in the activation record, you can see that this activation took slightly over 2 seconds to complete.

### Use Actions to call an external API
{: #openwhisk_apicall_action}

The examples so far are self-contained JavaScript functions. You can also create an action that calls an external API.

This example invokes a Yahoo Weather service to get the current conditions at a specific location.

1. Save the following content in a file called `weather.js`.

  ```javascript
  var request = require('request');

  function main(params) {
      var location = params.location || 'Vermont';
      var url = 'https://query.yahooapis.com/v1/public/yql?q=select item.condition from weather.forecast where woeid in (select woeid from geo.places(1) where text="' + location + '")&format=json';

      return new Promise(function(resolve, reject) {
          request.get(url, function(error, response, body) {
              if (error) {
                  reject(error);
              }
              else {
                  var condition = JSON.parse(body).query.results.channel.item.condition;
                  var text = condition.text;
                  var temperature = condition.temp;
                  var output = 'It is ' + temperature + ' degrees in ' + location + ' and ' + text;
                  resolve({msg: output});
              }
          });
      });
  }
  ```
  {: codeblock}

 The action in the example uses the JavaScript `request` library to make an HTTP request to the Yahoo Weather API, and extracts fields from the JSON result. The [References](./openwhisk_reference.html#openwhisk_ref_javascript_environments) detail the Node.js packages that you can use in your Actions.

  This example also shows the need for asynchronous Actions. The action returns a Promise to indicate that the result of this action is not available yet when the function returns. Instead, the result is available in the `request` callback after the HTTP call completes, and is passed as an argument to the `resolve()` function.

2. Run the following commands to create the action and invoke it:

  ```
  wsk action create weather weather.js
  ```
  {: pre}

  ```
  wsk action invoke --result weather --param location "Brooklyn, NY"
  ```
  {: pre}

  ```json
  {
      "msg": "It is 28 degrees in Brooklyn, NY and Cloudy"
  }
  ```

### Package an action as a Node.js module
{: #openwhisk_js_packaged_action}

As an alternative to writing all your action code in a single JavaScript source file, you can write an action as a `npm` package. Consider as an example a directory with the following files:

First, `package.json`:

```json
{
  "name": "my-action",
  "main": "index.js",
  "dependencies" : {
    "left-pad" : "1.1.3"
  }
}
```
{: codeblock}

Then, `index.js`:

```javascript
function myAction(args) {
    const leftPad = require("left-pad")
    const lines = args.lines || [];
    return { padded: lines.map(l => leftPad(l, 30, ".")) }
}

exports.main = myAction;
```
{: codeblock}

The action is exposed through `exports.main`. The action handler itself can have any name, as long as it conforms to the usual signature of accepting an object and returning an object (or a `Promise` of an object). Per Node.js convention, you must either name this file `index.js` or specify the file name that you prefer as the `main` property in package.json.

To create an OpenWhisk action from this package:

1. Install first all dependencies locally

  ```
  npm install
  ```
  {: pre}

2. Create a `.zip` archive containing all files (including all dependencies):

  ```
  zip -r action.zip *
  ```
  {: pre}

  Using the Windows Explorer action for creating the zip file results in an incorrect structure. OpenWhisk zip Actions must have `package.json` at the root of the zip, while Windows Explorer places it inside a nested folder. The safest option is to use the command line `zip` command.
  {: tip}

3. Create the action:

  ```
  wsk action create packageAction --kind nodejs:6 action.zip
  ```
  {: pre}

  When creating an action from a `.zip` archive with the CLI tool, you must explicitly provide a value for the `--kind` flag by using `nodejs:6` or `nodejs:8`.

4. You can invoke the action like any other:

  ```
  wsk action invoke --result packageAction --param lines "[\"and now\", \"for something completely\", \"different\" ]"
  ```
  {: pre}
  
  ```json
  {
      "padded": [
          ".......................and now",
          "......for something completely",
          ".....................different"
      ]
  }
  ```

Finally, note that while most `npm` packages install JavaScript sources on `npm install`, some also install and compile binary artifacts. The archive file upload currently does not support binary dependencies but rather only JavaScript dependencies. Action invocations may fail if the archive includes binary dependencies.

### Package an action as a single bundle
{: #openwhisk_js_webpack_action}

It is convenient to only include the minimal code into a single `.js` file that includes dependencies. This approach allows for faster deployments, and in some circumstances where packaging the action as a zip might be too large because it includes unnecessary files.

You can use a JavaScript module bundler such as [webpack](https://webpack.js.org/concepts/). When webpack processes your code, it recursively builds a dependency graph that includes every module that your action needs.

Here is a quick example using webpack:

Taking the previous example `package.json` add `webpack` as a development depency and add some npm script commands.
```json
{
  "name": "my-action",
  "main": "dist/bundle.js",
  "scripts": {
    "build": "webpack --config webpack.config.js",
    "deploy": "bx wsk action update my-action dist/bundle.js --kind nodejs:8"
  },
  "dependencies": {
    "left-pad": "1.1.3"
  },
  "devDependencies": {
    "webpack": "^3.8.1"
  }
}
```
{: codeblock}

Create the webpack configuration file `webpack.config.js`.
```javascript
var path = require('path');
module.exports = {
  entry: './index.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'bundle.js'
  },
  target: 'node'
};
```
{: codeblock}

Set the variable `global.main` to the main function of the action.
From the previous example:
```javascript
function myAction(args) {
    const leftPad = require("left-pad")
    const lines = args.lines || [];
    return { padded: lines.map(l => leftPad(l, 30, ".")) }
}
global.main = myAction;
```
{: codeblock}

If your function name is `main`, use this syntax instead:
```javascript
global.main = main;
```
{: codeblock}


To build and deploy an OpenWhisk Action using `npm` and `webpack`:

1. First, install dependencies locally:

  ```
  npm install
  ```
  {: pre}

2. Build the webpack bundle:

  ```
  npm run build
  ```
  {: pre}

  The file `dist/bundle.js` is created, and is used to deploy as the Action source code.

3. Create the Action using the `npm` script or the CLI.
  Using `npm` script:
  ```
  npm run deploy
  ```
  {: pre}
  Using the CLI:
  ```
  bx wsk action update my-action dist/bundle.js
  ```
  {: pre}


Finally, the bundle file that is built by `webpack` doesn't support binary dependencies but rather JavaScript dependencies. So Action invocations will fail if the bundle depends on binary dependencies, because this is not included with the file `bundle.js`.

## Create action sequences
{: #openwhisk_create_action_sequence}

You can create an action that chains together a sequence of Actions.

Several utility Actions are provided in a package that is called `/whisk.system/utils` that you can use to create your first sequence. You can learn more about packages in the [Packages](./openwhisk_packages.html) section.

1. Display the Actions in the `/whisk.system/utils` package.

  ```
  wsk package get --summary /whisk.system/utils
  ```
  {: pre}

  ```
  package /whisk.system/utils: Building blocks that format and assemble data
   action /whisk.system/utils/head: Extract prefix of an array
   action /whisk.system/utils/split: Split a string into an array
   action /whisk.system/utils/sort: Sorts an array
   action /whisk.system/utils/echo: Returns the input
   action /whisk.system/utils/date: Current date and time
   action /whisk.system/utils/cat: Concatenates input into a string
  ```

  You are using the `split` and `sort` Actions in this example.

2. Create an action sequence so that the result of one action is passed as an argument to the next action.

  ```
  wsk action create sequenceAction --sequence /whisk.system/utils/split,/whisk.system/utils/sort
  ```
  {: pre}

  This action sequence converts some lines of text to an array, and sorts the lines.

3. Invoke the action:

  ```
  wsk action invoke --result sequenceAction --param payload "Over-ripe sushi,\nThe Master\nIs full of regret."
  ```
  {: pre}

  ```json
  {
      "length": 3,
      "lines": [
          "Is full of regret.",
          "Over-ripe sushi,",
          "The Master"
      ]
  }
  ```

  In the result, you see that the lines are sorted.

**Note**: Parameters that are passed between Actions in the sequence are explicit, except for default parameters.
Therefore, parameters that are passed to the action sequence are only available to the first action in the sequence.
The result of the first action in the sequence becomes the input JSON object to the second action in the sequence (and so on).
This object does not include any of the parameters that are originally passed to the sequence unless the first action explicitly includes them in its result.
Input parameters to an action are merged with the action's default parameters, with the former taking precedence and overriding any matching default parameters.
For more information about invoking action sequences with multiple named parameters, see [Setting default parameters](./openwhisk_actions.html#openwhisk_binding_actions).

## Create Python Actions
{: #creating-python-actions}

The process of creating Python Actions is similar to that of JavaScript Actions. The following sections guide you through creating and invoking a single Python action, and adding parameters to that action.

### Create and invoke a Python action
{: #openwhisk_actions_python_invoke}

An action is simply a top-level Python function. For example, create a file called `hello.py` with the following source code:

```python
def main(args):
    name = args.get("name", "stranger")
    greeting = "Hello " + name + "!"
    print(greeting)
    return {"greeting": greeting}
```
{: codeblock}

Python Actions always consume a dictionary and produce a dictionary. The entry method for the action is `main` by default but can be specified explicitly to create the action with the `wsk` CLI by using `--main`, as with any other action type.

You can create an OpenWhisk action called `helloPython` from this function as follows:

```
wsk action create helloPython hello.py
```
{: pre}

The CLI automatically infers the type of the action from the source file extension. For `.py` source files, the action runs by using a Python 2.7 runtime. You can also create an action that runs with Python 3.6 by explicitly specifying the parameter `--kind python:3`. For more information, see the Python 2.7 vs 3.6 [reference](./openwhisk_reference.html#openwhisk_ref_python_environments).

Action invocation is the same for Python Actions as it is for JavaScript Actions:

```
wsk action invoke --result helloPython --param name World
```
{: pre}

```json
  {
      "greeting": "Hello World!"
  }
```

### Package Python Actions in zip files
{: #openwhisk_actions_python_zip}

You can package a Python action and dependent modules in a zip file.
The filename of the source file that contains the entry point (e.g., `main`) must be `__main__.py`.
For example, to create an action with a helper module called `helper.py`, first create an archive containing your source files:

```bash
zip -r helloPython.zip __main__.py helper.py
```
{: pre}

Then create the action:

```bash
wsk action create helloPython --kind python:3 helloPython.zip
```
{: pre}

### Package Python Actions with a virtual environment in zip files
{: #openwhisk_actions_python_virtualenv}

Another way of packaging Python dependencies is by using a virtual environment (`virtualenv`) which allows you to link additional packages that can be installed via [`pip`](https://packaging.python.org/installing/) for example.
To ensure compatibility with the OpenWhisk container, package installations inside a virtualenv must be done in the target environment.
So the docker image `openwhisk/python2action` or `openwhisk/python3action` are used to create a virtualenv directory for your action.

As with basic zip file support, the name of the source file that contains the main entry point must be `__main__.py`. To clarify, the contents of `__main__.py` is the main function, so for this example you can rename `hello.py` to `__main__.py` from the previous section. In addition, the virtualenv directory must be named `virtualenv`. See the following example scenario for installing dependencies, packaging them in a virtualenv, and creating a compatible OpenWhisk action.

1. Given a [requirements.txt ![External link icon](../icons/launch-glyph.svg "External link icon")](https://pip.pypa.io/en/latest/user_guide/#requirements-files) file that contains the `pip` modules and versions to install, run the following to install the dependencies and create a virtualenv using a compatible Docker image:
    ```
    docker run --rm -v "$PWD:/tmp" openwhisk/python3action bash -c "cd tmp && virtualenv virtualenv && source virtualenv/bin/activate && pip install -r requirements.txt"
    ```
    {: pre}

2. Archive the virtualenv directory and any additional Python files:
    ```
    zip -r helloPython.zip virtualenv __main__.py
    ```
    {: pre}

3. Create the action:
    ```
    wsk action create helloPython --kind python:3 helloPython.zip
    ```
    {: pre}

While these steps are shown for Python 3.6, you can do the same for Python 2.7 as well.


## Create PHP Actions
{: #creating-php-actions}

The process of creating PHP Actions is similar to that of JavaScript Actions. The following sections guide you through creating and invoking a single PHP action, and adding parameters to that action.

### Create and invoke a PHP action
{: #openwhisk_actions_php_invoke}

An action is simply a top-level PHP function. For example, create a file called `hello.php` with the following source code:

```php
<?php
function main(array $args) : array
{
    $name = $args["name"] ?? "stranger";
    $greeting = "Hello $name!";
    echo $greeting;
    return ["greeting" => $greeting];
}
```

PHP Actions always consume an associative array and return an associative array. The entry method for the action is `main` by default but can be specified explicitly when you create the action with the `wsk` CLI by using `--main`, as with any other action type.

You can create an OpenWhisk action called `helloPHP` from this function as follows:

```
wsk action create helloPHP hello.php
```
{: pre}

The CLI automatically infers the type of the action from the source file extension. For `.php` source files, the action runs by using a PHP 7.1 runtime. For more information, see the PHP [reference](./openwhisk_reference.html#openwhisk_ref_php).

Action invocation is the same for PHP Actions as it is for JavaScript Actions:

```
wsk action invoke --result helloPHP --param name World
```
{: pre}

```json
  {
      "greeting": "Hello World!"
  }
```

### Package PHP Actions in zip files
{: #openwhisk_actions_php_zip}

You can package a PHP action along with other files and dependent packages in a zip file.
The filename of the source file that contains the entry point (for example, `main`) must be `index.php`.
For example, to create an action that includes a second file that is called `helper.php`, first create an archive that contains your source files:

```bash
zip -r helloPHP.zip index.php helper.php
```
{: pre}

Then create the action:

```bash
wsk action create helloPHP --kind php:7.1 helloPHP.zip
```
{: pre}

## Create Swift Actions
{: #creating-swift-actions}

The process of creating Swift Actions is similar to that of JavaScript Actions. The following sections guide you through creating and invoking a single swift action, and adding parameters to that action.

You can also use the online [Swift Sandbox](https://swiftlang.ng.bluemix.net) to test your Swift code without having to install Xcode on your machine.

### Create and invoke an Action

An action is simply a top-level Swift function. For example, create a file called
`hello.swift` with the following content:

```swift
func main(args: [String:Any]) -> [String:Any] {
    if let name = args["name"] as? String {
        return [ "greeting" : "Hello \(name)!" ]
    } else {
        return [ "greeting" : "Hello stranger!" ]
    }
}
```
{: codeblock}

Swift Actions always consume a dictionary and produce a dictionary.

You can create a {{site.data.keyword.openwhisk_short}} action called `helloSwift` from this function as
follows:

```
wsk action create helloSwift hello.swift --kind swift:3.1.1
```
{: pre}
 

Always specify `swift:3.1.1` as previous Swift versions are not supported.
{: tip}

Action invocation is the same for Swift Actions as it is for JavaScript Actions:

```
wsk action invoke --result helloSwift --param name World
```
{: pre}

```json
  {
      "greeting": "Hello World!"
  }
```

**Attention:** Swift Actions that are run in a Linux environment is still in development, and {{site.data.keyword.openwhisk_short}} usually uses the latest available release, which is not necessarily stable. In addition, the version of Swift that is used with {{site.data.keyword.openwhisk_short}} might be inconsistent with versions of Swift from stable releases of XCode on MacOS.

### Package an Action as a Swift executable
{: #openwhisk_actions_swift_zip}

When you create an OpenWhisk Swift action with a Swift source file, it has to be compiled into a binary before the action is run. Once done, subsequent calls to the action are much faster until the container that holds your action is purged. This delay is known as the cold-start delay.

To avoid the cold-start delay, you can compile your Swift file into a binary and then upload to OpenWhisk in a zip file. As you need the OpenWhisk scaffolding, the easiest way to create the binary is to build it within the same environment it runs in. See the following steps:

- Run an interactive Swift action container by using the following command:
  ```
  docker run --rm -it -v "$(pwd):/owexec" openwhisk/action-swift-v3.1.1 bash
  ```
  {: pre}
  
- Copy the source code and prepare to build it.
  ```
  cp /owexec/hello.swift /swift3Action/spm-build/main.swift 
  ```
  {: pre}

  ```
  cat /swift3Action/epilogue.swift >> /swift3Action/spm-build/main.swift
  ```
  {: pre}

  ```
  echo '_run_main(mainFunction:main)' >> /swift3Action/spm-build/main.swift
  ```
  {: pre}

- (Optional) Create the `Package.swift` file to add dependencies.
   ```
   swift import PackageDescription
   
   let package = Package(
     name: "Action",
         dependencies: [
             .Package(url: "https://github.com/apple/example-package-deckofplayingcards.git", majorVersion: 3),
             .Package(url: "https://github.com/IBM-Swift/CCurl.git", "0.2.3"),
             .Package(url: "https://github.com/IBM-Swift/Kitura-net.git", "1.7.10"),
             .Package(url: "https://github.com/IBM-Swift/SwiftyJSON.git", "15.0.1"),
             .Package(url: "https://github.com/watson-developer-cloud/swift-sdk.git", "0.16.0")
         ]
   )
   ```
   {: pre}

  This example adds `swift-watson-sdk` and `example-package-deckofplayingcards` dependencies.
  Notice that `CCurl`, `Kitura-net`, and `SwiftyJSON` are provided in the standard Swift action so you can include them in your own `Package.swift`.

- Copy Package.swift to spm-build directory
  ```
  cp /owexec/Package.swift /swift3Action/spm-build/Package.swift
  ```
  {: pre}

- Change to the spm-build directory
  ```
  cd /swift3Action/spm-build
  ```
  {: pre}

- Compile your Swift Action.
  ```
  swift build -c release
  ```
  {: pre}

- Create the zip archive.
  ```
  zip /owexec/hello.zip .build/release/Action
  ```
  {: pre}

- Exit the Docker container.
  ```
  exit
  ```
  {: pre}

You can see that hello.zip is created in the same directory as hello.swift. 

- Upload it to OpenWhisk with the action name helloSwifty:
  ```
  wsk action update helloSwiftly hello.zip --kind swift:3.1.1
  ```
  {: pre}

- To check how much faster it is, run 
  ```
  wsk action invoke helloSwiftly --blocking
  ```
  {: pre}

The time that it took for the action to run is in the "duration" property and compare to the time it takes to run with a compilation step in the hello action.

## Create Java Actions
{: #creating-java-actions}

The process of creating Java Actions is similar to that of JavaScript and Swift Actions. The following sections guide you through creating and invoking a single Java action, and adding parameters to that action.

In order to compile, test, and archive Java files, you must have a [JDK 8](http://www.oracle.com/technetwork/java/javase/downloads/index.html) installed locally.

### Create and invoke an action
{: #openwhisk_actions_java_invoke}

A Java action is a Java program with a method called `main` that has the exact signature as follows:
```java
public static com.google.gson.JsonObject main(com.google.gson.JsonObject);
```
{: codeblock}

For example, create a Java file called `Hello.java` with the following content:

```java
import com.google.gson.JsonObject;
public class Hello {
    public static JsonObject main(JsonObject args) {
        String name = "stranger";
        if (args.has("name"))
            name = args.getAsJsonPrimitive("name").getAsString();
        JsonObject response = new JsonObject();
        response.addProperty("greeting", "Hello " + name + "!");
        return response;
    }
}
```
{: codeblock}

Then, compile `Hello.java` into a JAR file `hello.jar` as follows:
```
javac Hello.java
```
{: pre}

```
jar cvf hello.jar Hello.class
```
{: pre}

[google-gson](https://github.com/google/gson) must exist in your Java CLASSPATH to compile the Java file.
{: tip}

You can create a OpenWhisk action called `helloJava` from this JAR file as
follows:

```
wsk action create helloJava hello.jar --main Hello
```
{: pre}

When you use the command line and a `.jar` source file, you do not need to specify that you are creating a Java action; the tool determines that from the file extension.

You need to specify the name of the main class by using `--main`. An eligible main class is one that implements a static `main` method. If the class is not in the default package, use the Java fully qualified class name, for example, `--main com.example.MyMain`.

If needed, you can also customize the method name of your Java action. This is done by specifying the Java fully-qualified method name of your action, for example, `--main com.example.MyMain#methodName`.

Action invocation is the same for Java Actions as it is for Swift and JavaScript Actions:

```
wsk action invoke --result helloJava --param name World
```
{: pre}

```json
  {
      "greeting": "Hello World!"
  }
```

## Create Docker Actions
{: #creating-docker-actions}

With {{site.data.keyword.openwhisk_short}} Docker Actions, you can write your Actions in any language.

Your code is compiled into an executable binary and embedded into a Docker image. The binary program interacts with the system by taking input from `stdin` and replying through `stdout`.

As a prerequisite, you must have a Docker Hub account.  To set up a free Docker ID and account, go to [Docker Hub](https://hub.docker.com).

For the instructions that follow, assume that the Docker user ID is `janesmith` and the password is `janes_password`.  Assuming that the CLI is set up, three steps remain to set up a custom binary for use by {{site.data.keyword.openwhisk_short}}. After that, the uploaded Docker image can be used as an action.

1. Download the Docker skeleton. You can download and install it by using the CLI as follows:

  ```
  wsk sdk install docker
  ```
  {: pre}

  The Docker skeleton is now installed at the current directory.
  
  ```
  ls dockerSkeleton/
  ```
  {: pre}

  ```
  Dockerfile      README.md       buildAndPush.sh example.c
  ```

  The skeleton is a Docker container template where you can inject your code in the form of custom binaries.

2. Set up your custom binary in the blackbox skeleton. The skeleton already includes a C program that you can use.

  ```
  cat dockerSkeleton/example.c
  ```
  {: pre}

  ```c
  #include <stdio.h>
  int main(int argc, char *argv[]) {
      printf("This is an example log message from an arbitrary C program!\n");
      printf("{ \"msg\": \"Hello from arbitrary C program!\", \"args\": %s }",
             (argc == 1) ? "undefined" : argv[1]);
  }
  ```
  {: codeblock}

  You can modify this file as needed, or, add additional code and dependencies to the Docker image.
  If the latter, you can tweak the `Dockerfile` as necessary to build your executable.
  The binary must be located inside the container at `/action/exec`.

  The executable receives a single argument from the command line. It is a string serialization of the JSON object that represents the arguments to the action. The program may log to `stdout` or `stderr`.
  By convention, the last line of output _must_ be a stringified JSON object which represents the result of the action.

3. Build the Docker image and upload it using a supplied script. You must first run `docker login` to authenticate, and then run the script with a chosen image name.

  ```
  docker login -u janesmith -p janes_password
  ```
  {: pre}

  ```
  cd dockerSkeleton
  ```
  {: pre}

  ```
  ./buildAndPush.sh janesmith/blackboxdemo
  ```
  {: pre}

  Notice that part of the example.c file is compiled as part of the Docker image build process, so you do not need C compiled on your machine.
  In fact, unless you are compiling the binary on a compatible host machine, it can not run inside the container since the formats do not match.

  Your Docker container can now be used as an OpenWhisk action.


  ```
  wsk action create example --docker janesmith/blackboxdemo
  ```
  {: pre}

  Notice the use of `--docker` to create an action. All Docker images are assumed to be hosted on Docker Hub.
  The action can be invoked as any other {{site.data.keyword.openwhisk_short}} action. 

  ```
  wsk action invoke --result example --param payload Rey
  ```
  {: pre}

  ```json
  {
      "args": {
          "payload": "Rey"
      },
      "msg": "Hello from arbitrary C program!"
  }
  ```

  To update the Docker action, run `buildAndPush.sh` to upload the latest image to Docker Hub. This allows the system to pull your new Docker image the next time it runs the code for your action. If there are no warm containers, new invocations use the new Docker image. However, if there is a warm container that uses a previous version of your Docker image, any new invocations continue to use that image unless you run `wsk action update`. This indicates to the system, that for new invocations, to execute a docker pull to get your new Docker image.

  ```
  ./buildAndPush.sh janesmith/blackboxdemo
  ```
  {: pre}

  ```
  wsk action update example --docker janesmith/blackboxdemo
  ```
  {: pre}

  You can find more information about creating Docker Actions in the [References](./openwhisk_reference.html#openwhisk_ref_docker) section.

  The previous version of the CLI supported `--docker` without a parameter and the image name was a positional argument. In order to allow Docker Actions to accept initialization data via a (zip) file, normalize the user experience for Docker Actions so that a positional argument, if present, must be a file (for example, a zip file) instead. The image name must be specified following the `--docker` option. Thanks to user feedback, the `--native` argument is included as shorthand for `--docker openwhisk/dockerskeleton`, so that executables that run inside the standard Docker action SDK are more convenient to create and deploy.
  
  For example, this tutorial creates a binary executable inside the container located at `/action/exec`. If you copy this file to your local file system and zip it into `exec.zip`, then you can use the following commands to create a docker action that receives the executable as initialization data. 

  ```
  wsk action create example exec.zip --native
  ```
  {: pre}

  Which is equivalent to the following command. 
  ```
  wsk action create example exec.zip --docker openwhisk/dockerskeleton
  ```
  {: pre}

## Creating Go actions
{: #creating-go-actions}

The `--native` option allows for packaging of any executable as an action. This works for Go as an example.
As with Docker actions, the Go executable receives a single argument from the command line.
It is a string serialization of the JSON object representing the arguments to the action.
The program may log to `stdout` or `stderr`.
By convention, the last line of output _must_ be a stringified JSON object which represents the result of the action.

Here is an example Go action.
```go
package main

import "encoding/json"
import "fmt"
import "os"

func main() {
    //program receives one argument: the JSON object as a string
    arg := os.Args[1]
   
    // unmarshal the string to a JSON object
    var obj map[string]interface{}
    json.Unmarshal([]byte(arg), &obj)

    // can optionally log to stdout (or stderr)
    fmt.Println("hello Go action")

    name, ok := obj["name"].(string)
    if !ok { name = "Stranger" }

    // last line of stdout is the result JSON object as a string
    msg := map[string]string{"msg": ("Hello, " + name + "!")}
    res, _ := json.Marshal(msg)
    fmt.Println(string(res))
}
```

Save the code above to a file `sample.go` and cross compile it for OpenWhisk. The executable must be called `exec`.
```bash
GOOS=linux GOARCH=amd64 go build -o exec
zip exec.zip exec
wsk action create helloGo --native exec.zip
```

The action may be run as any other action.
```bash
wsk action invoke helloGo -r -p name gopher
{
    "msg": "Hello, gopher!"
}
```

Logs are retrieved in a similar way as well.
```bash
wsk activation logs --last --strip
my first Go action.
```

## Creating native actions
{: #creating-native-actions}

Using `--native`, you can see that any executable may be run as an OpenWhisk action. This includes `bash` scripts,
or cross compiled binaries. For the latter, the constraint is that the binary must be compatible with the
`openwhisk/dockerskeleton` image.

## Monitor action output
{: #watching-action-output}

{{site.data.keyword.openwhisk_short}} Actions might be invoked by other users, in response to various events, or as part of an action sequence. In such cases, it can be useful to monitor the invocations.

You can use the {{site.data.keyword.openwhisk_short}} CLI to watch the output of Actions as they are invoked.

1. Issue the following command from a shell:
  ```
  wsk activation poll
  ```
  {: pre}

  This command starts a polling loop that continuously checks for logs from activations.

2. Switch to another window and invoke an action:

  ```
  wsk action invoke /whisk.system/samples/helloWorld --param payload Bob
  ```
  {: pre}

  ```
  ok: invoked /whisk.system/samples/helloWorld with id 7331f9b9e2044d85afd219b12c0f1491
  ```

3. Observe the activation log in the polling window:

  ```
  Activation: helloWorld (7331f9b9e2044d85afd219b12c0f1491)
    2016-02-11T16:46:56.842065025Z stdout: hello bob!
  ```

  Similarly, whenever you run the poll utility, you see in real time the logs for any Actions that are run on your behalf in OpenWhisk.


## List Actions
{: #listing-actions}

You can list all the Actions created using the following command:

```
wsk action list
```
{: pre}

As you write more Actions, this list gets longer and it can be helpful to group related Actions into [packages](./openwhisk_packages.html). To filter your list of Actions to just those within a specific package, you can use the following command syntax: 

```
wsk action list [PACKAGE NAME]
```
{: pre}


## Delete Actions
{: #deleting-actions}

You can clean up by deleting Actions that you do not want to use.

1. Run the following command to delete an action:
  ```
  wsk action delete hello
  ```
  {: pre}

  ```
  ok: deleted hello
  ```

2. Verify that the action no longer appears in the list of Actions.
  ```
  wsk action list
  ```
  {: pre}

  ```
  actions
  ```

## Access action metadata within the action body
{: #accessing-action-metadata-within-the-action-body}

The action environment contains several properties that are specific to the running action.
These allow the action to programmatically work with OpenWhisk assets via the REST API,
or set an internal alarm when the action is about to use up its allotted time budget.
The properties are accessible via the system environment for all supported runtimes:
Node.js, Python, Swift, Java and Docker Actions when using the OpenWhisk Docker skeleton.

* `__OW_API_HOST` the API host for the OpenWhisk deployment running this action
* `__OW_API_KEY` the API key for the subject invoking the action, this key may be a restricted API key
* `__OW_NAMESPACE` the namespace for the _activation_ (this may not be the same as the namespace for the action)
* `__OW_ACTION_NAME` the fully qualified name of the running action
* `__OW_ACTIVATION_ID` the activation id for this running action instance
* `__OW_DEADLINE` the approximate time when this action will have consumed its entire duration quota (measured in epoch milliseconds)
