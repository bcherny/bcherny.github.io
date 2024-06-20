---
layout: post
title: 'NPM and NodeJS should do more to make ES Modules easy to use'
date: 2024-06-19
categories: JavaScript, TypeScript
---

Coming back to JavaScript and TypeScript after a few years neck deep in Python and Hack, I kept hitting a number of new, cryptic errors when running NodeJS code in my dev environment:

```sh
# when I ran ESM TypeScript code the wrong way:
Error [ERR_REQUIRE_ESM]: Must use import to load ES Module

# when I imported an ESModule from a CommonJS .js file:
Error [ERR_REQUIRE_ESM]: require() of ES Module .../lodash.js from .../index.cjs not supported

# when I imported an ESModule from a .ts file:
error TS1479: The current file is a CommonJS module whose imports will produce 'require' calls

# when I used ES6 import syntax in a .js file:
SyntaxError: Cannot use import statement outside a module
```

These errors are all related to importing, typechecking, and loading modules. The JavaScript ecosystem moves fast, and things changing over the last few years was not a surprise. However, it was surprising to see so many errors related to such a core piece of the language!

## How we got here

Modules in JavaScript and TypeScript have changed significantly over time:

- Years ago, there was no module system for JavaScript and TypeScript. A number of solutions sprang up around ways to declare and load modules: IIFEs, [LabJS](https://github.com/getify/LABjs), [AMD](https://github.com/amdjs/amdjs-api/blob/master/AMD.md), [require.js](https://requirejs.org/), [TypeScript namespaces](https://www.typescriptlang.org/docs/handbook/namespaces.html), and more. Tooling support and interop were hit or miss.
- CommonJS emerged as a standard-by-convention for modules, across browser, server, JavaScript, and TypeScript.
- When ES6 came out, folks started switching over to `import` and `export` syntax (from CommonJS's `require` and `module.exports`), using tools like Babel and TypeScript to compile code down to CommonJS.
- CommonJS can be challenging to statically analyze, and uses an [inefficient](https://www.youtube.com/watch?v=W5CXzo4TZVU), synchronous module loading algorithm at runtime. ES Modules were [introduced](https://tc39.es/ecma262/#sec-modules) as the way to use `import` and `export`, while at the same time improving code load times at runtime.
- ES Modules introduced significant complexity for NodeJS in particular: instead of reusing the *.js* and *.ts* file extensions, ES Modules in NodeJS require either using *.mjs*, or setting `type=module` in your *package.json*. Interoperating these modules with an ecosystem-ful of CommonJS remains painful.

## Current state, by the numbers

I was curious -- since ES Modules (`import`/`export`) were introduced in [2015](https://262.ecma-international.org/6.0/#sec-modules), and NodeJS has supported `type=module/commonjs`, *.mjs*, and *.cjs*, with the goal of replacing *.js*, since [2019](https://nodejs.org/api/packages.html#type), to what degree have these new conventions been adopted?

I answered this with data, using two approaches:

1. Looking at the most starred JavaScript and TypeScript repos on Github
2. Looking at the most downloaded packages on NPM

The [results](https://github.com/bcherny/es-module-stats) are not rosy. After 5+ years, adoption of ES Modules remains weak:

1. Between 9-27% of JavaScript/TypeScript projects declare themselves to be ES Modules via the `type` (and lesser-used `exports`) fields in their *package.json*s.
2. Less than 6% of JavaScript/TypeScript files declare that they are ES Modules via the *.mjs*, *.cjs*, *.mts*, etc. file extensions.

Note that these ranges come from the two approaches I used to estimate the numbers. Head [here](https://github.com/bcherny/es-module-stats) for more detailed data and code.

## How do we fix it?

This helps explain why it's so painful to interoperate ES Modules and CommonJS across both NodeJS and TypeScript: enough libraries use ES Modules that for many projects you need to either use ES Modules, or figure out how to interoperate ES Modules with your CommonJS code. At the same time, enough code still uses CommonJS that you often need to figure out how to include that legacy code in your otherwise-ES Module project.

The benefits of ES Modules are significant. Rolling everything back to CommonJS is not the way forward. Is there more we can do to simplify the ecosystem, and push harder on adoption? Some ideas:

1. We should kill *.mjs*, *.cjs*, *.mts*, etc. The vast majority of projects use `type=module` in their *package.json*, rather than file extensions. It would simplify things considerably if we drop support for these new file extensions and stick to *.js*, *.jsx*, *.ts*, and *.tsx*.
2. We should make `type=module` the [default](https://github.com/npm/cli/issues/7594) for new *package.json* files for the `npm init`, `yarn init`, and `pnpm init` commands. Package managers' `publish` commands should warn when `type` is not set to `module`.
3. We should upgrade the most common libraries used by the community to ES Modules, either manually or through automated pull requests (this feels like something that can be semi-automated).
4. The NPM registry can require an explicit `module` field on new packages, making it clear when a package intentionally uses CommonJS (eg. because it targets legacy NodeJS versions).
5. NodeJS can officially drop support for `require` and `module.exports` in a future version, creating a bit more pressure to migrate.

I'd love to hear others' thoughts. Have you also felt the pain of interoperating ES Modules and CommonJS?

---

Discuss this post on [HackerNews](https://news.ycombinator.com/item?id=40737508) or on [Threads](https://www.threads.net/@boris_cherny/post/C8aDJuGI5HM).
