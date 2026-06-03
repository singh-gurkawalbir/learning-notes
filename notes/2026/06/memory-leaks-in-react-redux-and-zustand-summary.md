---
title: "Memory Leaks in React, Redux, and Zustand"
slug: "memory-leaks-in-react-redux-and-zustand-summary"
type: "concept"
tags: []
summary: ""
created: 2026-06-03
updated: 2026-06-03
source_question: "What are ways in which memory leaks happen in React. Redux. Zustand explain"
links:
review:
  last_reviewed: null
  next_review: 2026-06-03
  step: 0
  confidence: 0
quiz:
---

**Tags:** React, Redux, Zustand, Memory Management, `useEffect`, Cleanup, Subscriptions, Timers, Event Listeners

## Summary
Memory leaks in React applications, especially when integrating with state management libraries like Redux and Zustand, primarily occur when components unmount but still maintain active references to external resources or subscriptions. This prevents the JavaScript garbage collector from reclaiming the memory associated with the unmounted component and its closures. Proper cleanup using `useEffect`'s return function is essential to prevent these leaks.

## Mental Model
Imagine your React component as a temporary visitor to a party. While it's at the party, it might tell other people (like the `window` object, a timer, or a Redux/Zustand store) to call it back later. If the component leaves the party (unmounts) but forgets to tell those people to stop calling, they'll keep trying to reach it. This persistent "attempt to call back" means the component, even though it's gone, is still being referenced and can't be fully cleaned up, leading to a memory leak.

The key to preventing these leaks is for the component to "clean up its tracks" before it leaves – explicitly removing event listeners, clearing timers, or unsubscribing from stores.

## How it actually works

Memory leaks in React, Redux, and Zustand environments typically stem from unmanaged external resources or subscriptions that hold a reference to an unmounted component's scope.

### React Leaks

React components often interact with the browser environment (e.g., `window` object) or set up internal processes (e.g., timers). If these interactions are not cleaned up when the component unmounts, they can lead to leaks.

1.  **Unsubscribed Event Listeners:**
    When you add an event listener (e.g., to `window`, `document`, or a DOM element) inside a `useEffect` hook and don't remove it in the cleanup function, the listener's callback function (which often closes over the component's state and props) remains active. This keeps the component's closure alive, preventing garbage collection.

    *   **Leak:**
        ```javascript
        import { useEffect } from 'react';
        function MyComponent() {
          useEffect(() => {
            // Adds listener, but never removes it
            window.addEventListener('resize', () => console.log('Resized!'));
          }, []); // Effect runs once
          return <div>...</div>;
        }
        ```
        When `MyComponent` unmounts, the closure for the event listener (which includes `MyComponent`'s scope) persists, preventing garbage collection.

    *   **Fix:**
        ```javascript
        import { useEffect } from 'react';
        function MyComponent() {
          useEffect(() => {
            const handleResize = () => console.log('Resized!');
            window.addEventListener('resize', handleResize);
            return () => { // Cleanup function
              window.removeEventListener('resize', handleResize);
            };
          }, []);
          return <div>...</div>;
        }
        ```

2.  **Uncleared Timers:**
    `setTimeout` or `setInterval` calls that are initiated in a `useEffect` hook but not cleared (`clearTimeout`, `clearInterval`) before the component unmounts will continue to execute. If these timer callbacks attempt to update state on an unmounted component, it can lead to warnings and, more importantly, the callback's closure (and the component's scope) being kept alive.

    *   **Leak:**
        ```javascript
        import { useEffect, useState } from 'react';
        function TimerComponent() {
          const [count, setCount] = useState(0);
          useEffect(() => {
            // Sets timer, but never clears it
            setInterval(() => setCount(c => c + 1), 1000);
          }, []);
          return <div>Count: {count}</div>;
        }
        ```
        If `TimerComponent` unmounts, the `setInterval` continues to run, attempting to update state on a non-existent component.

    *   **Fix:**
        ```javascript
        import { useEffect, useState } from 'react';
        function TimerComponent() {
          const [count, setCount] = useState(0);
          useEffect(() => {
            const intervalId = setInterval(() => setCount(c => c + 1), 1000);
            return () => { // Cleanup function
              clearInterval(intervalId);
            };
          }, []);
          return <div>Count: {count}</div>;
        }
        ```

3.  **External Subscriptions (e.g., Observables, WebSockets):**
    Similar to event listeners, if a component subscribes to an external data source (like a WebSocket, a custom observable, or a third-party library's event emitter) and doesn't explicitly unsubscribe when it unmounts, the subscription listener will persist, holding a reference to the component's context.

### Redux Leaks

Redux itself, as a predictable state container, is generally not a direct source of memory leaks. Leaks typically arise from how Redux state is consumed or interacted with within React components.

1.  **Forgotten `store.subscribe()` Cleanup:**
    If you manually subscribe to the Redux store using `store.subscribe(listener)` outside of `react-redux` hooks, you must call the returned unsubscribe function when the subscribing component unmounts. `react-redux`'s `useSelector` hook handles this cleanup automatically.

    *   **Leak:**
        ```javascript
        import { useEffect } from 'react';
        import store from './reduxStore'; // Your Redux store
        function ManualReduxListener() {
          useEffect(() => {
            // Subscribes, but doesn't unsubscribe
            store.subscribe(() => {
              console.log('Redux state changed!', store.getState());
            });
          }, []);
          return <div>...</div>;
        }
        ```
        The listener function is held by the Redux store, preventing its garbage collection (and the component's closure) even after `ManualReduxListener` unmounts.

    *   **Fix:**
        ```javascript
        import { useEffect } from 'react';
        import store from './reduxStore'; // Your Redux store
        function ManualReduxListener() {
          useEffect(() => {
            const unsubscribe = store.subscribe(() => {
              console.log('Redux state changed!', store.getState());
            });
            return () => { // Cleanup function
              unsubscribe();
            };
          }, []);
          return <div>...</div>;
        }
        ```

2.  **Large, Unneeded State:**
    While not a "leak" in the traditional sense of uncollected components, storing very large objects or extensive, immutable data in the Redux store that is no longer needed or actively used can consume significant memory. If this state is never cleaned up (e.g., by dispatching actions to remove it), it can lead to increased memory usage over time.

### Zustand Leaks

Zustand's `useStore` hook automatically manages subscriptions and unsubscriptions, making direct leaks from the hook rare. However, similar to Redux, manual subscription requires cleanup.

1.  **Manual `store.subscribe()` Without Cleanup:**
    If you directly use `myZustandStore.subscribe()` to listen for state changes without calling the returned unsubscribe function in a cleanup effect, the listener callback will persist, holding a reference to the component's context.

    *   **Leak:**
        ```javascript
        import { useEffect } from 'react';
        import { myZustandStore } from './zustandStore'; // Your Zustand store
        function ManualZustandListener() {
          useEffect(() => {
            // Subscribes, but doesn't unsubscribe
            myZustandStore.subscribe(
              (state) => console.log('Zustand state changed:', state),
              (state) => state.someValue // Selector (optional)
            );
          }, []);
          return <div>...</div>;
        }
        ```
        Similar to Redux, the listener function remains active in the Zustand store's subscription list after the component unmounts.

    *   **Fix:**
        ```javascript
        import { useEffect } from 'react';
        import { myZustandStore } from './zustandStore'; // Your Zustand store
        function ManualZustandListener() {
          useEffect(() => {
            const unsubscribe = myZustandStore.subscribe(
              (state) => console.log('Zustand state changed:', state),
              (state) => state.someValue
            );
            return () => { // Cleanup function
              unsubscribe();
            };
          }, []);
          return <div>...</div>;
        }
        ```

2.  **Retained Large State:**
    Similar to Redux, if large, immutable data is stored in a Zustand store and is no longer relevant or actively used, but never removed from the store, it can lead to increased memory consumption.

## Why
Memory leaks occur because the JavaScript garbage collector (GC) cannot reclaim memory for objects that are still reachable. In the context of React components, if an external entity (like `window`, a timer, or a state management store) holds a reference to a function (e.g., an event handler, a timer callback, a subscription listener) that was defined within a component's scope, that function's closure (and by extension, the component's variables and context) remains "reachable." Even if the component itself has unmounted from the DOM, the GC sees this external reference and assumes the memory is still in use, thus failing to collect it. The `useEffect` cleanup function provides a mechanism to break these external references, making the component's memory eligible for garbage collection.

## Failure Modes
*   **Forgetting `useEffect` cleanup:** The most common failure mode, leading to persistent event listeners, timers, or subscriptions.
*   **Manual subscriptions without unsubscriptions:** Directly using `store.subscribe()` in Redux or Zustand without calling the returned `unsubscribe` function.
*   **Storing excessively large data:** Keeping large, obsolete data structures in global state stores (Redux, Zustand) without proper mechanisms to clear or prune them.
*   **Circular references (less common in modern React):** While JavaScript's GC is good at handling circular references, complex object graphs can sometimes prevent collection if an outside root reference exists.

## Quiz

1.  A React component adds a `window.scroll` event listener in `useEffect`. What is the primary mechanism to prevent a memory leak when this component unmounts?
    a) Using `useCallback` for the event handler.
    b) Returning a cleanup function from `useEffect` that calls `window.removeEventListener`.
    c) Wrapping the event listener in `useMemo`.
    d) Relying on React's automatic garbage collection.

2.  Consider a Redux application where a component uses `store.subscribe()` directly. If the component unmounts without proper cleanup, what specific part of the application will hold onto a reference, preventing garbage collection?
    a) The React DOM tree.
    b) The Redux store's internal subscription list.
    c) The browser's event loop.
    d) The component's local state.

3.  Which of the following scenarios is *least likely* to cause a direct memory leak related to component unmounting in a well-structured React/Zustand application?
    a) Setting a `setInterval` without clearing it.
    b) Adding a `document.click` listener without removing it.
    c) Using Zustand's `useStore` hook to select a small piece of state.
    d) Manually subscribing to a WebSocket connection without an unsubscribe handler.
