# proxyquire [![Build Status](https://secure.travis-ci.org/thlorenz/proxyquire.png)](http://travis-ci.org/thlorenz/proxyquire)

Proxies nodejs's require in order to make overriding dependencies during testing easy while staying **totally unobstrusive**.

# Features

- **no changes to your code** are necessary 
- non overriden methods of a module behave like the original
- "use strict" compliant


# Example

**foo.js:**

```javascript
var path = require('path');

module.exports.extnameAllCaps = function (file) { 
  return path.extname(file).toUpperCase();
};

module.exports.basenameAllCaps = function (file) { 
  return path.basename(file).toUpperCase();
};
```

**foo.test.js:**

```javascript
var proxyquire =  require('proxyquire')
  , assert     =  require('assert')
  , pathStub   =  { };

// when not overridden, path.extname behaves normally
var foo = proxyquire.resolve('./foo', __dirname, { 'path': pathStub });
assert.equal(foo.extnameAllCaps('file.txt'), '.TXT');

// override path.extname
pathStub.extname = function (file) { return 'Exterminate, exterminate the ' + file; };

// path.extname now behaves as we told it to
assert.equal(foo.extnameAllCaps('file.txt'), 'EXTERMINATE, EXTERMINATE THE FILE.TXT');

// path.basename and all other path module methods still function as before
assert.equal(foo.basenameAllCaps('/a/b/file.txt'), 'FILE.TXT');
```


# Usage

Two simple steps to override require in your tests:

- `var proxyquire = require('proxyquire');` on top level of your test file
- `proxyquire.resolve(...)` the module you want to test and pass along stubs for modules you want to override

# API

## Resolve module to be tested

***proxyquire.resolve({string} mdl, {string} test__dirname, {Object} stubs)***

- **mdl**: path to the module to be tested e.g., `../lib/foo`
- **test__dirname**: the `__dirname` of the module containing the tests
- **stubs**: key/value pairs of the form `{ modulePath: stub, ... }`
    - module paths are relative to the tested module **not** the test file 
    - therefore specify it exactly as in the require statement inside the tested file
    - values themselves are key/value pairs of functions/properties and the appropriate override

### Examples

**Assume:**
```javascript

// bar module
module.exports = { 
    toAtm: function toAtm(val) { return  0.986923267 * val; }
};


// foo module 
// requires bar which we need to stub out in tests
var bar = require('./bar');
[ ... ]

```

```javascript
// foo-test module which is one folder below foo (e.g., in ./tests/)

/**
* a) Resolve and override in one step:
*/
var foo = proxyquire.resolve('./foo', __dirname, {
  './bar': { toAtm: function (val) { return 0; /* wonder what happens now */ } }
});

// [ .. run some tests .. ]

/**
* b) Resolve with empty stub and add overrides later
*/
var barStub = { };

var foo =  proxyquire.resolve('./foo', __dirname, { './bar': barStub }); 

// Add override
bar.toAtm = function (val) { return 0; /* wonder what happens now */ };

// [ .. run some tests .. ]

// Change override
bar.toAtm = function (val) { return -1 * val; /* or now */ };

// [ .. run some tests .. ]

// Resolve and override multiple modules in one step - oh my!
var foo = proxyquire.resolve('./foo', __dirname, {
    './bar' : { toAtm: function (val) { return 0; /* wonder what happens now */ } }
  , path    : { extname: function (file) { return 'exterminate the name of ' + file } }
});
```


