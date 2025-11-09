---
title: "Revisiting Communication Models: REST, GraphQL, and RPC"
date: 2019-8-05
categories:
  - Service Design
tags:
  - Reflection
  - Journal
layout: single
author_profile: true
read_time: true
comments: false
share: true
---

> Curiosity · Craft · Simplicity  
> Build to wonder.

---

### Prologue

Years ago, I worked on large-scale mobile internet projects where gRPC was almost everywhere.  
At the time, the goal was clear: performance, scalability, and cross-language consistency.  

Recently, in a course, REST came up again — something I hadn’t seriously touched in a while.  
That moment made me pause. I didn’t relearn REST, it's too easy, I simply realized how much our communication needs have evolved.  
So I decided to write down some thoughts — not a tutorial, but a reflection on how systems talk.

---

### Environment and Tools

All examples were run locally — no virtual machines, no Docker.  
We will build some complicated systems later, but for now, just keep it simple.

| Tool | Purpose | Note |
|------|----------|------|
| Node.js v18+ | Runtime environment | Common base for all three implementations |
| Express.js | Web framework | For REST and RPC endpoints |
| express-graphql | GraphQL middleware | For a lightweight GraphQL service |
| Postman | Testing | Unified request/response comparison |
| VS Code | Editor | Code and notes |
| curl (optional) | Command-line testing | Minimal verification |

Data is stored in memory

---

### REST — The Language of the Web

REST is probably the most intuitive and widely used concept in modern APIs.  
It models resources, not actions. The web itself was built this way — each URI a noun, each HTTP verb a verb.

![REST Architecture Diagram](/assets/images/service_design/api_comparison/restful.png)  
*Figure: A simple REST architecture — data → server → resource (URI) → Client response.*

**Key characteristics:**

- Resource-oriented: `/book/123`  
- Uses standard HTTP methods (GET, POST, PUT, DELETE)  
- Strong caching and visibility  
- Predictable but rigid structure  

Example:

```http
GET /books/1
```

```json
{ "id": 1, "title": "RESTful in Action" }
```

REST brings order to chaos. When teams grow, those conventions become discipline.  
But the tradeoff is rigidity — the shape of your data and routes are locked once published. And it is not elegant as long as your service grows complex.

---

### GraphQL — The Client’s Voice

GraphQL came later, born from frontend fatigue:  
mobile clients, multiple screen layouts, and endless REST endpoints that never quite fit.

![GraphQL Query Flow](/assets/images/service_design/api_comparison/graphql.png)  
*Figure: GraphQL query compares  with Rest query.*

Instead of serving pre-defined shapes, the client describes exactly what it wants:

```graphql
{
  book(id: 1) {
    title
    author
  }
}
```

The server resolves each field dynamically.  
You fetch less, but you compute more.

GraphQL gives clients freedom and that freedom isn’t free.  
Caching becomes tricky, queries can explode in complexity,  
and the backend must now understand data dependency graphs.

Still, when used well, it’s elegant.  
It turns APIs from predefined menus into conversational interfaces.

However, GraphQL is not a blank check.  
If a query starts joining too many tables or traversing deep relationships,  
the cost quickly outweighs the convenience — both in performance and security.  
Uncontrolled queries can lead to massive joins, heavy resolver chains,  
and even the unintentional exposure of sensitive data that was never meant to be queried.  
Good GraphQL design enforces **depth limits**, **query cost analysis**, and **field-level access control**,  
reminding us that flexibility only works when it’s responsibly constrained.

---

### RPC — The Native Instinct

RPC feels like the most primitive and direct form of communication.  
You don’t ask for resources — you call functions.  
A request is a method, some parameters, and a result.

![RPC Communication Flow](/assets/images/service_design/api_comparison/rpc.png)  
*Figure:  RPC sequence — a package of method call and parameters.*

Example (JSON-RPC style):

```json
{
  "method": "getBookById",
  "params": { "id": 1 }
}
```

RPC works well inside systems — microservices, internal APIs, or trusted layers.  
It’s fast, minimal, and easy to reason about.  
But it’s opaque to outsiders: there’s no universal meaning in `getBookById`, no shared semantics.

According to a lot tutorials, if REST is public diplomacy, RPC is internal code-switching.

---

### The Trade-offs

| Aspect | REST | GraphQL | RPC |
|--------|------|----------|-----|
| Core Model | Resource | Query | Procedure |
| Flexibility | Medium | High | Low |
| Performance | Moderate | Variable | High |
| Caching | Strong | Weak | Custom |
| Complexity | Low | High | Low |
| Typical Use | Public APIs | Multi-device frontends | Internal services |

Each model solves a different layer of communication pain:

- REST builds consistency across teams.  
- GraphQL optimizes flexibility for clients.  
- RPC maximizes efficiency between trusted peers.

---

### Reflections

I’ve stopped asking “which is better.”  
What matters is where the complexity sits.

REST moves it to the interface contract.  
GraphQL shifts it to the query and resolver layer.  
RPC keeps it inside the code.

In production systems, you’ll often see all three:

- REST at the edges, where humans and documentation matter.  
- GraphQL at the mid-layer, aggregating front-end data.  
- RPC underneath, linking microservices.

It’s less about evolution, more about layering.

---

### Closing Thoughts

Whether it’s REST, GraphQL, or RPC,  
we’re still solving the same problem:  
how to make two systems talk clearly, efficiently, and predictably.  

Btw , I  developed a  mobile application several years ago, During the design phase, we have a heated  debate on which communication model to use. Restful, gRPC, GraphQL, thrift, etc. Each has its pros and cons. In the end, we chose gRpc for its performance and strong typing, which suited our needs well. One important pain point, we have multiple frontend clients (iOS, Android, Web, TV and some Android-based Terminal devices) that require consistent data structures. In this scenario, gRPC can  maintain strong typing across different platforms. Besides, gPRC provide unified API definition through Protocol Buffers, which simplifies client development. 
Of course, it also takes some complexity on the server side, but overall, it was a worthwhile trade-off for our use case.

---

Build to wonder.
