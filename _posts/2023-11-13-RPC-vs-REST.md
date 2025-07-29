---
layout: post
title: RPC vs REST
subtitle: When to use What?
tags: [Network]
comments: true
author: Hank Kim
thumbnail-img: /assets/img/rpc.png
---

# RPC vs REST: When to Use What

## The Ethereum Moment That Started It All

Picture this: you're trying to get the latest block number from the Ethereum blockchain. You fire up your terminal and run:

```bash
curl https://eth.llamarpc.com \
  --request POST \
  --header "Content-Type: application/json" \
  --data '{"jsonrpc":"2.0", "method":"eth_blockNumber", "params":[], "id":1}'
```

And boom! You get back:

```json
{ "jsonrpc": "2.0", "id": 1, "result": "0x12a7d9f" }
```

That hex number `0x12a7d9f` translates to block 19,721,183. Pretty straightforward, right? But here's the thing - you just used RPC, not REST. And if you're like most developers, you probably didn't even think twice about it.

This got me thinking: when do we actually choose RPC over REST, and why does it matter?

## What's Really Going On Here?

### The RPC Way: "Hey Server, Do This Thing"

Remote Procedure Call (RPC) is like calling a function on another computer. In our Ethereum example, we're literally calling the `eth_blockNumber` method on a remote server. It's direct, it's explicit, and it feels just like calling a local function.

```javascript
// It's like doing this, but the function lives on another server
const blockNumber = eth_blockNumber();
```

The beauty of RPC is in its simplicity. You know exactly what function you're calling, what parameters it expects, and what it returns. No guessing, no REST conventions to remember.

### The REST Way: "Here's a Resource, Do CRUD Operations"

REST (Representational State Transfer) is different. It's all about resources and standard operations. Instead of calling functions, you perform actions on resources using HTTP verbs.

If Ethereum used REST (which it doesn't), getting the block number might look like:

```bash
GET /api/blockchain/latest-block
```

The philosophy is different. REST says "here's a resource (latest block), and you can GET, POST, PUT, or DELETE it." RPC says "here's a function (eth_blockNumber), call it."

## The Tale of Two Architectures

Let me break down the real differences with some practical examples:

### RPC in Action

```javascript
// Adding a product - RPC style
POST /addProduct
{
  "name": "Coffee Mug",
  "price": "15.99",
  "category": "Kitchen"
}

// Getting product details - RPC style
POST /getProduct
{
  "productId": "123"
}

// Updating price - RPC style
POST /updateProductPrice
{
  "productId": "123",
  "newPrice": "12.99"
}
```

Notice how everything is a POST request? That's because we're calling functions, not manipulating resources. The function name is in the URL, and parameters go in the body.

### REST in Action

```javascript
// Adding a product - REST style
POST /products
{
  "name": "Coffee Mug",
  "price": "15.99",
  "category": "Kitchen"
}

// Getting product details - REST style
GET /products/123

// Updating price - REST style
PUT /products/123
{
  "price": "12.99"
}
```

See the difference? REST uses different HTTP verbs and treats everything as a resource with a URL. The action is implied by the HTTP method.

## When RPC Makes Perfect Sense

RPC shines when you're dealing with actions that don't fit neatly into CRUD operations. Think about these scenarios:

### Complex Operations

```javascript
// Processing a payment - this isn't really "creating" something
POST /processPayment
{
  "amount": "99.99",
  "cardToken": "xyz123",
  "merchantId": "abc456"
}
```

### Remote Computations

```javascript
// Running fraud detection - clearly an action, not a resource
POST /detectFraud
{
  "transactionData": {...},
  "userHistory": {...}
}
```

### System Operations

```javascript
// Restarting a service - definitely not CRUD
POST /restartService
{
  "serviceId": "user-auth-service"
}
```

These operations are naturally function-like. Trying to force them into REST can feel awkward and artificial.

## When REST Rules the World

REST is fantastic when you're dealing with actual resources that users create, read, update, and delete:

### Data Management

```javascript
// Managing blog posts
GET / posts; // List all posts
GET / posts / 123; // Get specific post
POST / posts; // Create new post
PUT / posts / 123; // Update entire post
PATCH / posts / 123; // Partial update
DELETE / posts / 123; // Delete post
```

### User Management

```javascript
// Managing users
GET / users;
POST / users;
PUT / users / 456;
DELETE / users / 456;
```

REST's uniform interface makes it predictable. Once you know the resource URL, you know exactly how to interact with it.

## The Modern Plot Twist: Enter gRPC and tRPC

Here's where things get interesting. While REST dominated web APIs for years, RPC never really went away. In fact, it's making a serious comeback with modern implementations:

### gRPC: RPC Gets a Makeover

Google's gRPC runs on HTTP/2 and uses Protocol Buffers for serialization. It's fast, efficient, and supports streaming - something traditional REST struggles with.

```protobuf
service ProductService {
  rpc GetProduct(GetProductRequest) returns (Product);
  rpc CreateProduct(CreateProductRequest) returns (Product);
  rpc UpdateProduct(UpdateProductRequest) returns (Product);
}
```

### tRPC: TypeScript's Answer

tRPC is gaining huge popularity in the TypeScript world because it provides end-to-end type safety:

```typescript
// Server
const appRouter = router({
  getProduct: procedure
    .input(z.object({ id: z.string() }))
    .query(({ input }) => {
      return getProductById(input.id);
    }),
});

// Client (fully typed!)
const product = await trpc.getProduct.query({ id: "123" });
```

The client knows exactly what the server expects and what it returns, all at compile time.

## The Microservices Reality Check

In today's microservices world, the choice isn't always clear-cut. Here's what I've learned from real projects:

### Server-to-Server Communication

For internal microservice communication, RPC often wins:

- You control both ends of the conversation
- Performance matters more than HTTP semantics
- Type safety is crucial
- You're calling specific business functions

### Client-Server Communication

For client-facing APIs, REST still has advantages:

- Universal HTTP tooling
- Easy to cache and proxy
- Familiar to frontend developers
- Great for mobile apps with limited connectivity

## Making the Choice: A Practical Framework

Here's my decision tree for choosing between RPC and REST:

### Choose RPC When:

- You're building internal APIs between services you control
- Operations don't map well to CRUD (calculations, processing, system operations)
- Performance is critical
- You need strong typing across the wire
- Real-time features like streaming are important

### Choose REST When:

- Building public APIs for external consumption
- Working with resources that have clear CRUD semantics
- Caching and HTTP semantics provide value
- Team familiarity with REST is high
- Mobile clients need predictable, cacheable responses

## The Hybrid Approach: Why Not Both?

Many successful systems use both. Here's a pattern I've seen work well:

```javascript
// Public API - REST for resources
GET / api / v1 / users / 123;
POST / api / v1 / orders;

// Internal API - RPC for operations
POST / internal / processOrder;
POST / internal / calculateShipping;
POST / internal / sendNotification;
```

## Wrapping Up: It's Not About Right or Wrong

The RPC vs REST debate isn't about finding the "correct" answer - it's about choosing the right tool for the job. REST isn't going anywhere, and neither is RPC. In fact, with technologies like GraphQL (which has RPC-like query capabilities) and the rise of real-time applications, we're seeing more hybrid approaches.

The key is understanding what each paradigm does well:

- **RPC excels at operations and actions**
- **REST excels at resource manipulation**

That Ethereum API call we started with? It's RPC because blockchain operations are naturally function-like. Getting the current block number is calling a specific function, not manipulating a REST resource.

But if you were building a blog, managing posts and comments would naturally fit REST patterns.

The best developers I know don't pick sides - they pick the right tool for each specific problem. And sometimes, that means using both in the same system.

What's your experience been? Have you found situations where the "obvious" choice turned out to be wrong? I'd love to hear your stories in the comments.
