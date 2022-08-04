---
layout: post
title: "What is Event Loop – Things To Know"
date: 2022-05-03
category:
  - JavaScript
  - Programming
image: assets/img/blog/event-loop.png
author: Andy
tags: JavaScript 
---
The event loop is what allows javascript to handle thousands and thousands of requests with just a single thread. This architecture allows Javascript and/or Node.js engines to offload operations to the system kernel.

The generic explanation given above is all good but how does it actually work? We are going to talk about this later in this article but first, we need to understand how javascript works and where the need for the event loop comes in.

Javascript is a single-threaded programming language. In the old days, javascript was used in browsers for basic interactive functionalities. At first, developers used Javascript for minor interactive functionalities for the websites but little did they know that javascript would become such a monster that it is today. Nowadays you can develop desktop, mobile or web applications using a programming language that no one took seriously at first. Basically, you can learn one language and be truly a full-stack developer.

In this article, we are going to further discuss what an Even loop means, how it works and what are the major considerations to keep in mind. So let’s get into it.

#### Asynchronous Programming

Let’s start explaining what asynchronous programming is and how it can change the approach to programming. The typical multithreaded application works with operations and every new operation will need a new thread to perform that operation in the OS kernel, of course, the real thing is a bit more complicated. So let’s take a real-life example.

Imagine you have to manage a restaurant. Let’s suppose the kitchen acts as a processor, waiters act as thread and orders act as operations that the processor has to process. If a waiter had to wait for every order to be done before starting to wait for another table that would mean that they can only serve as many tables as there are waiters at the restaurant. However, a classical restaurant works with an asynchronous approach. The waiter would leave instructions to the kitchen and would not wait for the order to be done and would take another order instead. After the order is done the waiter would be notified to serve the meal to the customer.
In programming, this would be relevant in terms of requests and responses. This means that the number of requests that an asynchronous approach can serve at the time is vastly superior to the synchronous approach.

Now that we know how asynchronous programming can change classical application flow, let’s take a look at how exactly the event loop works under the hood. There are three parts in the event loop that we need to know about including Call Stack, Event Queue/Callback Queue, WebAPIs (in the case of NodeJs, C++ APIs). We are going to talk about these parts down below.

#### How the Event Loop Works

Call stack holds all the synchronous function calls. Call stack will execute code in a last in first out approach. So last the most inner nested code will be executed first and then one level above that and so on. If the function call is asynchronous it will add a new call entry on WebAPIs and the API will add a new call to the callback queue after the asynchronous action is done.

Now we have two stacks with functions to be executed. This is where the Event Loop comes in. Event Loop’s job is to watch Callback Queue and Call Stack, whenever Call Stack is empty and we have called in Callback Queue. The event loop will move calls to Call Stack and execute them in succession.
So to explain it in different terms, the event loop will wait until all blocking/synchronous code is done and then add asynchronous calls into the call stack. This simple architectural approach is what allows javascript to handle way more requests than the traditional multi-threaded synchronous approach.

#### What Should You Keep in Mind?

Before you start using javascript for every task, there are some things to consider in the first place. The important thing to note is that there are several drawbacks to this architecture. The biggest disadvantage of this approach is that it is not recommended to handle slow or blocking operations.

Slow Operations are functions that need a long time to be completed are slow for javascript, this might seem really obvious but in the traditional approach, slow instruction for programs will only block one thread and other threads are still available for other requests, but since we only have one thread in NodeJs it will block execution for every incoming request.

#### Frequently Asked Questions
##### What is an event loop in JavaScript?
The event loop is an architectural design pattern that allows code to run asynchronously on the javascript engines and makes it possible for code to execute blocking instructions first and when available execute asynchronous functions.

##### How does the Event Loop work?
The event loop utilizes three major components, Call Stack, Event Queue, and WebAPIs/C++ APIs. It manages function calls in a way that asynchronous events would go into the event queue and will be called only after the call stack is empty.