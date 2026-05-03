---
title: "JavaScript Event Loop Quick Example"
slug: "javascript-event-loop-quick-example"
tags: ["javascript", "event-loop", "concurrency", "async", "fundamentals"]
summary: "This example demonstrates how the JavaScript event loop handles asynchronous operations like `setTimeout` to maintain a non-blocking execution."
created: 2026-05-03
updated: 2026-05-03
source_question: "Give me a small example of how event loop works"
review:
  last_reviewed: null
  next_review: 2026-05-03
  step: 0
  confidence: 0
quiz:
---

The JavaScript event loop is what allows JS to perform non-blocking operations despite being single-threaded. It constantly checks if the call stack is empty. If it is, it takes the first function from the callback queue and pushes it onto the call stack to be executed.

Here's a small example:

```javascript
console.log("Start");

setTimeout(() => {
  console.log("Timeout callback executed");
}, 0); // This will run after "End"

console.log("End");
```

**Explanation:**

1.  `console.log("Start")` is pushed onto the call stack and executes immediately, printing "Start".
2.  `setTimeout` is called. It's a Web API function, so it's handed off to the browser (or Node.js runtime) to manage. The callback `() => { console.log("Timeout callback executed"); }` is registered. Even with a 0ms delay, it doesn't execute immediately because it's asynchronous.
3.  `console.log("End")` is pushed onto the call stack and executes immediately, printing "End".
4.  The call stack is now empty.
5.  At this point, the browser's timer for `setTimeout` has expired (or will very quickly). The `setTimeout` callback is moved from the Web API environment to the **callback queue** (also known as the task queue or message queue).
6.  The **event loop** sees that the call stack is empty and that there's a callback in the callback queue. It takes the `setTimeout` callback and pushes it onto the call stack.
7.  The callback executes, printing "Timeout callback executed".

The output will be:
```
Start
End
Timeout callback executed
```
This shows how `setTimeout` (and other async operations like network requests or user events) don't block the main thread, allowing synchronous code to finish first.
