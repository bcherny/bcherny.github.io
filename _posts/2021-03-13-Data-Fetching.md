---
layout: post
title: Data Management on the Web Still Sucks
date: 2021-03-13
categories: JavaScript
---

It's 2021, and data management on the web still sucks. This is crazy!

If you're building a web app at scale you have plenty of options for data fetching, transport, caching, and management:

- `fetch`, `XMLHTTPRequest`, etc. (REST)
- [Swagger](https://swagger.io/) (REST)
- [Falcor](https://github.com/Netflix/falcor) (REST)
- [Apollo](https://github.com/apollographql/apollo-client) (GraphQL)
- [Relay](https://github.com/facebook/relay) (GraphQL)
- [Protobuf](https://github.com/protobufjs/protobuf.js)
- [gRPC](https://github.com/grpc/grpc-node)
- ...

Each of these does a few things well, leaving the rest to you. What do you need to do well to be a good data fetching library?

## Fetching Data

- **Consistency when fetching data**. When your app fetches new data, the UI should automatically update everywhere that data is used.
- **Request batching**. When multiple requests are made at the same time, they should be combined into one request, when reasonable, to avoid hitting browser-imposed concurrency limits.
- **Avoiding network waterfalls**. When fetching data for a component, you should make one request that includes data for all its descendents too, to avoid lots of network round trips. GraphQL solves this with fragments, but it's still painful in modern clients (eg. Relay does this by requiring you manually compose fragments up the React tree so they're fetched together, then pass fragment pointers down the tree to tell Relay where to look for fetched data; it also requires a compilation step).
- **UX when fetching data**. It's a big burden for engineers to have to manually add loading spinners and error states, and engineers often forget.
- **Colocation**. You should declare data dependencies close to where they're used, ideally in the same file. Same principle as colocating CSS with your components.
- **File size**. You shouldn't have to ship kilobytes of metadata describing your data schema to clients in order to fetch data.
- **Type safety**. Your code should know what type of data it got from the server.
- **Versioning**. Updating your data model should be backwards-compatible; clients should be forwards-compatible.
- **Realtime updates**. Sometimes, you want data to be pushed, not pulled (eg. for things like notifications). But the APIs for pushing and pulling data are often different, both on the client and on the server.

## Updating Data

- **Consistency when updating data**. When you update a piece of data, your data fetching library should automatically refetch dependent data that will change after the request completes (eg. if you update `Person.FirstName`, `Person.FullName` should automatically be refetched).
- **Optimistic updates**. The UI should optimistically render the success state before a server response comes back, if there's a high chance the request will succeed. This makes your UI feel snappy and fun to use. Today's data fetching libraries make this a lot of manual work.
- **Request queueing**. Requests that depend on one another need to be queued; independent requests can be sent in parallel. Otherwise, network weather and server queuing can cause your client to end up in a state that's inconsistent with the server.
- **UX when updating data**. Product engineers shouldn't need to manually render loading spinners and success and error notifications.
- **Durability when updating data**. Exponential backoff for retrying idempotent requests and idempotence tokens for network failures; eager persistance to disk for client/server failures.
- **Type safety**. You shouldn't be able to send invalid input to the server, for some definition of invalid.
- **Versioning**. Updates to your mutation APIs should be backwards-compatible; clients should be forwards-compatible.

## Why?

I think the reason this hasn't been solved is it's a hard problem that's at the intersection of a bunch of fast-moving technologies:

1. Programming languages (to infer what data depends on what)
2. Databases (client-side stores are small databases)
3. Server APIs and protocols (REST, GraphQL, RPC, etc.)
4. ~~UI frameworks~~ (at this point, React has won)
5. User experience and perceived performance

So far, every attempt at solving data fetching has either tightly coupled all of these layers, leading to trouble gaining mass adoption (think [Meteor](https://www.meteor.com/)); other solutions have tackled just one layer, ignoring the rest. This smells to me like the abstractions & APIs for each layer are wrong: you shouldn't need tight coupling to solve data fetching. Instead, you need better abstractions.

Is this a problem that can be solved?
