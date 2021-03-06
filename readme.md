![Not.Js - "All-in-one" type checking, validation, error handling and messaging.](https://dev-to-uploads.s3.amazonaws.com/i/4bs7198m86uofoyh0w6g.png)
[![npm version](https://img.shields.io/npm/v/you-are-not.svg?style=flat-square)](https://www.npmjs.com/package/you-are-not)
[![Build Status](https://badgen.net/travis/calvintwr/you-are-not?style=flat-square)](https://travis-ci.com/calvintwr/you-are-not)
[![Coverage Status](https://badgen.net/coveralls/c/github/calvintwr/you-are-not?style=flat-square)](https://coveralls.io/r/calvintwr/you-are-not)
[![license](https://img.shields.io/npm/l/you-are-not.svg?style=flat-square)](https://www.npmjs.com/package/you-are-not)
[![install size](https://badgen.net/packagephobia/install/you-are-not?style=flat-square)](https://packagephobia.now.sh/result?p=you-are-not)

>*Not* is a minimal, blazing fast, intuitive, API-centric, and customisable type-checking/validation/error handing and messaging helper -- all in a small and neat pack.

## Why *Not*?
*Not* gives **actionable** error messages, so you know exactly what has gone wrong with your inputs/arguments/API. Meet project deadlines. Code with accuracy. Build friendly APIs.

**Not** is different from TypeScript. It is for runtime checking, and is API-centric/friendly in error handling/messaging. It complements Typescript.
![Not.Js - "All-in-one" type checking, validation, error handling and messaging.](https://dev-to-uploads.s3.amazonaws.com/i/l74jrtmfy2p305fw9wvy.gif)

*This module has no dependencies.*


**Quick Example of Code Reduction**

Instead of the horrible:
```js
if (typeof foo !== 'string' ||
    typeof foo !== 'number' ||
    (typeof foo === 'number' && !isNaN(foo)) ||
    !Array.isArray(foo)
) {  throw Error("Not valid, but I don't know why.") }
```
You write:
```js
not(['string', 'number', 'array'], foo)
// or
is(['string', 'number', 'array'], foo)
```

## Simple Usage

### 1. For API input type-checking, validation, sanitisation and error messaging:

```js
let schema = {
    id: 'number',
    name: 'string'
}

// payload may be a parsed body from API-requestors, or function arguments
let payload = {
    id: "1", // this will error
    name: "foo",
    role: "admin" // simulating malicious payload. this will be sanitised
}

let sanitised = Not.checkObject(
    'objName',
    schema,
    payload,
    { returnPayload: true }
)

// if failed: Not will throw TypeError, with an `trace` array containing error messages.
// if passed: will be an Object with sanitised payload

console.log(sanitised) // here you have a sanitised payload
```
**Actionable error messages:**
```
Wrong Type (objName.id): Expecting type `number` but got `string`: "1".
```


### 2. Lightweight ("simplified mode") type-checking

Simplified mode cuts down verbiage. *Not* uses a **double negative mechanism** (it is more powerful and will be explain further below).
```js
const Not = require('you-are-not')
const not = Not.create() // this exposes a simplified #not with no overloads

let testString    = 'string'
let testNotString = 123

not('string', testString) // will return false
not('string', testNotString) // will throw error
not('integer', testNotString) // will return false
```

## Installation

```
npm i --save you-are-not
```

## Basic Usage

### 1. Disable error throwing

Require/import `NotWontThrow`:
```js
// `require` syntax
const Not = require('you-are-not').NotWontThrow
// or `import` syntax
import { NotWontThrow as Not } from 'you-are-not'      
```
Or overwrite with assignments:
```js
Not.willThrowError = false
let test1 = Not.not('string', 123)
let test2 = Not.is('number', 123)
```
Or if using simplified mode, pass in options `willThrowError: false`:
```js
let not = Not.create({ willThrowError: false })
let test = not('string', 123)
console.log(test) // 'Wrong Type: Expecting `string` but got `number`: 123.
```

### 2. Check Objects
```js
const Not = require('you-are-not')   // `require` syntax
import Not from 'you-are-not' // `import` syntax

// a schema that replicates intuitively your object structure
let schema    = { string: 'string',     number: 'number'     }

// `payload` is the object you want to test
let payload = { string: 'correct type', number: 'I am string' }

// compare `candidate` with our `schema`
let errors = Not.checkObject(
    'objectName', // specify a name for object
    schema,
    candidate
)
```
In console, Not will tell you exactly what has gone wrong with your inputs.
```
> Error: Wrong types provided, See `trace`.
      trace: [
          "Wrong Type (objectName.number): Expecting type `number` but got `string`: "I am string"."
      ]
```
Or if you have switched off error throwing:
```js
console.log(errors)
// outputs:
//"Wrong Type (objectName.number): Expecting type `number` but got `string`: "I am string"."

// In API, send user this message:
respondToUser(errors)
```

### 3. *Not* Also Has `#is`

```js
let is = Not.createIs()
is('array', []) // returns true
is('number', NaN) // throws error
```
*Note: #is does not throw by default.
Because `#is` needs to return true when the check passes, it is not as powerful as `#not`.



## Double Negative Mechanism Explained
*Not* prefers a more powerful "double negative" mechanism, to definitively return `false` when the check passes.

It follows the sensible human logic -- "This is not what I want, let's stop and do something.":
```js
if (not('string', 'i am string')) // do something
// All good, let's move on.
```

When *Not* fails, **it throws an error by default**. Or, you can switch to return a `string` **which can be used to evaluate to `true` to perform some operations!**
```js
const not = Not.create({ willThrowError: false })
// instead of throwing, `not` will return string

let input  = ['a', 'sentence']
let result = not('string', input) // returns a string, which can evaluate `true`

if (result) input = input.join(' ')
// so you can do your own error handling, or transformation

// code below can safely use `input` as string :)
input.toLowerCase()
```

## Full Usage

### 1. Options

Customise *Not* with options.

**By direct assignment to the *Not* prototype:**

```js
Not.willThrowError = true(default)/false
Not.timestamp = true/false(default)
Not.messageInPOJO = true/false(default) // messageInPOJO will only work if willThrowError is false.
```
(Note: POJO stands for **P**lain **O**ld **J**avascript **O**bject. JSON was considered except terminalogically not fully correct as the output may not always comply to JSON, given that it can contain functions.)

**Or if you wish to use the simplified mode:**

**Simplified**

Sometimes you may only need simple `#not` or `#is`. You can use the simplified methods:
```js
const Not = require('you-are-not')

let options = {
    willThrowError: false,
    timestamp: true,
    messageInPOJO: true // messageInPOJO only works if this is disabled
}
let not = Not.create(options)
let is = Not.createIs() // default usage without specifying options

not('string', 123)
is('number', 123)
```

### 2. Valid types

**The valid types you can check for are:**
```js
Primitives:
'string'
'number'
'array'
'object'
'function'
'boolean'
'null'
'undefined'
'symbol'
'nan' // this is an opinion. NaN should not be of type number in the literal sense.

Aggregated:
'optional' // which means 'null' and 'undefined'

Other custom types:
'integer'
```

### 3. Checking for Multiple Types

You can also check for multiple types by passing an array. This is useful when you want your API to accept both string and number:
```js
let not = Not.create()
let id        = "123"
let anotherId = 123
let emailOptional = undefined

not(['string', 'number'], id)
not(['string', 'number'], anotherId)
not(['optional', 'string'], emailOptional) // if available, but be string.
// all will evaluate to false.
```



### 4. Methods Available

**The *Not* prototype has the following methods available:**
```js
Not.not(expect, got, name, note)
Not.is(expect, got, name, note)
Not.checkObject(objectName, schema, payload, options)

Not.lodge(expect, got, name, note)
Not.resolve([callback]) // this is used with #lodge.

Not.defineType(options)
```

### 5. Methods: `#not` and `#is`
```js
Not.not(expect, got, name, note)
Not.is(expect, got, name, note)
```
`expect`: (string or array of strings) The types to check for (see below on "3. Types to check for".

`got`: (any) This is the the subject/candidate/payload you are checking.

`name`: (string | optional) You can define a name of the subject/candidate/payload, which will be included into the error message.

`note`: (string | optional) Any additional notes you wish to add to the error message.

**Returns:**
1. If passed: `false`.
2. If failed: throws `TypeError` (default), `string`  (if `willNotThrow: false`) or `POJO/JSON` (if `messageInPOJO: true`).

### 6. Methods: `#checkObject`

```js
Not.checkObject(objectName, schema, payload, options)
```

`objectName`: (string) Name of object.

`schema`: (object) An object depicting your schema.

`payload`: (object) The payload to check for.

`callback/options`: (function or object | optional). To receive a sanitised payload, set options `returnPayload` to `true` (see examples 2 or 3 below).

**Returns:**
1. If `callback/options` is a `callback` function, it will run the `callback`:
```js
checkObject(name, schema, payload, function(errors) {
    // do something with errors.
})
```
(Note: When callback is provided, Not assumes you want to handle things yourself, and will not throw errors regardless of the `willThrowError` flag.)

2. If `callback/options` is `{ returnPayload: true }`, `#checkObject` returns (a) the sanitised payload (object) when check passes, (b) or an array of errors if check fails:
```js
let sanitised = checkObject(
    name,
    schema,
    payload,
    { returnPayload: true }
)
if (Array.isArray(sanitised) {
    // do something with the errors
    return
}
// or continue using the sanitised payload.
DB.find(sanitised)
```

3. If `callback/options` is `{ callback: function() {}, returnPayload: true }`:
```js
let callback = function(errors, payload) {
    if(errors.length > 0) {
        // do something with the errors
        return
    }

    DB.find(payload)
}

checkObject(
    name,
    schema,
    payload,
    {
        returnPayload: true,
        callback: callback
    }
)
```

### 7. Methods: `#checkObject` - More on Defining Schema

The schema looks exactly like the payload you are expecting.

*Not* also has optional notation:
```js
// you can use optional notations like this:
"info?": {
    gender: 'string',
    "age?": 'number'
}
//is same as
info__optional: {
    gender: 'string',
    age__optional: 'number'
}
//is same as
info__optional: {
    gender: 'string',
    age: ['number', 'optional']
}
```

### 8. Methods: `#checkObject` - Sanitisation Logic

*Not* can sanitise your payload:

```js
let schema = {
    valid: 'string',
    toSanitise: {
        keepThis: 'array'
    }
}
let payload = {
    valid: 'abc',
    toSanitise: {
        keepThis: [],
        omitThis: 123
    }
}
let sanitised = Not.checkObject(
    'request',
    schema,
    payload, {
        returnPayload: true
    }
})

// if `sanitised` is an array, means something failed.
if(Array.isArray(sanitised)) {
    throw sanitised
}

console.log(sanitised)
// outputs:
{
    valid: 'abc',
    toSanitise: {
        keepThis: []
    }
}
```
## 9. Methods: `#defineType`: Define your own checks

#### Simple example
*Not* has a built-in custom type called `integer`, and suppose if you were to define it yourself, it will look like this:
```js
Not.defineType({
    primitive: 'number', // you must define your primitives
    type: 'integer', // name your test
    pass: function(candidate) {
        return candidate.toFixed(0) === candidate.toString()
        // or ES6:
        // return Number.isInteger(candidate)
    }
})
Not.not('integer', 4.4) // gives error message
Not.is('integer', 4.4) // returns false

```
#### Advanced example

Having trouble with empty `[]` or `{}` that sometimes is `false` or `null` or `undefined`?
Define a "falsey" type like this:

```js
let is = Not.createIs({ willThrowError: false })
Not.defineType({
    primitive: ['null', 'undefined', 'boolean', 'object', 'nan', 'array' ],
    type: 'falsey',
    pass: function(candidate) {
        if (is('object', candidate)) return Object.keys(candidate).length === 0
        if (is('array', candidate)) return candidate.length === 0
        if (is('boolean', candidate)) return candidate === false
        // its the other primitives null, undefined and nan
        // which is to be passed as falsey straight away without checking
        return true
    }
})

Not.not('falsey', {}) // returns false
Not.not('falsey', [null]) // returns error message
Not.is('falsey', []) // returns true
Not.is('falsey', undefined) // returns true
Not.is(['falsey', 'function'], function() {}) // returns true
```

### 10. Methods: `#lodge` and `#resolve`


You can also use `#lodge` And `#resolve` to bulk checking with more control:

```js
// create a descendant
let apiNot = Object.create(Not)

apiNot.lodge('string', request.name, 'name')
apiNot.lodge('boolean', request.subscribe, 'subscribe')
apiNot.lodge(['string', 'array'], request.friends, 'friends')
apiNot.lodge(['number', 'string'], request.age, 'age')
// and many more lines

apiNot.resolve()
/* OR */
apiNot.resolve(errors => {
    // optional callback, custom handling
    throw errors
})
```
(Note: This will not return any payload, since you intended to micro-manage.)


### 11. POJO Output
`messageInPOJO: true`
```js
let not = Not.create({
    messageInPOJO: true,
    willThrowError: false
})
not('array', { wrong: "stuff" }, 'payload', 'I screwed up.')
//outputs:
{
    message: 'Wrong Type (payload): Expect type `array` but got `object`: { wrong: "stuff" }. I screwed up.',
    expect: 'array',
    got: { wrong: "stuff" },
    gotType: 'object',
    name: 'payload',
    note: 'I screwed up.',
    timestamp: undefined
}
```


## Info: *Not*'s Type-Checking Logic ("Opinions")
**Native Javscript typing has a few quirks:**
```js
typeof [] // object
typeof null // object
typeof NaN // number
```
Those are technically not wrong (or debatable), but often gets in the way.

**By default, *Not* will apply the following treatment:**
1. `NaN` is not a **'number'**, and will be **'nan'**.
2. `Array` and `[]` are of **'array'** type, and not **'object'**.
3. `null` is **'null'** and not an **'object'**.
4. Instance of `new` `#String`, `#Number` or `#Boolean` are their not of type **'object'** but their respective types.

**Switch Off *Not*'s Opinions:**

You can switch off opinionated type-checking:
```js
let not = Not.create({ isOpinionated: false })
```
When false, all Javascript the quirks will be restored, on top of *Not*'s opinions: An `Array` will both be an **'array'** as well as **'object'**, and `null` will both be **'null'** and **'object'**:
```js
not('object', []) // returns false -- `[]` is an object
not('array', []) // returns false -- `[]` is an array
not('object', null) // returns false -- `null` is an object
```
**Switch Off Opinions Partially:**
```js
// both #createIs and #create can take in the same options
let NotWithPartialOpinions = Not.createIs({
    opinionatedOnNaN:     false,
    opinionatedOnArray:   false,
    opinionatedOnNull:    false,
    opinionatedOnString:  false,
    opinionatedOnNumber:  false,   
    opinionatedOnBoolean: false
})

// or mutate the object before instantiating.
let NotWithPartialOpinions = Object.create(Not)
Object.assign(NotWithPartialOptions, {
    opinionatedOnNaN:     false,
    opinionatedOnArray:   false,
    opinionatedOnNull:    false,
    opinionatedOnString:  false,
    opinionatedOnNumber:  false,   
    opinionatedOnBoolean: false
})
let not = NotWithPartialOpinions.create()
let is  = NotWithPartialOpinions.createIs()
```

## More Advanced Usage
### Customise your message, by replacing the #msg method
You have to mutate the prototype:
```js
const CustomNot = require('you-are-not')

//overwrite the msg function with your own
CustomNot.msg = function(expect, got, gotType, name, note) {
    let msg = 'Hey there! We are sorry that something broke, please try again!'
    let hint = ` [Hint: (${name}) expect ${expect} got ${gotType} at ${note}.]`

    // return different messages depending on environment
    return global.isDeveloperMode ? msg += hint : msg
}
```

```js
let customNot = CustomNot.create()
global.isDeveloperMode = true
customNot('string', [], 'someWrongInput', 'file.js - xxx function')
```
Will throw:
```
! Error: Hey there! We are sorry that something broke, please try again! [Hint: (someWrongInput) expect string got array at file.js - xx function. ]
```

## License

*Not* is MIT licensed.
