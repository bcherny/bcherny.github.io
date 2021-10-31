---
layout: post
title: 'React+TypeScript: Use unions of objects for props'
date: 2019-12-24
categories: TypeScript React
---

When typing React component `props` using TypeScript, I often see code that makes use of optional and nullable fields:

```tsx
type Props = {
  href?: string
  onClick?: () => void
  text: string
}

function Button(props: Props) {
  if ('href' in props) {
    return (
      <a className="Button" href={props.href}>
        {props.text}
      </a>
    )
  }
  if ('onClick' in props) {
    return (
      <button className="Button" onClick={props.onClick}>
        {props.text}
      </button>
    )
  }
  throw ReferenceError('You must pass either href or onClick to <Button />')
}

let a = <Button text="Click me" href="https://github.com" />
let b = <Button text="Click me" onClick={() => {}} />

// OK. Should be an error
let c = <Button text="Click me" href="https://github.com" onClick={() => {}} />

// OK. Throws an error at runtime
let d = <Button text="Click me" />
```

In this example, `Button` is a React component that takes a prop `text`, and either an `href` or `onClick` prop controlling what happens when you press the button. Either `href` or `onClick` might be defined, and we throw a runtime exception if neither of them are defined.

This code has a few problems:

1. The error case (when neither `href` nor `onClick` are passed in) is caught at runtime. Can we catch it at compile time instead?
2. As a person using `<Button />`, I don't know if I should pass in `href`, `onClick`, or both.
3. We only handled 2 cases in our component's implementation, but there are actually 4 cases to handle!

| Was `href` passed in? | Was `onClick` passed in? | Did we handle it? |
| --------------------- | ------------------------ | ----------------- |
| No                    | No                       | No                |
| Yes                   | No                       | Yes               |
| No                    | Yes                      | Yes               |
| Yes                   | Yes                      | No                |

When you have an object type with nullable, optional, or union-typed fields, the number of states the object can be in quickly explodes. This example has two optional fields, leading to 2<sup>2</sup>=4 states. If it had 3 optional fields that would be 2<sup>3</sup>=8 states, 4 would be 2<sup>4</sup>=16 states, and so on. The number of states can grow fast, slowing down typechecking, introducing subtle bugs in your code, and making your APIs a little bit harder to reason about.

When possible, use unions of objects instead of objects with nullable fields.

Since TypeScript unions are not disjoint by default, you'll want to also add a special "tag" field on each object in the union: a field that's typed as a literal (string, number, boolean, etc.) that hints to TypeScript that the union is disjoint -- that is, that `props` must be the first object in the union, or the second, but not an object that has fields from both. (Concretely, if you don't add the field, TypeScript won't complain about `c` below):

```tsx
type Props =
  | {text: string; type: 'href'; href: string}
  | {text: string; type: 'onClick'; onClick(): void}

function Button(props: Props) {
  if (props.type === 'href') {
    return (
      <a className="Button" href={props.href}>
        {props.text}
      </a>
    )
  }
  return (
    <button className="Button" onClick={props.onClick}>
      {props.text}
    </button>
  )
}

let a = <Button text="Click me" type="href" href="https://github.com" />
let b = <Button text="Click me" type="onClick" onClick={() => {}} />

// Compile-time error
let c = (
  <Button
    text="Click me"
    type="href"
    href="https://github.com"
    onClick={() => {}}
  />
)

// Compile-time error
let d = <Button text="Click me" />
```

This approach enforces at compile time that either `href` or `onClick`, and not both, are passed as props to `<Button />`. That means we don't need to throw runtime exceptions anymore, and we can trust TypeScript to verify that we've covered our cases.

Note that you can't always use this approach. There are times when it's impossible or needlessly wordy to split out your object of unions into a union of objects. Keep this technique in mind for the times that it does make your code more terse, safe, and easy to use.
