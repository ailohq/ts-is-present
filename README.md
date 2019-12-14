# ts-is-present

The `ts-is-present` package provides common functions to let you filter out the `null` or `undefined`
values from arrays in your code AND end up with the types that you expect.

## Short explanation

The following code feels like it should type check, but it does not:

![Failing code](https://i.imgur.com/d8EBtg6.png)

It fails because the TypeScript type checker can't intuit that the lambda function eliminates the undefined values:

![Reasons for failing code](https://i.imgur.com/32biELe.png)

This library provides the three `isPresent`, `isDefined` and `isFilled` functions to solve this issue in the way that you would
expect the `filter` function to work:

![Working code](https://i.imgur.com/WqgHTrU.png)

Use this library to dramatically simplify your TypeScript code and get the full power of your types.

## Deeper Explanation

An example of the fundamental problem can be [found in the TypeScript bug tracker](https://github.com/microsoft/TypeScript/issues/16069) 
but we will try and explain it again simply here.

Firstly, TypeScript can not look at the following 
lambda function `x => x !== undefined` and derive the type `(t: T | undefined): t is T`. 
Instead, the best it manage is to derive the type: `(t: any): boolean`.

Secondly, TypeScript has two type definitions for the `filter` function. They are:

``` typescript
// Definition 1
filter<S extends T>(callbackfn: (value: T, index: number, array: T[]) => value is S, thisArg?: any): S[];
    
// Definition 2
filter(callbackfn: (value: T, index: number, array: T[]) => unknown, thisArg?: any): T[];
```

If we look at those types carefully they differ in an interesting way. 

The second definition expects a callback function where the return type of that callback is `unknown`; 
this will be treated as a truthy value when the filtering is performed. Most importantly, in this 
function, if you give it an `Array<T>` then you will get back an `Array<S>`; even if the lambda 
that you provided "proves" that the type could be restricted further.

The first definition, however, expects that the return type of the callback will be `value is S` 
where the generic definition of `S extends T` applies. This means that, if you give this version of 
filter an `Array<T>` and a function that can tell if a particular `T` is actually of the more restrictive 
type `S` then it will give you back an `Array<S>`. This is the critical feature of the `filter` type definitions
that lets the functions defined in this library refine the types inside a filter.

In short, when you write the following code the second `filter` definition is used:

``` typescript
results.filter(x => x !== undefined)
```

However, when you use this library the first `filter` definition is used:

``` typescript
results.filter(isDefined)
```

That is why this library helps you derive the types you expect.

