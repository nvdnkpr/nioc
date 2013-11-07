#NIoc (node ioc)

A simple to use IOC container for Node.js.

## License
Released under [MIT License] (https://github.com/psmithiv/nioc/blob/master/LICENSE).

## Highlights
* Beans are defined via a simple JSON file.
* Compatable with both Object as well as Class based node.js modules.
* All beans are created at application launch. 
* In the event that a bean injects a bean that has not yet been created, it will be created on demand.
* Ability to pass config objects to Class based modules and/or call bean methods post construction.
* Injection available via global 'inject' method.
* Ability to inject both complete beans as well as individual bean properties.

## Installation
From within your project, execute 'npm install nioc' from the command line. Npm will create a nioc folder within the standard node_modules folder. The module it's self will be located in 'node_modules/nioc/bin'.

## Getting Started
### Initializing NIoc
In your applications index.js/server.js file simply require the NIoc module and call the returned method passing in the path to your bean definitions json file. If no bean definitions file is specified when calling 'nioc()', NIoc will automatically look for a beans.json file one level up from 'node_modules'.

```js
var nioc = require('nioc');
nioc('path to beans.json file');
```

<b>NOTE: Since nioc.js is requiring/loading the definitions file. The path is relative to 'node_modules/nioc/bin'. It is reccomended that you set the NODE_PATH environmental variable to the root of your application and reference your bean definitions json file from there.</b>

### Defining Beans
In your beans.json file, beans are defined as objects and reqire a single 'path' property which references the node.js module to be made available for injection. 

```js
{
  "beanC": {
    "path": "path to module to be made available for injection (BeanC.js)"
  }
}
```

<b>NOTE: Since nioc.js is requiring/loading the definitions file. The path is relative to 'node_modules/nioc/bin'. It is reccomended that you set the NODE_PATH environmental variable to the root of your application and reference your bean/module file from there.</b>

Additionaly, there are two other properties that may also be specified on the bean object: 'config' and 'postConstruct'.

#### "config"
If the node.js module is a javascript class (typeof exports = module.exports === 'function'), NIoc will create a new instance of the module/class and pass the value of the config property as an attribute to the method specified.

```js
{
    "beanC" : {
        "path": "path to module to be made available for injection (BeanC.js)"
    },

    "beanAA": {
        "path": "path to module to be made available for injection (BeanA.js)",
        "config": {
            "configProp1": "foo2",
            "configProp2": "bar2"
        }
    }
}
```

#### "postConstruct"
Lastly, a 'postConstruct' array may be defined on the bean object. This property consists of an array of objects containing a 'method' property (required) and an additional 'arguments' property. The 'methods' property defines what method to call on the bean post construction, while the 'arguments' array is a list of values to be passed as attributes to the specified method.

```js
{
    "beanC" : {
        "path": "path to module to be made available for injection (BeanC.js)"
    },

    "beanAA": {
        "path": "path to module to be made available for injection (BeanA.js)",
        "config": {
            "configProp1": "foo2",
            "configProp2": "bar2"
        }
    },

    "beanA": {
        "path": "path to module to be made available for injection (BeanA.js)",
        "config": {
            "configProp1": "foo",
            "configProp2": "bar"
        },
        "postConstruct": [{
            "method": "postConstructA",
            "arguments": [
                "postConstructParamString",
                100
            ]
        }]
    }
}
```

### Creating Beans
The recomended approach to creating beans for NIoc is to define class based node.js modules as seen below. This will allow for the same node.js module to be defined more than once using different id's as well as allow for using the 'config' property on the bean definition. Also, this will allow for planed future features such as specifying ' "singleton": false ' on the bean definition so that a unique instance of the bean will be returned each time inject('bean id') is called. OOP is good ;)

<b>NOTE: Object based support is in place for 3rd party modules (ex. express)</b>

```js
//wire up module
exports = module.exports = init;

/**
 * Example BeanA.js
 *
 * @class
 */
function init(config) {
    /**
     * @private
     * @type {*}
     */
    var me = this;

    /**
     * @public
     * @type {String}
     */
    me.configProp1 = '';

    /**
     * @public
     * @type {String}
     */
    me.configProp2 = '';

    /**
     * @constructor
     */
    function constructor() {
        me.configProp1 = config.configProp1;
        me.configProp2 = config.configProp2;
    };

    me.postConstructA = function(stringArgument, numberArgument) {
        console.log('INFO: BeanA.postConstructA  -  stringArgument: ' + stringArgument + '  -  numberArgument: ' numberArgument);
    }

    me.postConstructB = function(arrayArgument) {
        console.log('INFO: BeanA.postConstructB  -  arrayArgument: ' + arrayArgument);
    }

    //constructo object
    constructor();
}
```
====

To try the example just execute 'npm install nioc' from the command line in the example folder.
