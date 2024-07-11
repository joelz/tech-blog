---
title: "读书笔记：OReilly.TypeScript.Cookbook.2023.8.epub"
date: 2023-11-03T12:01:58+08:00
description: "类型体操练起来"
tags: [book, typescript]
featured_image: "/tech-blog/images/notebook.jpg"
categories: book
comment : false
---

## 4.8 Using ThisType to Define this in Objects

**Problem:** Your app requires complex configuration objects with methods, where this has a different context depending on usage.

**Solution:** Use the built-in generic `ThisType<T>` to define the correct this.

```typescript
// An object of functions ...
type FnObj = Record<string, () => any>;
// ... to an object of return types
type MapFnToProp<FunctionObj extends FnObj> = {
  [K in keyof FunctionObj]: ReturnType<FunctionObj[K]>;
};

type Options<Data, Computed extends FnObj, Methods> = {
  data(this: {})?: Data;
  computed?: Computed & ThisType<Data>;
  methods?: Methods & ThisType<Data & MapFnToProp<Computed> & Methods>;
};

declare function create<Data, Computed extends FnObj, Methods>(
  options: Options<Data, Computed, Methods>
): any;

// use the types
const instance = create({
  data() {
    return {
      firstName: "Stefan",
      lastName: "Baumgartner",
    };
  },
  computed: {
    fullName() {
      // has access to the return object of data
      return this.firstName + " " + this.lastName;
    },
  },
  methods: {
    hi() {
      // use computed properties just like normal properties
      alert(this.fullName.toLowerCase());
    },
  },
});
```

## 6.5 Dealing with Recursion Limits

We want to use TypeScript’s feature to recursively call conditional types to create a valid string identifier out of any string, by removing whitespace and invalid 
characters.

```typescript
type RemoveWhiteSpace<T extends string> = T extends `${infer A} ${infer B}`
  ? RemoveWhiteSpace<`${Uncapitalize<A>}${Capitalize<B>}`>
  : T;  
type Identifier = RemoveWhiteSpace<"Hello World!">; // "helloWorld!”

type StringSplit<T extends string> = T extends `${infer Char}${infer Rest}`
  ? Capitalize<Char> | Uncapitalize<Char> | StringSplit<Rest>
  : never;
type Chars = StringSplit<"abcdefghijklmnopqrstuvwxyz">;
//  "a" | "A" | "b" | "B" | "c" | "C" | "d" | "D" | "e" | "E" |
//  "f" | "F" | "g" | "G" | "h" | "H" | "i" | ...

type CreateIdentifier<T extends string> =
  RemoveWhiteSpace<T> extends `${infer A extends Chars}${infer Rest}`
  ? `${A}${CreateIdentifier<Rest>}`
//  ^ Type instantiation is excessively deep and possibly infinite.(2589)_.
  : RemoveWhiteSpace<T> extends `${infer A}${infer Rest}`
  ? CreateIdentifier<Rest>
  : T;
  
type CreateIdentifier<T extends string, Acc extends string = ""> =
  RemoveWhiteSpace<T> extends `${infer A extends Chars}${infer Rest}`
  ? CreateIdentifier<Rest, `${Acc}${A}`>
  : RemoveWhiteSpace<T> extends `${infer A}${infer Rest}`
  ? CreateIdentifier<Rest, Acc>
  : Acc;
  
type Identifier = CreateIdentifier<"Hello Wor!ld!">; // "helloWorld"
```

## 7.2 Typing a promisify Function
**Problem:** You want to convert callback-style functions to Promises and have them perfectly typed.

**Solution:** Function arguments are tuple types. Make them generic using variadic tuple types.

```typescript
function promisify<Args extends unknown[], Res>(
  fn: (...args: [...Args, (result: Res) => void]) => void
): (...args: Args) => Promise<Res> {
  return function (...args: Args) { 
    return new Promise((resolve) => { 
      function callback(res: Res) { 
        resolve(res);
      }
      fn.call(null, ...[...args, callback]); 
    });
  };
}
```

## 8.3 Remapping Types

**Problem:** Constructing types gives you flexible, self-maintaining types, but the editor hints leave a lot to be desired.

**Solution:** Use the Remap<T> and DeepRemap<T> helper types to improve editor hints.

```typescript
type Remap<T> = {
  [K in keyof T]: T[K];
};

type DeepRemap<T> = T extends object
? {
    [K in keyof T]: DeepRemap<T[K]>;
  }
: T;
```

## 12.7 Deciding Whether to Use Function Overloads or Conditional Types

Function overloads provide better readability and an easier way to define expectations from your type than conditionals. Use them when the situation requires.

```typescript
// Function overloads
function fetchOrder(customer: Customer): Order[]
function fetchOrder(product: Product): Order[]
function fetchOrder(orderId: number): Order
function fetchOrder(param: Customer | Product): Order[]
function fetchOrder(param: Customer | number): Order | Order[]
function fetchOrder(param: number | Product): Order | Order[]
// the implementation
function fetchOrder(param: any): Order | Order[] {
  //...
}

// Here, conditional types can reduce your function signature tremendously:
type FetchParams = number | Customer | Product;
type FetchReturn<T> = T extends Customer
  ? Order[]
  : T extends Product
  ? Order[]
  : T extends number
  ? Order
  : never;

function fetchOrder<T extends FetchParams>(params: T): FetchReturn<T> {
  //...
}
```

One scenario where function overloads remain handy is if you have different argument lists for your function variants. This means not only the arguments (parameters) themselves can have some variety (this is where conditionals and variadic tuples are fantastic) but also the number and position of arguments.

```typescript
function search(query: string): Promise<unknown[]>;
function search(query: string, callback: (result: unknown[]) => void): void;
// This is the implementation, it only concerns you
function search(
  query: string,
  callback?: (result: unknown[]) => void
): void | Promise<unknown> {
  // Implement
}
```

Another situation where function overloads can make things easier is when you need exact arguments and their mapping.”

