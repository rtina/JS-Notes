# JavaScript Scope and Closures: A Practical Guide

---

## 1. What Is Scope?

### Overview
**Scope** defines where variables and functions are accessible in your code.
Think of scope as visibility boundaries created by code structure.

### Main Types of Scope
- **Global Scope**: Accessible from anywhere in the program.
- **Function Scope**: Variables declared inside a function are only available within that function.
- **Block Scope**: Variables declared with `let` and `const` are limited to `{ ... }` blocks.
- **Lexical Scope**: Scope is determined by where code is written, not where functions are called.

---

## 2. Global, Function, and Block Scope

### Example
```js
const appName = "ScopeLab"; // Global scope

function greetUser(user) {
  const message = `Welcome to ${appName}`; // Function scope

  if (user.isAdmin) {
    let badge = "ADMIN"; // Block scope
    console.log(message, badge);
  }

  // console.log(badge); // ReferenceError: badge is not defined
}

greetUser({ isAdmin: true });
```

### Detailed Walkthrough
1. `appName` is declared in global scope, so every function can read it.
2. `greetUser` is called with `{ isAdmin: true }`. A new function scope is created.
3. Inside `greetUser`, `message` is created in function scope and can be used anywhere inside that function.
4. The `if` block runs because `user.isAdmin` is `true`.
5. `badge` is declared with `let`, so it exists only inside the `if` block.
6. `console.log(message, badge)` works because both variables are visible at that line.
7. Outside the block, `badge` no longer exists. Accessing it causes `ReferenceError`.

### Key Takeaways
- `var` is function-scoped.
- `let` and `const` are block-scoped.
- Accessing a variable outside its scope throws `ReferenceError`.

---

## 3. Lexical Scope (Core Rule Behind Closures)

Lexical scoping (also known as static scoping) is a convention where a variable's scope is determined by its physical location in the source code. 
In JavaScript, this means that inner functions have access to variables declared in their outer (parent) scopes. This relationship is "static" because it is set when you write the code, not when the code runs.

A function can access:
- its own variables,
- variables from outer functions,
- global variables.

It **cannot** access variables from inner functions.

```js
const topic = "JavaScript";

function outer() {
  const section = "Scope";

  function inner() {
    const note = "Lexical";
    console.log(topic, section, note); // Works
  }

  inner();
  // console.log(note); // ReferenceError
}

outer();
```

### Detailed Walkthrough
1. `topic` is global, so it is visible to both `outer` and `inner`.
2. Calling `outer()` creates a new scope where `section` exists.
3. `inner` is defined inside `outer`, so it is lexically connected to `outer`'s scope.
4. When `inner()` runs, JavaScript resolves identifiers in this order:
   - Check `inner` scope.
   - If not found, check `outer` scope.
   - If still not found, check global scope.
5. `note` is found in `inner`, `section` in `outer`, and `topic` in global scope.
6. `note` cannot be accessed from `outer` because parent scopes cannot read child-local variables.

---

## 4. What Is a Closure?

### Definition
A **closure** is created when a function remembers variables from its lexical scope even after the outer function has finished executing.

### Example
```js
function createCounter() {
  let count = 0;

  return function increment() {
    count += 1;
    return count;
  };
}

const counterA = createCounter();
console.log(counterA()); // 1
console.log(counterA()); // 2
console.log(counterA()); // 3
```

`increment` keeps access to `count` through closure, even though `createCounter` has already returned.

### Detailed Walkthrough
1. `createCounter()` is called and creates local variable `count = 0`.
2. It returns `increment`, but `increment` still references `count`.
3. `counterA` now stores that returned function.
4. First call `counterA()`:
   - `count` becomes `1`
   - returns `1`
5. Second call uses the same preserved `count`:
   - `count` becomes `2`
   - returns `2`
6. Third call:
   - `count` becomes `3`
   - returns `3`
7. The key point: closure preserves state between calls without using global variables.

---

## 5. Why Closures Matter

Closures are useful for:
- **Data privacy** (private state)
- **Function factories**
- **Memoization/caching**
- **Callbacks and async behavior**

### Function Factory Example
```js
function makeMultiplier(factor) {
  return function (value) {
    return value * factor;
  };
}

const double = makeMultiplier(2);
const triple = makeMultiplier(3);

console.log(double(5)); // 10
console.log(triple(5)); // 15
```

### Detailed Walkthrough
1. `makeMultiplier(2)` creates a new scope where `factor = 2`, then returns an inner function.
2. That returned function is assigned to `double` and closes over `factor = 2`.
3. `makeMultiplier(3)` does the same independently, creating `triple` with `factor = 3`.
4. `double(5)` computes `5 * 2` because its closure remembers `2`.
5. `triple(5)` computes `5 * 3` because its closure remembers `3`.
6. Each returned function has its own private captured environment.

---

## 6. Common Closure Pitfall (`var` in Loops)

```js
for (var i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 4, 4, 4
```

All callbacks share the same `i` because `var` is function-scoped.

### Detailed Walkthrough (`var`)
1. `var i` creates one shared binding for the entire loop.
2. The loop schedules 3 timers quickly, then continues immediately.
3. Before timers execute, the loop ends and `i` becomes `4`.
4. Each callback later reads the same shared `i`, which is now `4`.
5. Result: `4, 4, 4`.

### Fix with `let`
```js
for (let i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// Output: 1, 2, 3
```

`let` creates a new block-scoped binding each iteration.

### Detailed Walkthrough (`let`)
1. With `let`, each iteration gets a fresh `i` binding.
2. First iteration closure captures `i = 1`.
3. Second captures `i = 2`.
4. Third captures `i = 3`.
5. When timers run, each callback reads its own captured value.
6. Result: `1, 2, 3`.

---

## 7. Interview-Level Summary

- Scope controls variable visibility.
- JavaScript uses lexical scoping.
- Closures happen when inner functions retain access to outer variables.
- Closures enable encapsulation and powerful abstractions.
- Prefer `let`/`const` over `var` to avoid scope bugs.

---

## 8. Quick Practice Prompts

1. Write a function `createBankAccount(initialBalance)` with `deposit` and `withdraw` methods using closure.
2. Build a `once(fn)` utility that allows a function to run only once.
3. Explain why closures are essential in event listeners and async callbacks.
