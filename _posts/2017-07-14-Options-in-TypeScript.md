---
github: https://github.com/bcherny/tsoption
layout: post
title: Options in TypeScript
date: 2017-07-14
categories: TypeScript JavaScript "Functional Programming" Scala Haskell
---

If you wrote in Scala or Haskell before you tried TypeScript, you may have found yourself wondering: Where are the Options at?

For those not familiar, [here](http://danielwestheide.com/blog/2012/12/19/the-neophytes-guide-to-scala-part-5-the-option-type.html) is an excellent introduction to Scala's `Option` type. At a high level, `Option` is an abstraction over `null` that gives useful semantics around running functions over possibly-null values. It implements a monad, a functor, and some other structures, but that's not important for this post.

I'll give a quick example of what options let you do. Let's say I start with a person, and I want to get the population for the city the person's company is in. If the person doesn't work at a company, or if the company isn't in the United States, or if population data is missing for the city the company is in, then I return `null`. Here's what my code looks like (some functions left unimplemented for clarity):

```ts
function getCompany(person: Person): Company | null {...}
function getUSCity(company: Company): string | null {...}
function getPopulation(city: string): number | null {...}

function population(person: Person): number | null {
  let company = getCompany(person)
  if (company === null) {
    return null
  }

  let city = getUSCity(company)
  if (city === null) {
    return null
  }

  return getPopulation(city)
}
```

If you're reading this, you've probably written something similar in the past (in JavaScript, TypeScript, Java, C#, C, or any other language where references can be `null`). And if you haven't used options before, you may be wondering how else you can express that code. Well, I'm here to tell you that with options, you can clean it up a lot:

```ts
function getCompany(person: Person): Option<Company> {...}
function getUSCity(company: Company): Option<string> {...}
function getPopulation(city: string): Option<number> {...}

function population(person: Person): Option<number> {
  return getCompany(person).flatMap(getUSCity).flatMap(getPopulation)
}
```

That's all there is to it. Let's zoom in on `population`, and inspect what the types are at each step:

```ts
function population(person: Person): Option<number> {
  return getCompany(person) // Option<Company>
    .flatMap(getUSCity) // Option<string>
    .flatMap(getPopulation) // Option<number>
}
```

And here's how you would consume `population` (in practice you want to avoid mapping back to `null` - that's the whole point of Options!):

```ts
let person = Person(...)
let pop = population(person).getOrElse(null)
```

TSOption's API is modelled after Scala's Options API, but is more powerful: because TypeScript supports `null` and `undefined` literal types, TSOption doesn't allow nulls to sneak in via `Some(null)` (which is allowed in Scala because in Java, everything is nullable).

If this sounds cool, check out the repo [here](https://github.com/bcherny/tsoption), or download TSOption on [NPM](https://www.npmjs.com/package/tsoption) today.
