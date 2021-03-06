What is this?
=============

A minimal javascript library provides utility functions for working on objects.

Why this?
=========

I know, there're ton of libraries for the purpose out there, I tried many of
  them, they're both good and bad depending on the jugdement perspective. 
 
But I come up with this because I want:

* *Clarify what mutates and what immutates the origin object* - it has `set` - immutates, `inset` - mutates.
* *Don't revent the wheel* - Only provide what built-in JS doesn't provide. So it doesn't include `Object.*` and `Array.*`.
* *Extensibility* - `fset` and `finset` come for this, do whatever you want with the old value then return the new value.
* *Work with React/Redux states* - See `set` and `fset` examples below.
* *No surprise* - `set`, `fset`, `inset` and `finset` *won't* create intermediate objects if the path doesn't exist.

How to use?
===========

* To walk on an object
  ```javascript
  import {walk} from '@thenewvu/objutil'
  
  const nodepaths = []
  const obj = {
    a: {b: {c: 1}},
    x: {y: {z: 2}}
  }
  walk(obj, (node, path) => {
    nodepaths.push({node, path})
  })
  expect(nodepaths).toEqual([
    {node: {b: {c: 1}}, path: 'a'},
    {node: {c: 1}, path: 'a.b'},
    {node: 1, path: 'a.b.c'},
    {node: {y: {z: 2}}, path: 'x'},
    {node: {z: 2}, path: 'x.y'},
    {node: 2, path: 'x.y.z'}
  ])
  ```
* To get a value in an object:
  ```javascript
  import {get} from '@thenewvu/objutil'

  const obj = {
    a: {b: {c: 1}},
    x: {y: {z: 2}}
  }
  expect(get(obj, 'a.b.c')).toEqual(1)
  expect(get(obj, 'a.b')).toEqual({c: 1})
  expect(get(obj, 'x.y.z')).toEqual(2)
  expect(get(obj, 'x.y')).toEqual({z: 2})
  expect(get(obj, 'x.a')).toEqual(undefined)
  ```
* To test if an object matches another object
  ```javascript
  import {test} from '@thenewvu/objutil'

  const obj1 = {
    a: {b: {c: 1}},
    x: {y: {z: 2}}
  }
  const obj2 = {
    a: {b: {c: 1}}
  }
  const obj3 = {
    x: {y: {z: 1}}
  }
  expect(test(obj1, obj2)).toEqual(true)
  expect(test(obj1, obj3)).toEqual(false)
  ```
* To mutate an object
  ```javascript
  import {inset, finset} from '@thenewvu/objutil'

  const obj = {
    a: {b: {c: 'old'}},
  }
  inset(obj, 'a.b.c', 'new')
  expect(obj).toEqual({
    a: {b: {c: 'new'}},
  })
  finset(obj, 'a.b.c', c => c + 'new')
  expect(obj).toEqual({
    a: {b: {c: 'newnew'}},
  })
  ```
* To set an object immutably (don't change the origin object, return the changed object)
  ```javascript
  import {set, fset} from '@thenewvu/objutil'

  const obj1 = {
    a: {b: {c: 'old', d: {e: 'old'}}},
    x: {y: {z: 'old'}}
  }
  const {a} = obj1
  const {b} = a
  const {c, d} = b
  const {e} = d
  const {x} = obj1
  const {y} = x
  const {z} = y

  const obj2 = set(obj1, 'a.b.c', 'new')
  expect(obj2 !== obj1).toEqual(true)
  expect(obj2.a !== a).toEqual(true)
  expect(obj2.a.b !== b).toEqual(true)
  expect(obj2.a.b.c === 'new').toEqual(true)
  expect(obj2.a.b.d === d).toEqual(true)
  expect(obj2.a.b.d.e === e).toEqual(true)
  expect(obj2.x === x).toEqual(true)
  expect(obj2.x.y === y).toEqual(true)
  expect(obj2.x.y.z === z).toEqual(true)

  const obj3 = fset(obj1, 'a.b.d.e', e => e + 'new')
  expect(obj3 !== obj1).toEqual(true)
  expect(obj3.a !== a).toEqual(true)
  expect(obj3.a.b !== b).toEqual(true)
  expect(obj3.a.b.c === c).toEqual(true)
  expect(obj3.a.b.d !== d).toEqual(true)
  expect(obj3.a.b.d.e === 'oldnew').toEqual(true)
  expect(obj3.x === x).toEqual(true)
  expect(obj3.x.y === y).toEqual(true)
  expect(obj3.x.y.z === z).toEqual(true)

  expect(obj1.a === a).toEqual(true)
  expect(obj1.a.b === b).toEqual(true)
  expect(obj1.a.b.c === c).toEqual(true)
  expect(obj1.a.b.d === d).toEqual(true)
  expect(obj1.a.b.d.e === e).toEqual(true)
  expect(obj1.x === x).toEqual(true)
  expect(obj1.x.y === y).toEqual(true)
  expect(obj1.x.y.z === z).toEqual(true)
  ```
* To execute multiple operations on an object immutably
  ```javascript
  import {exec} from '@thenewvu/objutil'

  const obj1 = {
    a: {b: {c: 'old'}},
    x: {y: {z: 'old'}},
    i: {j: {k: 'old'}}
  }
  const {a} = obj1
  const {b} = a
  const {c} = b
  const {x} = obj1
  const {y} = x
  const {z} = y
  const {i} = obj1
  const {j} = i
  const {k} = j

  const obj2 = exec.on(obj1)(
    exec.fset('a.b.d', d => d || {}),
    exec.fset('a.b.d.e', e => 'new'),
    exec.set('a.b.c', 'new'),
    exec.set('x.y.z', 'new')
  )

  // only change what need to be changed
  expect(obj2 !== obj1).toEqual(true)
  expect(obj2.a !== a).toEqual(true)
  expect(obj2.a.b !== b).toEqual(true)
  expect(obj2.a.b.c === 'new').toEqual(true)
  expect(obj2.a.b.d).toEqual({e: 'new'})
  expect(obj2.x !== x).toEqual(true)
  expect(obj2.x.y !== y).toEqual(true)
  expect(obj2.x.y.z === 'new').toEqual(true)
  expect(obj2.i === i).toEqual(true)
  expect(obj2.i.j === j).toEqual(true)
  expect(obj2.i.j.k === k).toEqual(true)
  
  // no change on origin object
  expect(obj1.a === a).toEqual(true)
  expect(obj1.a.b === b).toEqual(true)
  expect(obj1.a.b.c === c).toEqual(true)
  expect(obj1.a.b.d === undefined).toEqual(true)
  expect(obj1.x === x).toEqual(true)
  expect(obj1.x.y === y).toEqual(true)
  expect(obj1.x.y.z === z).toEqual(true)
  expect(obj1.i === i).toEqual(true)
  expect(obj1.i.j === j).toEqual(true)
  expect(obj1.i.j.k === k).toEqual(true)
  ```
