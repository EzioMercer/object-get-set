# Object.{get, set}

# Table of content:
+ [Motivations](#motivations)
+ [Open-ended questions](#open-ended-questions)
  + [Delimiter](#delimiter)
  + [Number like keys](#number-like-keys)
  + [Path like an iterable object](#path-like-an-iterable-object)
  + [Primitive value as a target](#primitive-value-as-a-target)
  + [`Object.set`](#objectset)
    + [Should it mutate or shallowly copy the target?](#should-it-mutate-or-shallowly-copy-the-target)
    + [Should it return anything?](#should-it-return-anything)
    + [Should it create and array?](#should-it-create-and-array)
+ [Similar possibilities in libraries](#similar-possibilities-in-libraries)
    + [Lodash](#lodash)
      + [_.get](#_-get)
      + [_.set](#_-set)
    + [Rambda](#rambda)
      + [R.path](#rpath)
      + [R.assocPath](#rassocpath)
    + [underscore](#underscore)
      + [_.get](#_-get-1)
+ [Links](#links)
    + [Discussions](#discussions)

# Motivations

This proposal suggests the new static methods for Object which will work with a path instead of a specific key:
+ get
+ set

Both of them are mostly inspired by libraries which are providing similar functionality

Path like a string:

```js
const obj = {
  a: 1,
  b: 2,
  c: {
    d: {
      e: 54
    }
  }
}

console.log(Object.get(obj, 'a')); // 1
console.log(Object.get(obj, 'a.b')); // undefined
console.log(Object.get(obj, 'c.d.e')); // 54
console.log(Object.get(obj, 'c.d.e.f.g.h')); // undefined

Object.set(obj, 'a', 78);
console.log(Object.get(obj, 'a')); // 78

Object.set(obj, 'a.q.z', 87);
console.log(Object.get(obj, 'a')); // 78
console.log(Object.get(obj, 'a.q')); // undefined
console.log(Object.get(obj, 'a.q.z')); // undefined

Object.set(obj, 'c.d.e', 123);
console.log(Object.get(obj, 'c.d.e')); // 123

Object.set(obj, 'c.d.e.f.g.h', [34, 67]);
console.log(Object.get(obj, 'c.d.e.f.g.h')); // [34, 67]
```

Path like an array (for keys which are hard or impossible to represent with joining by dot):

```js
const obj = {
  a: 1,
  b: 2,
  c: {
    d: {
      e: 54
    }
  }
}

console.log(Object.get(obj, ['a'])); // 1
console.log(Object.get(obj, ['a', 'b'])); // undefined
console.log(Object.get(obj, ['c', 'd', 'e'])); // 54
console.log(Object.get(obj, ['c', 'd', 'e', 'f', 'g', 'h'])); // undefined

Object.set(obj, ['a'], 78);
console.log(Object.get(obj, ['a'])); // 78

Object.set(obj, ['a', 'q', 'z'], 87);
console.log(Object.get(obj, ['a'])); // 78
console.log(Object.get(obj, ['a', 'q'])); // undefined
console.log(Object.get(obj, ['a', 'q', 'z'])); // undefined

Object.set(obj, ['c', 'd', 'e'], 123);
console.log(Object.get(obj, ['c', 'd', 'e'])); // 123

Object.set(obj, ['c', 'd', 'e', 'f', 'g', 'h'], [34, 67]);
console.log(Object.get(obj, ['c', 'd', 'e', 'f', 'g', 'h'])); // [34, 67]
```

Let's say that you request settings of one of your clients from API, and it returns the JSON.
You want to set a default value in a deeply nested setting.
Without this proposal, you will write something like this:

```js
const settings = await fetch("some-url");

settings.feed ??= {};
settings.feed.filters ??= {}
settings.feed.filters.priceRange ??= {};
settings.feed.filters.priceRange.min ??= 0;
settings.feed.filters.priceRange.max ??= 100;
```

but with the proposal, I can rewrite it like this:

```js
const settings = await fetch("some-url");

const priceRangePath = "feed.filters.priceRange";
const priceMaxPath = `${priceRangePath}.max`;
const priceMinPath = `${priceRangePath}.min`;

const priceMin = Object.get(settings, priceMinPath) ?? 0;
const priceMax = Object.get(settings, priceMaxPath) ?? 100;

Object.set(settings, priceMinPath, priceMin);
Object.set(settings, priceMaxPath, priceMax);

// Or like one-liner

Object.set(settings, priceMinPath, Object.get(settings, priceMinPath) ?? 0);
Object.set(settings, priceMaxPath, Object.get(settings, priceMaxPath) ?? 100);
```

And now if one day the filters move from `feed` into `user-preferences`, I will just rewrite the `priceRangePath`.
Also, as you can see, I can easily create the abstraction on the paths that I need and change them only in one place.
I understand that I can also create abstractions with functions for the previous method,
but for me, string constants are much less likely to cause bugs

# Open-ended questions

## Delimiter

Should a delimiter be a dot?

It is very intuitive but may be there are better options

## Number like keys

How should number keys be implemented?

Possible options:

+ Only like a string (`'a.0.b'` or `['a', '0', 'b']`)
+ Only like a number (`'a.0.b'` or `['a', 0, 'b']`)
+ Only in square brackets but separate from a previous key (`'a.[0].b'` or `['a', '[0]', 'b']`)
+ Only in square brackets but right next a previous key (`'a[0].b'` or `['a', '[0]', 'b']`)
+ Any combination of previous options

## Path like an iterable object

Can a path be represented like a non-Array object with `[Symbol.iterator]` symbol

## Primitive value as a target

What should happen if instead of an object, the number or string or etc. is passed?
Should we throw an Error or do nothing?

## `Object.set`

### Should it mutate or shallowly copy the target?

For example `loadash` mutates the original object but `Rambda` doesn't.
I prefer to mutate the original object because in my practice I need it the most 

### Should it return anything?

Of course every method returns at least `undefined` I mean should it return anything but `undefined`?
We can return:
+ the reference to a target object (doesn't matter will method mutating or not the target)
+ the reference to a new object
+ boolean which will represent an operation successful or not (closely related to [Primitive value as a target](#primitive-value-as-a-target)).

  For example, an object / property can be frozen, non-writable and controlled by Proxy

### Should it create and array?

```js
const path = "a.b.0.c";
const target = {a: {}};

Object.set(target, path, 1);

console.log(target);
```

What should be logged out

```js
{
    a: {
        b: [
            {
                c: 1
            }
        ]
    }
}
```

or

```js
{
    a: {
        b: {
            '0': {
                c: 1
            }
        }
    }
}
```
? 

# Similar possibilities in libraries

## Lodash

### [_.get](https://lodash.com/docs/4.17.15#get)

```js
var object = { 'a': [{ 'b': { 'c': 3 } }] };
 
_.get(object, 'a[0].b.c');
// => 3
 
_.get(object, ['a', '0', 'b', 'c']);
// => 3
 
_.get(object, 'a.b.c', 'default');
// => 'default'
```

### [_.set](https://lodash.com/docs/4.17.15#set)

Note: mutate

```js
var object = { 'a': [{ 'b': { 'c': 3 } }] };
 
_.set(object, 'a[0].b.c', 4);
console.log(object.a[0].b.c);
// => 4
 
_.set(object, ['x', '0', 'y', 'z'], 5);
console.log(object.x[0].y.z);
// => 5
```

## Rambda

### [R.path](https://ramdajs.com/docs/#path)

```js
R.path(['a', 'b'], {a: {b: 2}}); //=> 2
R.path(['a', 'b'], {c: {b: 2}}); //=> undefined
R.path(['a', 'b', 0], {a: {b: [1, 2, 3]}}); //=> 1
R.path(['a', 'b', -2], {a: {b: [1, 2, 3]}}); //=> 2
R.path([2], {'2': 2}); //=> 2
R.path([-2], {'-2': 'a'}); //=> undefined
```

### [R.assocPath](https://ramdajs.com/docs/#assocPath)

Note: shallow clone

```js
R.assocPath(['a', 'b', 'c'], 42, {a: {b: {c: 0}}}); //=> {a: {b: {c: 42}}}

// Any missing or non-object keys in path will be overridden
R.assocPath(['a', 'b', 'c'], 42, {a: 5}); //=> {a: {b: {c: 42}}}
```

## underscore

### [_.get](https://underscorejs.org/#get)

```js
_.get({a: 10}, 'a');
// => 10

_.get({a: [{b: 2}]}, ['a', 0, 'b']);
// => 2

_.get({a: 10}, 'b', 100);
// => 100
```

# Links

## Discussions

+ [Object.{get, set}](https://es.discourse.group/t/object-get-set/2372)

---

### P.S.
Please don't hesitate to:

+ give a star ☆
+ share your ideas
+ share this proposal with others
+ provide more useful examples
+ provide possible problems with this proposal
+ suggest text improvements
+ help with `emu` file


You can browse the [ecmarkup output](https://EzioMercer.github.io/destructuring-with-alias-binding/)
or browse the [source](https://github.com/EzioMercer/destructuring-with-alias-binding/blob/main/spec.emu)
