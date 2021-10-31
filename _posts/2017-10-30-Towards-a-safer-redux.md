---
layout: post
title: Towards a Safer Redux
date: 2017-10-30
categories: JavaScript TypeScript ReactJS
---

Let's get right into it. From Redux's official [TodoMVC example](http://redux.js.org/docs/introduction/Examples.html#todos), here's how to take a Todo app and add an _Add Todo_ action:

```js
// containers/AddTodo.jsx

import { connect } from 'react-redux'
import { addTodo } from '../actions'

let AddTodo = connect()(({ dispatch }) =>
  ...
  <button type="submit" onClick={() => dispatch(addTodo(input.value))>
    Add Todo
  </button>
)

// actions/index.js

export let addTodo = (text) => ({
  type: 'ADD_TODO',
  id: nextTodoId++,
  text
})

// reducers/todos.js

export let todos = (state = [], action) => {
  switch (action.type) {
    case 'ADD_TODO':
      return [
        ...state,
        {
          id: action.id,
          text: action.text,
          completed: false
        }
      ]
    ...
  }
}

// reducers/index.js

import { combineReducers } from 'redux'
import todos from './todos'

export let todoApp = combineReducers({
  todos,
  ...
})

// index.js

import reducer from './reducers'

let store = createStore(reducer)
```

Yup, you've probably seen this before. It's the Redux way: dispatch an Action in your React component (making sure that Action is defined elsewhere), define how that Action updates your State in a reducer, combine that reducer with other reducers, and register them with your store.

The idea is a beautiful one: from an initial value, your application state evolves as a sequence of Actions is folded over it. Actions are serializable and replayable, and state is a global singleton passed down your Component tree.

But do you really need all this boilerplate??

Boilerplate isn't just boring to write, but it's _unsafe_: the more of it you have, the higher the chances you'll make a mistake writing it. To add to that, [it is really hard](https://github.com/piotrwitek/react-redux-typescript-guide) to make Redux typesafe in TypeScript: you'll have to add types and sum/enum types to model your actions, and do a lot of work at the application level to make sure everything is actually safe. When adding one litte Action involves writing code across 4 files, you may find yourself reaching for a better abstraction.

### Enter [Babydux](https://github.com/bcherny/babydux).

With Babydux the above code is simplified substantially:

```js
// containers/AddTodo.jsx

import { withStore } from '../store'
import { addTodo } from '../actions'

let AddTodo = withStore()(({ store }) =>
  ...
  <button type="submit" onClick={() =>
    store.set('todos')([...store.get('todos'), {
      id: nextTodoId++,
      text: input.value,
      completed: false
    }])
  }>
    Add Todo
  </button>
)

// store.js

let store = createStore({
  todos: []
})

export let withStore = connect(store)
```

If you're using TypeScript, all you need to manually type is your store - no typing Actions, Action enums or higher order components:

```ts
// store.ts

type Store = {
  todos: {
    id: number
    text: string
    completed: boolean
  }[]
}

let store = createStore<Store>({
  todos: [],
})
```

Here's how Babydux works:

Based on the initial values for your Store, Babydux creates an Action and a Reducer for each key on your store. Actions share their corresponding key's name (in our Todos example, Babydux will create one Action: `todos`).

To read a value from the store, you use a familiar getter API: `store.get('todos')`. Babydux tells TypeScript that the return type of this expression is a list of Todos.

To write to the store, you use a familiar setter API (curried for convenience): `store.set('todos')([])`. Babydux tells TypeScript that the second argument is a list of Todos. If you call `set` with an invalid property name or if the value you give it is of the wrong type, TypeScript will give an error at compile time.

Under the hood, writing to the store (1) dispatches an Action on the store, (2) that Action fires Babydux's corresponding built-in reducer to update the store, and (3) any subscribers are notified of the change. It looks like a regular setter, buit it's really an Action emitter under the hood.

Unlike Redux's `subscribe`, which acts on a store and accepts a vanilla callback, Babydux subscribers are full-featured Rx observables. In Babydux, we call these subscribers **Effects**. Effects give you fine-grained control over how to react to a property on your store updating. Redux lets you do this in an ad-hoc way, but Babydux makes it incredibly easy by structuring Actions as explicitly reactive streams of events; while Redux just fires your `subcribe` function from time to time, Babydux gives `subscribe` semantics. For example, say we want to save our Todos to the server anytime they're updated. We can do it like this:

```ts
store
  .on('todos') // When todos are changed,
  .debounce(1000) // Save at most once per second
  .subcribe((todos) =>
    fetch('/todos', {
      method: 'POST', // POST to the /todos route,
      body: JSON.stringify(todos), // Sending the most recent Todos
    })
  )
```

With Redux you would put a network request like this in a Reducer, or in Middleware. With Babydux, you put it in an Effect.

Because Actions map 1-to-1 to reducers, which map 1-to-1 to Store properties, TODO
