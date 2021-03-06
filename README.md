[travis-url]: http://travis-ci.org/muroc/offensive.js
[travis-image]: https://travis-ci.org/muroc/offensive.js.svg?branch=master

[david-url]: https://david-dm.org/muroc/offensive.js
[david-image]: https://david-dm.org/muroc/offensive.js.svg

[david-url-dev]: https://david-dm.org/muroc/offensive.js#info=devDependencies
[david-image-dev]: https://david-dm.org/muroc/offensive.js/dev-status.svg

[npm-url]: https://npmjs.org/package/offensive
[npm-image]: https://badge.fury.io/js/offensive.svg

# offensive :facepunch: js
[![Build Status][travis-image]][travis-url]
[![Dependency Status][david-image]][david-url]
[![devDependency Status][david-image-dev]][david-url-dev]
[![NPM version][npm-image]][npm-url]

A human-readable, fast and boilerplate-free contract programming library
for JavaScript.

**Why would I want it?**

 1. It reduces the boilerplate of writing error messsages to zero,
 2. Provides very intuitive and extensible DSL for writing assertions (zero learning curve),
 3. Enables easy implementation of offensive and defensive techniques,
 4. Is compatible with web browsers (when used via [browserify][browserify])
  and with node ([v0.10][node-v0.10], [v0.12][node-v0.12], [v4.x][node-v4], [v6.x][node-v6]).

[browserify]: https://github.com/substack/node-browserify

[node]: https://github.com/nodejs/node
[node-v0.10]: https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V010.md
[node-v0.12]: https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V012.md
[node-v4]: https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V4.md
[node-v6]: https://github.com/nodejs/node/blob/master/doc/changelogs/CHANGELOG_V6.md

## Installation

```shell
npm install --save offensive
```

## Usage

### Offensive Programming

Programming offensively is about throwing exceptions a lot. As soon
as corrupted state or illegal parameter is detected, program is crashed
with a descriptive error message. This technique greatly helps in finding
bugs at&nbsp;their cause.
```js
var check = require('offensive');

// Let's say we have class `Time` with a constructor
// that accepts initializer object.
function Time(init) {
  // Contract of this constructor is satisfied
  // if init contains 'timestamp' property of type number.
  check(init, 'init').is.anObject();
  check(init.timestamp, 'init.timestamp').is.aNumber();

  ...
}
```
Now, following erroneus call...
```js
new Time({});
```
...will result in throwing following exception.
```
ContractError: init.timestamp must be a numer; got undefined
  at Object.aNumber (node_modules/offensive/lib/registry/assertion.js:59:21)
  at Time (example.js:9:24)
  at example.js:20:0
```
Above example uses only [`.anObject`][object] and [`.aNumber`][number] assertions.
[See full list of offensive.js built-in assertions][assertions].

### Defensive Programming

Offensive programming is not applicable when collaborating with
external components. We don't want our program to crash in&nbsp;response
to a bug in another program. Logging an error and trying to correct
it or simply ignoring erroneus input would be a&nbsp;better way of
handling such situation.
```js
// Now, let's create a function that fetches time data
// from a time server and returns instance of Time.
function fetchTime(url, callback) {
  check(url, 'url').is.anInstanceOf(URL);
  check(callback, 'callback').is.aFunction();

  var json = "";

  http.get(url.toString(), (res) => {
    res.on('data', (chunk) => { json += chunk; });
    res.on('end', parseAndReturn);
    res.resume();
  });

  function parseAndReturn() {
    var init = JSON.parse(json);

    // JSON received from the server may not fulfill our contract.
    // In order to prevent crashing, we need to validate this
    // external input before constructing an instance of Time.
    var context = check.defensive(init, 'init from '+ url).is.anObject;
    if (context._result) {
      context = check.defensive(init.timestamp, 'init.timestamp from '+ url).is.aNumber;
    }
    if (!context._result) {
      log(context._message);
      callback(context._message);
      return
    }

    callback(null, new Time(init));
  });
}
```

**Further Rading:**
 * [What is the difference between offensive and defensive
   programming?][defensive-design]

[defensive-design]: http://softwarephilosophy.ninja/defensive-design

## API Reference

**Table of Contents**

1. [Check Function][check]
1. [Assertions][assertions]
2. [Boolean Operators][operators]
3. [Interfaces][interfaces]

[check]: #check-function
<a id=check-function></a>
### Check Function
```js
module.exports = function check(value, name) { ... }
```
Creates a [Context][interfaces] object. All [assertions][assertions]
called on returned context will be applied to passed **value**.
In case some assertions fail, **name** will be used as part of
error message.
```js
var check = require('offensive');
...

check(arg, 'arg')...
```

[assertions]: #assertions
### Assertions

offensive.js contains following built-in assertions.

**Table of Contents**

 1. [`.Null()`][null]
 1. [`.Undefined()`][undefined]
 1. [`.Empty()`][empty]
 1. [`.ofType(requiredType)`][of-type]
 1. [`.aBoolean()`][boolean]
 1. [`.aNumber()`][number]
 1. [`.aString()`][string]
 1. [`.anObject()`][object]
 1. [`.aFunction()`][function]
 1. [`.anArray()`][array]
 1. [`.anInstanceOf(RequiredClass)`][instance-of]
 1. [`.True()`][true]
 1. [`.False()`][false]
 1. [`.truthy()`][truthy]
 1. [`.falsy()`][falsy]
 1. [`.equalTo()`][equal-to]
 1. [`.exactly()`][exactly]
 1. [`.lessThan(rightBounds)`][less-than]
 1. [`.greaterThan(leftBounds)`][greater-than]
 1. [`.inRange(leftBounds, rightBounds)`][in-range]
 1. [`.property(propertyName, propertyValue)`][property]
 1. [`.method(methodName)`][method]
 1. [`.propertyOfType(propertyName, propertyType)`][property-of-type]
 1. [`.propertyLessThan(propertyName, rightBounds)`][property-less-than]
 1. [`.propertyGreaterThan(propertyName, leftBounds)`][property-greater-than]
 1. [`.length(requiredLength)`][length]
 1. [`.oneOf(set, name)`][one-of]
 1. [`.elementThatIs(index, assertName, condition)`][element]
 1. [`.eachElementIs(assertName, condition)`][each-element]
 1. [`.onlyNumbers()`][only-numbers]
 1. [`.onlyStrings()`][only-strings]
 1. [`.onlyObjects()`][only-objects]
 1. [`.onlyFunctions()`][only-functions]
 1. [`.onlyInstancesOf(RequiredClass)`][only-instances-of]

[null]: #null-assertion
<a id=null-assertion></a>
#### `.Null()` aliases: `.null`, `.Nil`, `.nil`
Asserts that checked value is `null` using `===`.
Typically used in combination with [`.not`][not] operator.
```js
check(arg, 'arg').is.not.Null();
```

[undefined]: #undefined-assertion
<a id=undefined-assertion></a>
#### `.Undefined()` aliases: `.undefined`
Asserts that checked value is `undefined`.
Typically used in combination with [`.not`][not] operator.
```js
check(arg, 'arg').is.not.Undefined();
```

[empty]: #empty-assertion
<a id=empty-assertion></a>
#### `.Empty()` aliases: `.empty`
Asserts that checked value is `null` or `undefined`.
Typically used in combination with [`.not`][not] operator.
```js
check(arg, 'arg').is.not.Empty();
```

[of-type]: #of-type-assertion
<a id=of-type-assertion></a>
#### `.ofType(requiredType)` aliases: `.type`
Asserts that checked value is of **requiredType** by ivoking `typeof` operator.
```js
check(arg, 'arg').is.ofType('boolean');
```

[boolean]: #boolean-assertion
<a id=boolean-assertion></a>
#### `.aBoolean()` aliases: `.Boolean`, `.boolean`
Asserts that checked value is a boolean by ivoking `typeof` operator.
```js
check(arg, 'arg').is.aBoolean();
```

[number]: #number-assertion
<a id=number-assertion></a>
#### `.aNumber()` aliases: `.Number`, `.number`
Asserts that checked value is a number by ivoking `typeof` operator.
```js
check(arg, 'arg').is.aNumber();
```

[string]: #string-assertion
<a id=string-assertion></a>
#### `.aString()` aliases: `.String`, `.string`
Asserts that checked value is a string by ivoking `typeof` operator.
```js
check(arg, 'arg').is.aString();
```

[object]: #object-assertion
<a id=object-assertion></a>
#### `.anObject()` aliases: `.Object`, `.object`
Asserts that checked value is an object by ivoking `typeof` operator.
Be wary that this will be true also for array instances and `null`.
Use [`.anArray`][array] and [`.Null`][null] in order to test for these
specific cases.
```js
check(arg, 'arg').is.anObject();
```

[function]: #function-assertion
<a id=function-assertion></a>
#### `.aFunction()` aliases: `.Function`, `.function`
Asserts that checked value is a function by ivoking `typeof` operator.
```js
check(arg, 'arg').is.aFunction();
```

[array]: #array-assertion
<a id=array-assertion></a>
#### `.anArray()` aliases: `.Array`, `.array`
Asserts that checked value is an array, by performing few
[duck typing][duck-typing] method checks.
```js
check(arg, 'arg').is.anArray();
```
[duck-typing]: https://en.wikipedia.org/wiki/Duck_typing

[instance-of]: #instanceof-assertion
<a id=instanceof-assertion></a>
#### `.anInstanceOf(RequiredClass)` aliases: `.instanceOf`
Asserts that checked value is a instance of **RequiredClass**, by
using `instanceof` operator.
```js
check(arg, 'arg').is.anInstanceOf(RegExp);
```

[true]: #true-assertion
<a id=true-assertion></a>
#### `.True()` aliases: `.true`
Asserts that checked value is a boolean of value `true`.
```js
check(arg, 'arg').is.True();
```

[false]: #false-assertion
<a id=false-assertion></a>
#### `.False()` aliases: `.false`
Asserts that checked value is a boolean of value `false`.
```js
check(arg, 'arg').is.False();
```

[truthy]: #truthy-assertion
<a id=truthy-assertion></a>
#### `.truthy()` aliases: `.Truthy`, `.truethy`, `.Truethy`
Asserts that checked value is truthy (converts to `true`).
```js
check(arg, 'arg').is.truthy();
```

[falsy]: #falsy-assertion
<a id=falsy-assertion></a>
#### `.falsy()` aliases: `.Falsy`, `.falsey`, `.Falsey`
Asserts that checked value is falsy (converts to `false`).
```js
check(arg, 'arg').is.falsy();
```

[equal-to]: #equal-to-assertion
<a id=equal-to-assertion></a>
#### `.equalTo(another)` aliases: `.equal`, `.equals`
Asserts that checked value is equal to **another**.
Comparison is made with `==` (double equals) operator.
```js
check(arg, 'arg').is.equalTo(100);
```

[exactly]: #exactly-assertion
<a id=exactly-assertion></a>
#### `.exactly(another)`
Asserts that checked value is exactly the same as **another**.
Comparison is made with `===` (triple equals) operator.
```js
check(arg, 'arg').is.exactly(instance);
```

[less-than]: #less-than-assertion
<a id=less-than-assertion></a>
#### `.lessThan(rightBounds)` aliases: `.lt`, `.less`
Asserts that checked value is a number, which is less than **rightBounds**.
```js
check(arg, 'arg').is.lessThan(100);
```

[greater-than]: #greater-than-assertion
<a id=greater-than-assertion></a>
#### `.greaterThan(leftBounds)` aliases: `.gt`, `.greater`
Asserts that checked value is a number, which is greater than **leftBounds**.
```js
check(arg, 'arg').is.greaterThan(0);
```

[in-range]: #in-range-assertion
<a id=in-range-assertion></a>
#### `.inRange(leftBounds, rightBounds)` aliases: `.between`
Asserts that checked value is a number, which is grater than **leftBounds - 1**
and less than **rightBounds**.
```js
check(arg, 'arg').is.inRange(0, 100);
```

[property]: #property-assertion
<a id=property-assertion></a>
#### `.property(propertyName, propertyValue)` aliases: `.field`
Asserts that checked value has property of name **propertyName**.
It also asserts that value of the property equals **propertyValue**
(if propertyValue is present). It uses `===` operator for comparing values.
```js
check(arg, 'arg').has.property('length');
check(arg, 'arg').contains.property('nodeName', 'DIV');
```

[method]: #method-assertion
<a id=method-assertion></a>
#### `.method(methodName)`
Asserts that checked value has property of name **methodName**
which is a function.
```js
check(arg, 'arg').has.method('toString');
```

[property-of-type]: #property-of-type-assertion
<a id=property-of-type-assertion></a>
#### `.propertyOfType(propertyName, propertyType)`
Asserts that checked value has property of name **propertyName** and that this
property is of type **propertyType**. Type is checked with `typeof` operator.
```js
check(arg, 'arg').has.propertyOfType('length', 'number');
check(arg, 'arg').contains.propertyType('nodeName', 'string');
```

[property-less-than]: #property-less-than-assertion
<a id=property-less-than-assertion></a>
#### `.propertyLessThan(propertyName, rightBounds)` aliases: `.propertyLT`, `.fieldLessThan`
`Asserts that checked value has property of name **propertyName**, value
of this property is of type `number`, and that this value is less than
**rightBounds**.
```js
check(arg, 'arg').has.propertyLessThan('index', 10);
```

[property-greater-than]: #property-greater-than-assertion
<a id=property-greater-than-assertion></a>
#### `.propertyGreaterThan(propertyName, leftBounds)` aliases: `.propertyGT`, `.fieldGreaterThan`
`Asserts that checked value has property of name **propertyName**, value
of this property is of type `number`, and that this value is greater than
**leftBounds**.
```js
check(arg, 'arg').has.propertyLessThan('index', 10);
```

[length]: #length-assertion
<a id=length-assertion></a>
#### `.length(requiredLength)` aliases: `.len`
Asserts that checked value has property of name "length" and value
of **requiredLength**.
```js
check(arg, 'arg').has.length(0);
```

[one-of]: #one-of-assertion
<a id=one-of-assertion></a>
#### `.oneOf(set, name)` aliases: `.elementOf`, `.containedIn`
Asserts that checked value is contained in given **set**. Given **name** (if
present) is used as a name of set in produced error message.
```js
check(arg, 'arg').is.oneOf([ 'started', 'running', 'finished' ]);
```

[element]: #elementthatis-assertion
<a id=elementthatis-assertion></a>
#### `.elementThatIs(index, assertName, condition)` aliases: `.elementWhichIs`
Asserts that:
 1. Checked value is an array of length at least **`index`**` + 1`,
 2. Element under **index** satisfies **condition**.

**condition** must be an object implementing [`Condition`][interfaces] interface
or a `function` with signature matching [`Condition.isSatisfiedBy(arg)`][interfaces]
method. **assertName** is used as assertion name in generated error message.
```js
check(arg, 'arg').has.elementThatIs(0, "an integer", Number.isInteger);
```

[each-element]: #eachelementis-assertion
<a id=eachelementis-assertion></a>
#### `.eachElementIs(assertName, condition)` <sup>aliases: `.everyElementIs`, `.allElements`, `.onlyElements`</sup>
Asserts that:
 1. Checked value is an array,
 2. Each element of this array satisfies **condition**.

**condition** must be an object implementing [`Condition`][interfaces] interface
or a `function` with signature matching [`Condition.isSatisfiedBy(arg)`][interfaces]
method. **assertName** is used as assertion name in generated error message.
```js
check(arg, 'arg').eachElementIs("an integer", Number.isInteger);
```

[only-numbers]: #onlynumbers-assertion
<a id=onlynumbers-assertion></a>
#### `.onlyNumbers()`
Asserts that:
 1. Checked value is an array,
 2. Each element of this array is a number.
```js
check(arg, 'arg').contains.onlyNumbers();
```

[only-strings]: #onlystrings-assertion
<a id=onlystrings-assertion></a>
#### `.onlyStrings()`
Asserts that:
 1. Checked value is an array,
 2. Each element of this array is a string.
```js
check(arg, 'arg').contains.onlyStrings();
```

[only-objects]: #onlyobjects-assertion
<a id=onlyobjects-assertion></a>
#### `.onlyObjects()`
Asserts that:
 1. Checked value is an array,
 2. Each element of this array is an object.
```js
check(arg, 'arg').contains.onlyObjects();
```

[only-functions]: #onlyfunctions-assertion
<a id=onlyfunctions-assertion></a>
#### `.onlyFunctions()`

Asserts that:
 1. Checked value is an array,
 2. Each element of this array is a function.
```js
check(arg, 'arg').contains.onlyFunctions();
```

[only-instances-of]: #onlyinstancesof-assertion
<a id=onlyinstancesof-assertion></a>
#### `.onlyInstancesOf(RequiredClass)`
Asserts that:
 1. Checked value is an array,
 2. Each element of this array is an instance of **RequiredClass**.
```js
check(arg, 'arg').contains.onlyInstancesOf(MyClass);
```

[operators]: #boolean-operators
### Boolean Operators

offensive.js implements following operators.

**Table of Contents**

1. [`.and`][and]
1. [`.or`][or]
1. [`.not`][not]

[and]: #and-operator
<a id=and-operator></a>
#### `.and` aliases: `.of`, `.with`
Logical conjunction of two boolean values which are separated by call to `.and` operator.
```js
check(arg, 'arg').has.length(2).and.contains.onlyNumbers();
```

[or]: #or-operator
<a id=or-operator></a>
#### `.either()` aliases: `.weather`
#### `.or()`
Logical alternative of two values which are separated by call to `.or` operator.
Result of whole expression between `.either` and `.or` is taken as left-hand-side
argument. First result after `.or` is taken as right-hand-side argument.
```js
check(arg, 'arg').is.either.anObject.or.aFunction();
```

[not]: #not-operator
<a id=not-operator></a>
#### `.not` aliases: `.no`, `.dont`, `.doesnt`
Logical negation of a value after `.not` operator.
```js
check(arg, 'arg').is.not.Undefined();
```

[interfaces]: #interfaces
### Interfaces

See [index.d.ts][interfaces].

[interfaces]: index.d.ts

## Extension API

offensive.js is extensible, but extension API is not documented yet.
If you wish to write an extension, take a look at the implementation
of [built-in assertions][assertions-code] and [offensive.js][main-code]
file.

[assertions-code]: lib/built-ins/assertions
[main-code]: offensive.js

## License

Copyright &copy; 2016 Maciej Chałapuk.
Released under [MIT license](LICENSE).

