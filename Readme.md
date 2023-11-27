# Node.js Design Patterns
- Contains Summarised Notes from the Book Nodejs Design Patterns 3rd Edition [Link 📕](https://www.nodejsdesignpatterns.com/)

# Chapter - 1 - The NodeJs Philosophy

### Small Module

1.  Easier to maintain and use
2.  Simpler to test and maitain
3.  Small in size & perfect for use in browser.
4.  DRY(Don't Repeat yourself) Principle applied at a whole new level.

### Small Surface Area

1.  Expose minimial set of functionality to outside world for use.

## How Node.js works

### Blocking I/O

eg: Using multiple threads to process multiple connections

- thread is not cheap in terms of system resources—it consumes memory and causes
  context switches—so having a long-running thread for each connection and not
  using it for most of the time means wasting precious memory and CPU cycles.

### Non - Blocking I/O

- In this operating mode, the
  system call always returns immediately without waiting for the data to be read or
  written. If no results are available at the moment of the call, the function will simply
  return a predefined constant, indicating that there is no data available to return at
  that moment.
- The most basic pattern for dealing with this type of non-blocking I/O is to actively
  poll the resource within a loop until some actual data is returned. This is called <strong>busywaiting</strong>
- Not Efficient, Huge Amount of wasted CPU time.

```
resources = [socketA, socketB, fileA]
while (!resources.isEmpty()) {
    for (resource of resources) {
        // try to read
        data = resource.read()
        if (data === NO_DATA_AVAILABLE) {
            // there is no data to read at the moment
            continue
        }
        if (data === RESOURCE_CLOSED) {
            // the resource was closed, remove it from the list
            resources.remove(i)
        } else {
            //some data was received, process it
            consumeData(data)
        }
    }
}
```

### Synchronous event demultiplexer

- The demultiplexer is set up with the group of resources to be watched.
  The call is synchronous and blocks until
  any of the watched resources are ready for read. When all the events are processed, the flow
  will block again on the event demultiplexer until new events are again
  available to be processed. This is called the <strong>event loop</strong>
  ![Synchronous event demultiplexer](./img/Synchronous_event_demultiplexer.png)

### The Reactor Pattern

- Idea: To have a handler associated with each I/O operation, Handler : represented by callback /promises in Node.js.
  ![Reactor Pattern](./img/reactor_pattern.png)

### Libuv, the I/O engine of Node.js

- implements the reactor
  pattern, thus providing an API for creating event loops, managing the event queue,
  running asynchronous I/O operations, and queuing other types of task.
- To build full platform:

1. set of bindings responsible for wrapping and exposing libuv and other
   low-level functionalities to JavaScript.
2. V8 engine
3. core JavaScript library that implements the high-level Node.js API

<hr/>
<br/>

# Chapter - 2 - The Module System

# Need for Modules

1. Having a way to split the codebase into multiple files
2. Allowing code reuse across different projects
3. Encapsulation
4. Managing Dependencies

# revealing module pattern

- Namespacing is the practice of creating an object which encapsulates functions and variables that have the same name as those declared in the global scope.
- One of the bigger problems with JavaScript in the browser is the lack of
  namespacing.
- To overcome this we have revealing module pattern.

```
const myModule = (() => {
 const privateFoo = () => {}
 const privateBar = []
 const exported = {
 publicFoo: () => {},
 publicBar: () => {}
 }
 return exported
})() // once the parenthesis here are parsed, the function
 // will be invoked
console.log(myModule)
console.log(myModule.privateFoo, myModule.privateBar)
```

- The above fn is an IIFE(immediately invoked function expression) & used to create private scope, exporting only parts that are meant to be public.
- the idea behind this pattern is used as a base for the
  CommonJS module system

### module.exports versus exports

| S.no | Module.exports                                                                                                           | Exports                                                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| 1    | When we want to export a single class/variable/function from one module to another module, we use the module.exports way | When we want to export multiple variables/functions from one module to another, we use exports way. |

### Resolving alorithm

- Is the core part behind the robustness of the Node.js
  dependency management, and it makes it possible to have hundreds or even
  thousands of packages in an application without having collisions or problems
  of version compatibility.
- The resolving algorithm is applied transparently for us when we
  invoke require().
- The resolving algorithm can be divided into the following three
  major branches: <br/>
  • File modules: If moduleName starts with /, it is already considered an
  absolute path to the module and it's returned as it is. If it starts with ./,
  then moduleName is considered a relative path, which is calculated starting
  from the directory of the requiring module.<br/>
  • Core modules: If moduleName is not prefixed with / or ./, the algorithm will
  first try to search within the core Node.js modules.<br/>
  • Package modules: If no core module is found matching moduleName, then
  the search continues by looking for a matching module in the first node\_
  modules directory that is found navigating up in the directory structure
  starting from the requiring module. The algorithm continues to search for
  a match by looking into the next node_modules directory up in the directory
  tree, until it reaches the root of the filesystem.

### Exporting a Function

Substack Pattern : reassigning the whole module.exports variable to a function.

```
module.exports = (message) => {
 console.log(`info: ${message}`)
}
```

Named Export

```
module.exports.verbose = (message) => {
 console.log(`info: ${message}`)
}
```

### Exporting an instance

```
class Logger {
    constructor(name) {
        this.count = 0
        this.name = name
    }
    log(message) {
        this.count++
        console.log('[' + this.name +  ' ' + this.count + '] ' + message)
    }
}

module.exports = new Logger('DEFAULT')
```

Monkey Patching : A module can modify other modules or objects in the global scope;
eg:

```
// file patcher.js
// ./logger is another module

require('./logger').customMessage = function () {
 console.log('This is a new functionality')
}
```

using new patcher module

```
// file main.js

require('./patcher')
const logger = require('./logger')
logger.customMessage()
```

### ESM: ECMAScript modules

- The most important differentiator between ESM and CommonJS is that ES modules
  are static, which means that imports are described at the top level of every module
  and outside any control flow statement.

```
//not valid code

if (condition) {
 import module1 from 'module1'
}
```

- It's very important to note that, as opposed to CommonJS, with
  ESM we have to specify the file extension of the imported modules.

```
//to avoid nameclash we can import as below.

import {log as log2} from './logger.js';
log2('message from log2');
```

### Default exports and imports

- A default export makes use of the
  export default keywords and it looks like this

```
export default function hello () {}
```

- In this case the `hello()` is ignored and entitiy exported is registered under the name default. `[Module] { default: [Function: hello] }`
- We cannout use default to import as it is a reserved keyword.
- Import , we don't have to wrap the import name around brackets or use the
  as keyword when renaming.

```
import sayHello from './hello.js';
```

### Async imports

```
eg:
import(translationModule) 
    .then((strings) => { 
        console.log(strings.HELLO)
    })
```
### Read-only live bindings
- Another fundamental characteristic of ES modules, which helps with cyclic
dependencies, is the idea that imported modules are effectively read-only live
bindings to their exported values.
```
// counter.js
export let count = 0

//main.js
import { count, increment } from './counter.js';

console.log(count) // prints 0
increment()
console.log(count) // prints 1
count++ // TypeError: Assignment to constant variable!

```

### Commonjs vs ESM
| S.no | CommonJs                                                                                                         | ESM                                                                                             |
| ---- | ------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------- |
| 1    | doesnt run in strict mode | runs in strict mode. |
| 2 |this equals exports | this is undefined|
| 3 | we dont have read only bindings |  we have read only bindings| 
| 4 | files by default is in cjs format | change file to .mjs / define type :module in packageJson| 
<br/>
<hr/>

# Chapter - 3 - Callbacks and Events