---
layout: post
title: On Derived State
date: 2018-08-10
categories: JavaScript TypeScript React
---

Derived state for frontend applications comes up a lot. State `B` depends on State `A`, so after you update `A`, by the time you read `B` you want it to be updated based on the latest value of `A`. There's a bunch of ways to model this, involving different tradeoffs:

- Do you treat derived data and non-derived data the same?
- Should users interact with the two the same way?
- Do you re-compute dependent data in a lazy or eager way?
- How much control do you need over re-computing derived data?

A popular solution is selector libraries like [Reselect](https://github.com/reduxjs/reselect). The way they work is:

1. `A` changes
2. Define a function ("selector") `S` that takes State `A` and returns State `B`
3. When you want to read `B`, you call `S` with `A`

To make this pattern work efficiently, `S` of course has to be memoized, and State `A` immutable for that memoization to work. This approach makes the following tradeoffs:

- You treat derived and non-derived data fundamentally differently
- Users interact with the two differently
- You re-compute derived data in a lazy way
- You don't have control over how derived data is re-computed

**[Undux](http://undux.org/) makes a different set of choices when it comes to derived data.**

With Undux, you treat derived data the same as non-derived data, both from the store's point of view and your user's point of view. That's because from your Component's point of view, the two are the same. In fact, if your component (or its container) knows what's derived and what isn't, that might be a sign of poor encapsulation.

To derive data in Undux, you just compute it with an effect. Optionally, prefix its key with something:

```js
store
  .on('networkData')
  .pipe(map(postProcessData))
  .subscribe(store.set('derived/processedNetworkData'))
```

This approach makes different tradeoffs than selector libraries do:

- You treat derived and non-derived data the same
- Users interact with the two the same (less a namespace like `derived/`, if you want)
- You recompute data in an eager way by default, though you can make it lazy if you want
- You have fine control over how to re-compute derived data:

```js
store
  .on('networkData')
  .pipe(
    debounceTime(200),
    filter((_) => _ !== null),
    map(postProcessData)
  )
  .subscribe(store.set('derived/processedNetworkData'))
```

Why does Undux do it this way? Isn't there something holy about a clean source of truth?

My view is there isn't. Fundamentally, all data is derived. Even the list of users you got back from a GraphQL endpoint -- that's a function of two things:

```js
users = F(initialState, API request)
```

Or, a count of how many times you pressed a button. That's derived from:

```js
count = F(initialCount, Stream(clicks))
```

That is, users are derived from an initial state and a network request. And a click count is derived from an initial count and a stream of clicks.

Selector libraries draw a line between primary state -- stuff like initial state, state from the network, state from user intreactions -- and n-ary state that's computed from that state. But isn't all state (except intial state) computed when it comes down to it? Because JavaScript doesn't have first-class monads, you don't usually think of it like that. But you should.

And when you do, you realize it makes life easier to forget about drawing lines about where state comes from, and letting components interact with all state the same way. It's a simpler mental model, it's easier to reason about, it's better enapsulation, and with Undux, is more powerful too.

What do you think?
