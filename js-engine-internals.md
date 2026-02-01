# JavaScript Engine Internals: Execution Context, Hoisting, TDZ, and Call Stack

---

## 1. Execution Context (Global vs. Function)

### Overview
An **Execution Context** is the environment in which JavaScript code is evaluated and executed. There are two main types:
- **Global Execution Context**: Created when the script first runs. There is only one per program.
- **Function Execution Context**: Created every time a function is invoked. Each has its own scope and variables.

### Key Takeaways
- Every execution context has a variable environment, scope chain, and `this` binding.
- Contexts are stacked (LIFO) as functions are called and returned.
- Understanding context is crucial for debugging scope and closure issues.

---

### Code Laboratory
```js
var a = 10;
function foo() {
  var b = 20;
  function bar() {
    var c = 30;
    return a + b + c;
  }
  return bar();
}
foo();
```


#### Graphical Representation: Execution Context Stack

When the code runs, the JavaScript engine creates and manages a stack of execution contexts. Each context represents an environment for code execution.

```
|----------------------|
|  bar() Context       |  <-- Top of stack (currently executing)
|----------------------|
|  foo() Context       |
|----------------------|
|  Global Context      |
|----------------------|
```

**Step-by-step:**
1. The script starts: Global Context is created (variables, functions declared).
2. `foo()` is called: a new Function Execution Context for `foo` is pushed onto the stack.
3. Inside `foo`, `bar()` is called: another Function Execution Context for `bar` is pushed.
4. When `bar` returns, its context is popped off. Then `foo` returns, and finally only the Global Context remains.

**In-Depth Explanation:**
- **Global Execution Context**: Created once, contains global variables and functions. Sets up the base environment.
- **Function Execution Context**: Created each time a function is invoked. Has its own variable environment, scope chain, and `this` value.
- **Stacking**: Contexts are managed in a stack (LIFO). The engine always executes the context on top.
- **Scope Chain**: Each context has access to its own variables and those of its parent contexts (lexical scoping).

**Debugging Tip:**
If you see a `ReferenceError`, check which context is active and what variables are available in its scope chain.


#### Code Walkthrough: Execution

1. **Global Context**
  - `a` is declared and initialized to 10.
  - `foo` is declared as a function.
2. **foo() is called**
  - New context for `foo` is created.
  - `b` is declared and initialized to 20.
  - `bar` is declared as a function.
3. **bar() is called inside foo**
  - New context for `bar` is created.
  - `c` is declared and initialized to 30.
  - Returns `a + b + c` (uses scope chain to access all variables).
4. **Return values propagate back down the stack.**

---

## 2. Hoisting (Variable vs. Function)

### Overview
**Hoisting** is JavaScript's behavior of moving declarations to the top of their containing scope during the creation phase. hoisting is the interpreter's behavior of moving declarations to the top of their containing scope before code execution. This allows you to use certain variables and functions before they appear to be defined in your code.
- **Function Declarations** are hoisted with their definitions.
- **Variable Declarations** (`var`) are hoisted but initialized as `undefined`.
- `let` and `const` are hoisted but not initialized (see TDZ).

### Key Takeaways
- Only declarations are hoisted, not initializations.
- Function expressions and arrow functions assigned to variables are not hoisted as functions.
- Hoisting can lead to subtle bugs if misunderstood.

---

### Code Laboratory
```js
console.log(foo); // ?
var foo = 42;
bar(); // ?
function bar() {
  console.log('bar called');
}
```


#### Graphical Representation: Hoisting

Before any code runs, the engine scans for declarations and "hoists" them to the top of their scope.

```
// Creation Phase (before execution):
Scope:
----------------------
foo: undefined
bar: function bar() {...}
----------------------
```

**In-Depth Explanation:**
- **Function Declarations**: Hoisted with their full definition. You can call them before their appearance in code.
- **Variable Declarations (`var`)**: Hoisted but initialized as `undefined`. Only the declaration is hoisted, not the assignment.
- **`let` and `const`**: Hoisted but not initialized (see TDZ below).

**Debugging Tip:**
If you get `undefined` when logging a variable before its declaration, it's because of hoisting. If you get a `ReferenceError`, it's likely a `let`/`const` in the TDZ.


#### Code Walkthrough: Hoisting in Action

1. `console.log(foo);` logs `undefined` because `foo` is hoisted but not yet assigned.
2. `var foo = 42;` assigns 42 to `foo`.
3. `bar();` works because `bar` is hoisted as a function.
4. Function expressions (e.g., `var baz = function() {}`) are not hoisted as functions, only as variables.

---

## 3. The Temporal Dead Zone (TDZ)

### Overview
The **Temporal Dead Zone** is the period between entering scope and variable declaration where `let` and `const` variables exist but are not initialized. Accessing them throws a `ReferenceError`.

### Key Takeaways
- TDZ applies only to `let` and `const`.
- Variables are hoisted but not initialized until their declaration is evaluated.
- Accessing variables in the TDZ is a common source of runtime errors.

---

### Code Laboratory
```js
console.log(a); // ?
let a = 5;
```


#### Graphical Representation: Temporal Dead Zone (TDZ)

When using `let` or `const`, the variable is hoisted but not initialized. Accessing it before the declaration results in a `ReferenceError`.

```
// Creation Phase:
Scope:
----------------------
a: <uninitialized> (TDZ)
----------------------

// Execution Phase (before declaration):
console.log(a); // ReferenceError!
```

**In-Depth Explanation:**
- **TDZ**: The period between entering scope and the actual declaration. The variable exists but cannot be accessed.
- **Why?**: This prevents bugs from using variables before they're ready.
- **`let`/`const` vs `var`**: `var` is initialized to `undefined` immediately, but `let`/`const` are not.

**Debugging Tip:**
If you see a `ReferenceError` for a variable you know is declared, check if you're accessing it before its declaration (TDZ).


#### Code Walkthrough: TDZ in Action

1. The engine hoists `a` but does not initialize it.
2. `console.log(a);` throws a `ReferenceError` because `a` is in the TDZ.
3. After `let a = 5;`, `a` is initialized and can be used.

---

## 4. The Call Stack

### Overview
The **Call Stack** is a stack data structure that records where in the program we are. Each function call creates a new stack frame. When a function returns, its frame is popped off.

### Key Takeaways
- The call stack is LIFO (Last-In, First-Out).
- Stack overflow occurs when the stack grows beyond its limit (e.g., infinite recursion).
- Understanding the call stack is essential for debugging synchronous and async code.

---

### Code Laboratory
```js
function a() {
  b();
}
function b() {
  c();
}
function c() {
  console.log('In c');
}
a();
```


#### Graphical Representation: Call Stack (Creation)

When the script loads, all function declarations are stored in the global context.

```
Global Context:
----------------------
a: function
b: function
c: function
----------------------
```

a -> b: b()
b -> c: c()
c --> b: return
b --> a: return
a --> Global: return

#### Graphical Representation: Call Stack (Execution)

As functions are called, new stack frames are pushed. When they return, frames are popped.

```
// After calling a():
|----------------------|
|  c()                 |
|----------------------|
|  b()                 |
|----------------------|
|  a()                 |
|----------------------|
|  Global Context      |
|----------------------|

// As each function returns, its frame is removed from the stack.
```

**In-Depth Explanation:**
- **Call Stack**: A LIFO stack that tracks function calls. Each call adds a new frame; each return removes one.
- **Stack Overflow**: If the stack grows too large (e.g., infinite recursion), the engine throws a stack overflow error.
- **Debugging**: Stack traces in errors show the call stack at the time of the error, helping you trace the path of execution.

**Code Walkthrough:**
1. `a()` is called: stack is [a, Global].
2. `b()` is called inside `a`: stack is [b, a, Global].
3. `c()` is called inside `b`: stack is [c, b, a, Global].
4. `c` returns, then `b`, then `a`, unwinding the stack.
