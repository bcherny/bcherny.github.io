---
layout: post
title: On Contagion
date: 2019-09-08
categories: "Philosophy of programming"
---

If you have a tree with a node that has a property `P`, and all of its parents also need to have property `P`, then `P` is contagious.

When can that happen?

- Exceptions bubble up through the call tree
- State has to be lifted up to the root of a call tree
- If a function is `async`, its ancestors must be too
- Centralized services also centralize any decentralized services that call them
- In a physical system, if you know the position and momentum of an object `A` and it collides with another object `B` that you don't know one of those quantities for, then you no longer know them about `A`. (caveat: I'm no physicist)

Concretely:

```js
async function a() {
  // a has to be async to await b()
  await b()
}

async function b() {
  // b has to be async to await c()
  await c()
}
```

As a programmer, that seems bad. Bubbling up breaks encapsulation and makes things harder to compose. If I have a nice application and need to mark a function deep in the app `async`, why do I need to update all of its callers to be `async` too? Why should a parent know about its grandchild?

For good reason. At runtime, some contagious things are special:

- Exceptions bubble up through the call stack
- `await` pauses execution

We want to model these runtime behaviors in our language:

- Asynchronous code needs to be awaited before continuing. As a consumer of an asynchronous function, you want to know that it's asynchronous so that you can treat it as special.
- State is a function parameter, because the function's computation depends on it. As a consumer, you need to pass the state to it.
- Functions that throw are modeled with a `throws` clause, `Either`, `Err`, or nothing, depending on the language. As a consumer, you might want to know that something throws so you can handle it.

In some languages, all of these are captured in a function's return type. The idea is that these aren't implementation details -- consumers really should know about them. These languages model runtime behavior using types.

- Async: `Future`/`Promise`
- State: The `State` monad, or passing a callback down the call stack
- Exception: `Either`, `Err`, `throws`

But, you sometimes want a trap door:

- Async: `.then`, React suspense boundaries
- State: Mutable state, React local state
- Exceptions: `try`/`catch`, React error boundaries

These are ways to avoid contagion.

What unites all of those? They're effects, that different languages model differently. Some languages keep them implicit (like `throw` in languages that don't have `throws` clauses), some make them explicit (the `State` monad).

There's nothing inherently contagious about these things, as evidenced by the languages that support trap doors for them. It's up to language designers to say "this thing should be contagious" or "this thing shouldn't".

What should be contagious, but isn't? What is contagious, but shouldn't be?
