---
layout: post
title: "Stop Using new. Use a Static Factory Method Instead."
date: 2026-03-24
author: Jason Schindler
categories: [csharp, design-patterns]
tags: [csharp, design-patterns, static-factory-method, oop, best-practices]
---

There's a design pattern I keep coming back to in C#, and once you start using it, you'll wonder how you ever lived without it. It's called the **Static Factory Method**, and it's one of those things that looks almost too simple to matter — until you realize how much cleaner your code becomes.

Here's the gist. Instead of this:

```csharp
var order = new Order(userId, productId, quantity);
```

You write this:

```csharp
var order = Order.PlaceOrder(userId, productId, quantity);
```

Same result. Completely different story.

---

## What the Pattern Actually Looks Like

The structure is straightforward. You make your constructor private so nobody can `new` up your class directly. Then you expose one or more `public static` methods that do the construction for you and return the fully-formed object.

```csharp
public class Order
{
    public string UserId { get; private set; }
    public string ProductId { get; private set; }
    public int Quantity { get; private set; }
    public DateTime PlacedAt { get; private set; }

    public static Order PlaceOrder(string userId, string productId, int quantity)
    {
        Order order = new Order();
        order.UserId = userId;
        order.ProductId = productId;
        order.Quantity = quantity;
        order.PlacedAt = DateTime.UtcNow;

        return order;
    }

    private Order()
    {
        // Private constructor — you shall not new
    }
}
```

Simple. Clean. And far more expressive once object creation involves any real behavior.

---

## Why I Love It

### 1. The Method Name Tells a Story

A constructor is anonymous. `new Order(...)` tells you *what* you're making, but nothing about *why* or *how*. With a static factory, the name carries meaning.

Compare these:

```csharp
// 😐 What kind of payment? What state is it in?
var payment = new Payment(amount, "PENDING");

// 😍 Oh. It's a new, pending payment.
var payment = Payment.CreatePending(amount);
```

This matters more than it sounds. Six months from now, when you're reading code at 11pm trying to track down a bug, that name is going to save you.

---

### 2. You Get Control Over Validation and Failure

Constructors tend to push you into an all-or-nothing model — usually throwing exceptions when something is invalid.

With a static factory, you have options. You can validate, handle failures gracefully, or choose how you want to signal problems:

```csharp
public static Order? PlaceOrder(string userId, string productId, int quantity)
{
    if (string.IsNullOrWhiteSpace(userId)) return null;
    if (quantity <= 0) return null;

    Order order = new Order();
    // ... rest of setup

    return order;
}
```

You could just as easily return a `Result` type or throw a domain-specific exception. The point is — *you're in control*.

---

### 3. You Can Have Multiple Named Constructors

This is my favorite one. C# doesn't allow you to have two constructors with the same signature. And overloaded constructors with different parameters? They tend to devolve into a mess of `null` checks and optional parameters.

Static factories let you have as many named entry points as you need:

```csharp
public static Order PlaceOrder(string userId, string productId, int quantity) { ... }

public static Order PlaceGiftOrder(string userId, string productId, string recipientId) { ... }

public static Order Reorder(Order previousOrder) { ... }
```

Three completely different ways to create an `Order`, each with a name that explains exactly what it does.

---

### 4. You Control What Gets Built

Because the factory method has access to the private constructor, you can set internal state that you'd never want exposed through a public constructor. Timestamps, generated IDs, calculated fields — all of that can be wired up internally.

```csharp
public static Session StartSession(string userId)
{
    Session session = new Session();
    session.UserId = userId;
    session.SessionId = Guid.NewGuid().ToString();
    session.StartedAt = DateTime.UtcNow;
    session.ExpiresAt = DateTime.UtcNow.AddHours(24);

    return session;
}
```

---

### 5. You Can Return Different Implementations (or Reuse Instances)

This is a big one that often gets overlooked.

A static factory doesn't have to return a new instance every time — or even the same concrete type.

```csharp
public static ILogger Create()
{
    if (Environment.IsDevelopment())
        return new ConsoleLogger();

    return new FileLogger();
}
```

Or even reuse objects:

```csharp
public static Currency FromCode(string code)
{
    return _cache[code];
}
```

You simply can't do this cleanly with constructors. This is where the pattern starts to move from "nice API" into real design power.

---

## When Should You Use It?

I tend to default to this pattern whenever:

- Object creation involves logic beyond simple assignment
- The object represents a domain concept with a meaningful creation event (`PlaceOrder`, `StartSession`, `RegisterUser`)
- I want the API to be expressive rather than just structural
- I expect multiple ways to create the object

---

## When You Probably Shouldn't

There are still plenty of cases where a constructor is perfectly fine:

- Simple DTOs or records
- Value objects with trivial state
- Performance-critical paths where indirection matters
- Objects with no meaningful "creation story"

Use the pattern where it adds clarity — not everywhere by default.

---

## The One Gotcha

If you're using an ORM like Entity Framework, be aware that EF needs to be able to instantiate your objects. The private constructor approach usually works — EF can use a private parameterless constructor — but you may need to configure it explicitly or make it `protected` depending on your setup.

It's a solvable problem, just worth knowing going in.

---

## Wrapping Up

The Static Factory Method isn't some exotic pattern from a dusty design patterns textbook. It's a practical, everyday tool that makes your code more readable, more intentional, and more maintainable.

It turns object creation into something explicit and expressive. Instead of just constructing objects, you're describing *what's happening* in your domain.

Give it a try. Your future self — the one debugging production at 11pm — will thank you.
