---
layout: post
title: 'Angular 1: 8 Lessons For Designing JavaScript Frameworks'
date: 2017-07-14
categories: JavaScript
---

I've used Angular 1 at my last couple of day jobs, and have gradually grown to be grumpy about it. At the time when it was released (2010), the framework had some really cool ideas baked in: automatic Model → View syncing (unlike [Backbone](backbonejs.org)), automatic View → Model syncing (like [Knockout](knockoutjs.com)), observing arbitrary expressions, a built-in dependency injector, a neat filter syntax. All of this was pretty revolutionary; the framework took some time to learn, but once you did, you were suddenly really, _really_ productive.

In the seven years since, Angular's Big Ideas have been battle tested by thousands of engineers. Through this testing gauntlet, a lot of problems with its Big Ideas emerged and Angular 1 has since been supplanted by Angular 2. So as sort of a post mortem, I wanted to put my thoughts about where Angular 1 failed down on paper, to inform my own framework designs as well as others' designs.

In no particular order, here goes:

### 1. Dirty checking is bad

When you update a model, Angular needs to:

1. Update the model's corresponding view
2. Check if any other model fields need to be updated

When does Angular do that, and how? And what is the "model"? Well, Angular doesn't actually have a first-class notion of models, so the "model" ends up being everything on the view's `$scope`, and everything on that `$scope`'s parent scope, and so on. Angular could greedily check every field on each model every time, and repaint the entire view any time something on a model changes, but in practice this is way too slow. So some optimization is necessary.

Dirty checking is the process of looping over the value of every property on every `$scope` object that could have changed, comparing it to its previous value, and if it changed, marking it as "dirty" (meaning update it anywhere it appears in the view, and check any other model fields that might need to update in response to it changing). This means (1) Angular doubles the amount of memory you use to store data on the `$scope`, and (2) every time a value on the `$scope` changes it takes _O(number of scopes _ number of properties/scope)\* time to reflect that change in the view. As your app grows and has more scopes, each update takes longer and longer.

By contrast, Backbone and React use setters to update their models. By having to call a special function every time the model changes, Backbone and React get to know exactly which property changed, and what its new value is. This makes their update mechanism way faster than dirty checking, and more importantly, their mechanism runs in constant _O(1)_ time for each changed property. Dirty checking is always slower than setters.

That's not the end of it. Dirty checking makes it hard to interoperate Angular and non-Angular code. Angular hooks into every event that has the potential to update the model (`$http` requests, `click` events, etc.), so it knows to run a dirty check after the event is done. Outside libraries (eg. your favorite jQuery-based calendar widget) don't use these same hooked functions, so you have to manually tell Angular that something might have changed by calling `$scope.$apply` yourself (unless that update is happening within a `$scope.$apply` already, in which case Angular will throw an error).

### 2. Watchers are hard to debug and trace

If you use `$watch`ers a lot, you'll often run into something like this:

1. Watcher A fires
2. Watcher B fires
3. Watcher C fires
4. Watcher B fires again
5. Watcher C fires again

You might the scratch your head and think "why did B and C fire twice?". With Angular's `$watch` mechanism the best you can do is to add a `console.log(oldValue, newValue)` at the top of every `$watch`er's callback (or, set breakpoints), open up Devtools, and see what the values were that caused your `$watch`ers to fire. Along with your knowledge of what expression each `$watch`er observes, this is usually enough to piece together what happened. Note that you can't get a proper [stack trace](https://github.com/angular/angular.js/issues/12606) for the change, meaning Angular can't tell you _why_ something changed, just that it _did_ change.

This sort of feedback loop built out of simple "if this then that" rules is at the heart of complex and chaotic systems. This is the last thing you want in a piece of software, because it makes behavior unpredictable and impossible to totally specify. Writing Angular code with lots of `$watch`ers (or the equivalent `Object.observe`, before that API was retracted) makes the debugging experience akin to debugging a distributed system, or maybe a neural network. Potentially never-ending cascades of updating values are something to be avoided at all costs; they make systems unpredictable, unspecifiable, and really hard to debug.

Because `$watch`ers might update something on the scope in response to something else changing, the dirty check loop (#1 above) might have to be repeated a few times, in case values on the `$scope` continue to change in response to a `$watch`er. A common issue people run into is the `$digest() iterations reached` error message, which means the dirty checking process had to run more than 5 times (or whatever your TTL is set to). This is Angular compassionately limiting how many times your feedback loop can run.

### 3. Dependency Injection makes it hard to interoperate Angular and non-Angular code

Angular shipped with a pretty cool idea: a built-in Dependency Injector (DI) inspired by [Guice](https://github.com/google/guice)'s success as a DI for Java. The idea is instead of instantiating services yourself, you just provide a Class and the framework takes care of instantiating the Class and providing instances of it to whatever services needs one.

The problem is, what happens if you want to interoperate Angular and non-Angular code?

Your Angular code will only have access to your non-Angular code if you explicitly expose it to the DI. And your non-Angular code will only have access to your Angular code if you explicitly export it. If your Angular code depends on built-in Angular services (say, `$http`), then you're out of luck. There are tools like [ngimport](https://github.com/bcherny/ngimport) to help mitigate this, but on its own Angular tends to make interop really hard. Compared with React or Vue where interop is free, this can be a big pill to swallow.

In practice, interoperability of complementary tools is really, really important. For programmers, these situations happen all the time:

- You need to add a big piece of functionality, and there's an existing open source solution, but it's not built for Angular.
- You're migrating from one framework to another, and you want to share services.
- You have multiple applications wirtten using different frameworks, and you want to share services.

A good framework should let yet mix and match tools, and get out of your way when you do. Of course you could rewrite your code in the One True Way for your framework of choice, but in practice that's not always feasible or desirable. This goes double for JavaScript, where frameworks, tools, and best practices change every month.

### 4. There is just one, global Dependency Injection registry per app

Each Angular app has a single `$injector`, meaning that you cannot register two services with the same name. Think about that for a second: say you have a service called `userService`; you then install a few Angular modules from NPM, add them to your app, and suddenly your app starts throwing errors. What happened? Well, one of the modules you installed also had a service called `userService`, which _overwrote_ your version of `userService`! Not only is the DI registry global, but when there are conflicts, _last one wins (silently)_.

### 5. Component APIs are fundamentally unsafe

If you're using TypeScript or Flow with your Angular app, you might expect to be able to make components type safe. Meaning if a property is misspelled or missing or of the wrong type, you'd like to see a compile time error. Say I have a calendar component that takes a `day`, `month`, and `year`, and I accidentally instantiate it as:

```js
<calendar day="10" month="'june'" yer="2017"></calendar>
```

I'd like to see two errors:

1. `The property month expects a value of type Number, but you passed "june" which is a String`
2. `The property "yer" does not exist. Did you mean "year"?`

Unfortunately, because component APIs are objects (`bindings`), and components are consumed in templates which are either strings or HTML, these sorts of errors cannot be caught at compile time without a specialized build tool (and I haven't seen one yet). With React, Vue, or Elm, these errors _are_ caught at compile time (right in your text editor), which is a huge win for the programmer that doesn't want to spend time chasing their own tail.

### 6. State is _everywhere_

Because Angular doesn't have a notion of first-class models (like Backbone or Ember) and doesn't namespace state within components (like React's `props` and `state`), state ends up being spread all around every component. Let's say this is your component:

```js
angular.component('car', {
  bindings: {
    color: '<',
  },
  controller: class {
    constructor() {
      super()
      this.position = [0, 0]
    }
    drive() {}
    park() {
      this.isParked = true
    }
  },
})
```

`this.color` is state that was passed in, `this.position` and `this.isParked` are pieces of state that are internal to the component, and `drive` and `park` are methods on the component. This is not obvious at a glance, and requires the reader to read through the whole `controller` class to see what is defined on `this`. The React way of doing things, where at a glance it's obvious what's state that was passed in and what's internal state, is a bit of convention that really improves readability and helps separate data from code.

### 7. Providers vs. Services vs. Factories

In Angular, the following are all equivalent:

```js
class Foo {..}

angular

// 1
.provider('foo', { $get: () => new Foo })

// 2
.factory('foo', () => new Foo)

// 3
.constant('foo', new Foo)

// 4
.value('foo', new Foo)

// 5
.service('foo', Foo)
```

Why are there 5 ways to do the same thing? Ergonomics, convenience, and terseness (but probably not usability).

### 8. Interoperating with TypeScript is painful

Because Angular's dependency injector uses either a closure or constructor as the injection site, when declaring a service you need to declare its interface separately, and you can't leverage TypeScript's type inference, or its ability to create a type and a value at once when declaring a function/class. For example:

```js
import {IHttpService, ILogService, IPromise} from 'angular'

angular.factory('Get', function ($http: IHttpService, $log: ILogService): Get {
  return function (url: string): IPromise<string> {
    return $http.get(url).then((data) => {
      $log.info('Got data!', data)
      return data
    })
  }
})

export interface Get {
  (url: string): IPromise<string>;
}
```

This becomes really painful when working with a lot of TypeScript code. Services and their typings get out of sync, and the code is not as safe as it would be if you let TypeScript flow the types for you.

The alternative - using a constructor function as the injection site - is even worse, because it means you can't pass arguments to instances of your class!

---

If you're thining about jumping ship from Angular 1, here's what you do:

1. Use [ngimport](https://github.com/bcherny/ngimport) so you can use regular `import`s for DI instead of Angular's special syntax. This frees you up to interoperate Angular, non-Angular, and TypeScript code with zero overhead.
2. Use [ngcomponent](https://github.com/coatue-oss/ngcomponent) for a React-style API for your component controllers.
3. If you're moving to React, use [react2angular](https://github.com/coatue-oss/react2angular) to embed your shiny new React components in your legacy Angular app.
4. If you're almost done moving to React, use [angular2react](https://github.com/coatue-oss/angular2react) to embed the remnants of your crusty Angular app in your sleek new React app.
