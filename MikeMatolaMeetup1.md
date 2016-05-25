# Programmatically filling...

### a JavaScript array with values (without using for loops or external libraries), or, the one useful use of the Array constructor (mit apply() special sauce), or, ES6/ES2015 makes this less ugly.

### Detroit JavaScript Meetup-1 talk by Michael Matola

---

## Use Cases

* Times (behind the scenes implemented with array)
* range
* Programmatically fill array with values, per a callback function

## (Shhh!)

* All of this could be done with a for loop instead
* All of this could be done with Underscore.js's or lodash's `_.range()` function

## My goals
* No for loop (more functional approach)
* No library (VanillaJS)

---

```
> function sayHi() { console.log('Hello, Detroit JavaScript.'); }
> function cube(x, i) { return Math.pow(3, i); }
>
> [3, 3, 3].forEach(sayHi);
> [25, 26, 27, 28].forEach( /* do something useful with these numbers */);
> [1, 3, 9, 27, 81, 243, 729, 2187, 6561, 19683].forEach( /* */ );
```

#### Aside

Array constructor behaves the same whether or not you invoke it with new, so `new Array(10)` and `Array(10)` produce the same results.

## Why doesn't the naive solution for programmatically filling work?

So let's try `map`ping over an array created with the array constructor.

```
> var arr = new Array(10).map(function(x, i) {
  return Math.pow(3, i);
});
> arr;
[undefined × 10]
```

Does `new Array(10)` give us a real array? -Yes.

```
> Array.isArray(arr);
true
> Object.prototype.toString.call(arr);
"[object Array]"
> arr.join();
",,,,,,,,,"
> console.dir(arr);
```

* Array[10]
  * length: 10

Contrast that to:

```
> console.dir([0, 1, 2, 3, 4, 5, 6, 7, 8, 9]);  
```

* Array[10]
  * length: 10
  * 0: 0
  * 1: 1
  * <and so forth>

Contrast that to:

```
> var arr2 = [,,,,,,,,,,];
> arr2;
[undefined × 10]
> console.dir(arr2);
```

* Array[10]
  * length: 10

## JavaScript arrays

* JavaScript arrays can have "holes".
* These holes are effectively missing indexes (think of a JavaScript array as more like a map/dictionary).
* Trying to access a hole results in `undefined`. An array element can also have the value `undefined`. It can be hard to tell the difference.
* You can create holes by assignment or by setting the length of an existing array.
* You almost never want holes.
* The Array constructor creates an array with the specified length, but with holes instead of indexes.
* `Array.prototyp.forEach` and `Array.prototype.map` skip holes.
* Using `Function.prototype.apply` on the Array constructor turns each hole into an index whose value is `undefined`.
* The only time I've found any of this array hole stuff to be useful is programmatically filling an array with values.

## Solution

So don't just *invoke* the Array constructor, *apply* the Array constructor with an invocation of the Array constructor as the second argument to turn the holes into elements with the value `undefined`:

```
> var arr = Array.apply(null, new Array(10));
> arr;
[undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined, undefined]
> console.dir(arr);
```

* Array[10]
  * length: 10
  * 0: undefined
  * 1: undefined
  * <and so forth>

Finally, the solution with `map`:

```
> var arr = Array.apply(null, new Array(10)).map(function(x, i) {
    return Math.pow(3, i);
  });
> arr;
[1, 3, 9, 27, 81, 243, 729, 2187, 6561, 19683]
> console.dir(arr);
```

* Array[10]
  * length: 10
  * 0: 1
  * 1: 3
  * 2: 9
  * <and so forth>

Alternately, use an array-like object:

```
> var arr = Array.apply(null, {length: 10}).map(function(x, i) {
    return Math.pow(3, i);
  });
> arr;
[1, 3, 9, 27, 81, 243, 729, 2187, 6561, 19683]
```

## ES2015 offers other additional solutions

* `Array.from` can convert an Iterable or array-like object into an array, taking a mapping function as optional second argument (more efficient than separately calling `map`)).
* Spread operator

```
Array.from(Array(10), (x, i) => Math.pow(3, i));
Array.from({length: 10}, (x, i) => Math.pow(3, i));
[...Array(10)].map((x, i) => Math.pow(3, i));
```

Not just numbers:

```
> [...Array(26)].map((x, index) => String.fromCharCode(index + 'a'.charCodeAt(0)));
```

## For grins

```
> Array.from({length:3}).forEach(sayHi);
> [...Array(3)].forEach(sayHi);
> [...Array(3).keys()].forEach(sayHi);
```

Note:

```
> var g = [...Array(3)];
> g
[undefined, undefined, undefined]
```

* Array[3]
  * length: 3
  * 0: undefined
  * 1: undefined
  * 2: undefined

```
> var g2 = [...Array(3).keys()];
> g2;
[0, 1, 2]
```

## Reference

*Speaking JavaScript* by Dr. Axel Rauschmayer, http://www.2ality.com
