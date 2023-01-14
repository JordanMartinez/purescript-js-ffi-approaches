# purescript-js-ffi-approaches

A repo exploring different ways to writing FFI to JavaScript with all of its idioms.

## JavaScript idioms

There are a variety of JavaScript idioms that are inherently type-unsafe

### Normalizing function arguments

```js
const threeArgs = (mainArg, argArray, options) => {
  if (argArray && !Array.isArray(argArray)) {
    options = argArray;
    argArray = null;
  }
  if (!options) {
    options = {}
  }
  options = {
    ...defaultOptions,
    ...options
  };
}
```
The above function can be described as: `threeArgs(mainArg[, args][, options])`. It can be called in four different ways:
- `threeArgs(mainArg, args, options)`
- `threeArgs(mainArg, options)`
- `threeArgs(mainArg, args)`
- `threeArgs(mainArg)`

Does PureScript add bindings to the full 3-arg version? If it does, how does it deal with default values? Are they defined in the PureScript code or the JavaScript code?

### Multi-Type Inputs

```js
(theArg) => {
  if (theArg === undefined) return "default message";
  if (theArg === null) return "default message (2)";
  if (typeof theArg === "string") {
    if (theArg === "main") return `some message ${theArg}`;
    if (theArg === "foo") return `slightly different msg ${theArg}`;
  }
  if (typeof theArg === "object" && Array.isArray(theArg)) {
    return `some args: ${theArg.join(", ")}`;
  }
  if (typeof theArg === "object") {
    if (theArg.someField) return "field is fine";
    if (theArg.hasStuff) return "has stuff";
  }
  return "unknown value message";
}
```

If a function takes `n`-many different types of arguments, does one define `n`-many functions, where each function accepts one of the types as its argument? Or does one define one function and use some combination of `unsafeCoerce` in PureScript to implement the bindings? How does one deal with optional fields in JavaScript Objects?

### Multi-Type Outputs

#### Basic Idea

```js
(theArg) => {
  if (!theArg) return true;
  if (Array.isArray(theArg)) return "no";
  if (theArg.someField) return [0];
  if (theArg.somethingIsBad) throw new Error("Bad!");
  if (theArg.recordType1) return { a: "foo" };
  if (theArg.recordType2) return { a: "foo", bar: "bar" };
  if (theArg.recordType3) return { a: "foo", baz: "baz" };
  return effectfulThingThatThrowsUnrelatedError();
}
```

To determine what type was returned, one needs to validate things. Is that done on the PureScript side of things? Or on the JavaScript side of things?

If returned values are `Object`s, how does one handle cases where the `Object`s share some fields but differ in other fields? Wrap such values in a sum type (e.g. `Maybe`)? Consider those differences as different types?

And what about thrown `Error`s? The only way to "return" the error below is to catch the error. However, the catcher may also catch unrelated errors. Either PureScript should catch this somehow or type this as `Effect` and accept that such code can throw `Error`s.

### Subtyping via multiple classes with a shared interface

```js
class Shared {
  constructor() { this.value = true; }
  get value() { return this.value; }
  set value(x) { this.value = x; }
}

class Even extends Shared {
  constructor() { super(); }
  get even() { return this.value ? 2 : 4; }
}

class Odd extends Shared {
  constructor() { super(); }
  get odd() { return this.value ? 1 : 3; }
}
```
