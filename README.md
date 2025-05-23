# Object.{get, set}

# Table of content:
+ [Motivations](#motivations)
+ [Features](#features)
+ [Improvements](#improvements)
+ [Open-ended questions](#open-ended-questions)
+ [Similar possibilities in other libraries](#similar-possibilities-in-other-libraries)
    + [Lodash](#lodash)
    + [Rambda](#rambda)
    + [underscore](#underscore)
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
console.log(Object.get(obj, ['c', 'd', 'e.f', 'g.h'])); // undefined

Object.set(obj, ['a'], 78);
console.log(Object.get(obj, ['a'])); // 78

Object.set(obj, ['a', 'q', 'z'], 87);
console.log(Object.get(obj, ['a'])); // 78
console.log(Object.get(obj, ['a', 'q'])); // undefined
console.log(Object.get(obj, ['a', 'q', 'z'])); // undefined

Object.set(obj, ['c', 'd', 'e'], 123);
console.log(Object.get(obj, ['c', 'd', 'e'])); // 123

Object.set(obj, ['c', 'd.e', 'f', 'g.h'], [34, 67]);
console.log(Object.get(obj, ['c.d', 'e', 'f.g', 'h'])); // [34, 67]
```

# Features

# Improvements

# Open-ended questions

# Similar possibilities in other libraries

### Lodash

### Rambda

### underscore

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
