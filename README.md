# The-Reconciliation-Algorithm-in-React


In React, the **Reconciliation Algorithm** has evolved over time. While React doesn’t have “types” in a traditional classification sense, we can understand the **different reconciliation mechanisms** across versions and rendering modes:

---

## 🔢 Types or Variants of Reconciliation in React

### 1. **Stack Reconciler (React 15 and earlier)**

* **Synchronous & Recursive**
* One big traversal – cannot be paused or split
* UI freezes on large updates
* No prioritization or interruption

🔸 **Used in:** React 15 and earlier

---

### 2. **Fiber Reconciler (React 16 and above)**

* **Asynchronous & Interruptible**
* Breaks updates into **units of work**
* Allows **pausing, resuming, canceling** updates
* Enables **prioritized rendering**

🔸 **Used in:** React 16+

---

### 3. **Concurrent Reconciler (React 18+)**

* **Built on Fiber**, but with **Concurrent Features**
* Supports **time-slicing, transitions, and streaming**
* Lets React **prepare UI in background** without blocking the main thread
* Uses `startTransition`, `useDeferredValue`, etc.

🔸 **Used in:** React 18+ (opt-in via concurrent features)

---

## 🧠 Summary Table

| Type                  | React Version | Key Feature                       | Scheduling  |
| --------------------- | ------------- | --------------------------------- | ----------- |
| Stack Reconciler      | React ≤15     | Sync rendering, no interruption   | Blocking    |
| Fiber Reconciler      | React 16–17   | Async units of work, pause/resume | Cooperative |
| Concurrent Reconciler | React 18+     | Prioritized, concurrent rendering | Concurrent  |

Let’s go **deep into each type of Reconciliation algorithm in React**, explaining the mechanism, evolution, behavior, and giving examples where possible:

---

## 🧱 1. **Stack Reconciler** (React 15 and earlier)

### ✅ Description:

* Used a **recursive, synchronous** approach.
* Rendered the entire tree in **one pass**.
* **No pausing, no prioritization** — it was all-or-nothing.
* UI would freeze during heavy computation or long renders.

### 🔁 How It Worked:

```tsx
function App() {
  return (
    <div>
      <Header />
      <Sidebar />
      <MainContent />
    </div>
  );
}
```

React calls:

```ts
render(App) → render(Header) → render(Sidebar) → render(MainContent)
```

All recursively, synchronously, and blocking the thread.

### ❌ Problem:

If `MainContent` takes 500ms to render, your **entire UI becomes unresponsive**.

---

## 🧵 2. **Fiber Reconciler** (React 16 and 17)

### ✅ Description:

* Introduced in React 16.
* Core is now a **linked list of “Fiber nodes”**, not recursion.
* Supports **breaking render work into units**.
* Enables:

  * **Pausing** mid-render
  * **Resuming** later
  * **Aborting**
  * **Reusing work**
  * **Laying the foundation for concurrency**

### 🧠 Key Features:

* **Work Loop API**: React processes the tree using a manual loop.
* **Double Buffering**: Work-in-progress vs current tree.
* **Scheduling**: Slight prioritization possible (though basic).

### 🔍 Fiber Node Example:

Each component becomes a `Fiber` object:

```ts
{
  type: 'Header',
  child: Fiber for Sidebar,
  sibling: Fiber for MainContent,
  return: Fiber for App,
  stateNode: DOM node or component instance
}
```

### 🔧 Example:

```tsx
function App() {
  return (
    <>
      <Header />
      <HeavyComponent />
    </>
  );
}
```

In Fiber, React builds the tree step-by-step:

```ts
→ Begin Work: App
→ Begin Work: Header
→ Complete Work: Header
→ Begin Work: HeavyComponent (pause if time runs out)
→ Resume later
→ Commit all once tree is built
```

> **Result**: Less UI freezing, more responsiveness.

---

## ⚡ 3. **Concurrent Reconciler** (React 18+)

### ✅ Description:

* **Built on Fiber**, but adds **true concurrency**.
* React can now **prepare multiple UI trees in parallel**.
* Let’s React **respond to high-priority updates instantly** (like typing).
* Introduces:

  * **Transitions** (`startTransition`)
  * **Deferred values** (`useDeferredValue`)
  * **Streaming SSR**
  * **Concurrent Suspense**

### 🧠 Core Change:

Uses **time-slicing** + `scheduler` package to **prioritize tasks**.

### 🔧 Example:

```tsx
import { startTransition } from 'react';

function SearchBar() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  function handleInput(e) {
    const value = e.target.value;
    setQuery(value);

    startTransition(() => {
      // low-priority update
      setResults(search(value));
    });
  }

  return <input value={query} onChange={handleInput} />;
}
```

> Typing (high-priority) remains smooth, while searching (low-priority) can delay.

---

## 📊 Final Summary Table

| Feature               | Stack Reconciler | Fiber Reconciler    | Concurrent Reconciler |
| --------------------- | ---------------- | ------------------- | --------------------- |
| Version               | React ≤15        | React 16, 17        | React 18+             |
| Core structure        | Recursive stack  | Linked list (Fiber) | Fiber + Scheduler     |
| Interruption          | ❌                | ✅                   | ✅                     |
| Scheduling            | ❌                | Basic               | Advanced (priority)   |
| Pausing/resuming work | ❌                | ✅                   | ✅                     |
| Streaming SSR         | ❌                | Partial             | ✅                     |
| Concurrent features   | ❌                | ❌                   | ✅                     |

Let’s dive deep into **React Fiber**, the core architecture introduced in **React 16** that fundamentally changed how React handles rendering, reconciliation, and updates.

---

## 🧠 What is React Fiber?

**Fiber** is a **reimplementation of React’s reconciliation algorithm**, designed to enable:

| Capability              | Purpose                                              |
| ----------------------- | ---------------------------------------------------- |
| Interruptible rendering | Prevent UI blocking for long updates                 |
| Time-slicing            | Break rendering into chunks and schedule efficiently |
| Prioritization          | Handle more important updates (e.g., typing) first   |
| Concurrency foundation  | Power features like `startTransition`, Suspense, etc |

---

## 📦 Why React Needed Fiber

The old algorithm (stack reconciler) was:

* **Synchronous**
* **Recursive**
* Not **interruptible** — it rendered the entire component tree in one go.

**Fiber allows React to:**

* Pause work
* Resume work later
* Restart or abandon work
* Assign priorities to work

---

## 🧱 How Fiber Works Internally

React converts each component into a **Fiber node** (a JavaScript object):

```js
const fiberNode = {
  type: Component,
  stateNode: instance or DOM node,
  child: next child fiber,
  sibling: next sibling fiber,
  return: parent fiber,
  alternate: previous version of this fiber (for diffing)
};
```

### 🔁 Fiber Tree Structure

* Fiber nodes form a **tree structure** (similar to VDOM).
* **Linked list traversal** → one node at a time (instead of recursion).

---

## 🔄 Fiber Phases

### 1. **Render Phase (Can be Interrupted)**

* Build a **work-in-progress** tree using:

  * `beginWork(fiber)`
  * `completeWork(fiber)`
* Builds an **effect list** of mutations.

### 2. **Commit Phase (Synchronous)**

* Apply DOM updates
* Run lifecycle methods (`componentDidMount`, `useEffect`)
* Non-interruptible

---

## 🧪 Example: Component Rendering with Fiber

```tsx
<App>
  <Header />
  <Main />
</App>
```

React builds fibers:

```text
App (root)
├── Header (child)
└── Main (sibling of Header)
```

Each becomes a unit of work:

```ts
performUnitOfWork(AppFiber)
→ performUnitOfWork(HeaderFiber)
→ performUnitOfWork(MainFiber)
```

If React is busy, it can pause between `Header` and `Main` and continue later.

---

## 🔁 Double Buffering

* **Current Tree** → what’s currently rendered
* **Work-in-Progress Tree** → being built in memory
* On commit, WIP replaces current

---

## 📊 Benefits of Fiber

| Feature                  | Benefit                               |
| ------------------------ | ------------------------------------- |
| Interruptible rendering  | Prevents jank on large renders        |
| Priority scheduling      | Faster response to user input         |
| Smooth animations        | Lower-priority updates don’t block UI |
| Better error handling    | Supports error boundaries             |
| Foundation for React 18+ | Concurrent Mode, Suspense, etc.       |

---

## 🧠 Key APIs Enabled by Fiber

* `ReactDOM.createRoot()` (concurrent features)
* `startTransition()`
* `useDeferredValue()`
* `Suspense` for data fetching
* `lazy()` loading

Here’s a **complete example** of how **React Fiber works under the hood**, including the phases, data structure, and how it enables **interruptible rendering** and **prioritization** — with simple code and a behind-the-scenes walkthrough.

---

## 🧱 Sample React Code

```tsx
function App() {
  return (
    <div>
      <Header />
      <MainContent />
    </div>
  );
}

function Header() {
  return <h1>Hello!</h1>;
}

function MainContent() {
  let start = performance.now();
  while (performance.now() - start < 300) {
    // Simulate heavy computation
  }
  return <p>Heavy content</p>;
}
```

---

## 🔍 How Fiber Handles This Internally

### 🔁 Step-by-Step Behind the Scenes

1. **Virtual DOM is created:**

```json
<App>
  <Header />
  <MainContent />
</App>
```

2. **React builds a Fiber Tree (Work-in-Progress):**

Each component becomes a Fiber Node:

```ts
{
  type: App,
  child: HeaderFiber,
  sibling: MainContentFiber,
  return: null
}
```

Each fiber looks like:

```ts
const fiber = {
  type: Component,
  stateNode: instance or DOM node,
  child: Fiber,
  sibling: Fiber,
  return: Parent Fiber,
  alternate: Previous version of this Fiber,
};
```

3. **Begin Work Phase:**

   * React starts with `<App />`
   * Moves to `<Header />` and creates its fiber
   * Then to `<MainContent />` and **pauses if time is up**

```ts
function workLoop(deadline) {
  while (nextUnitOfWork && deadline.timeRemaining() > 1) {
    nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
  }

  if (nextUnitOfWork) {
    requestIdleCallback(workLoop);
  }
}
```

✅ This means React **yields control** before blocking the main thread.

---

## 📤 Commit Phase

Once the whole tree is built:

* React commits all side-effects and DOM updates.
* It swaps `current` ↔ `work-in-progress` tree (double buffering).

---

## 🧠 Why Fiber Is Powerful

Without Fiber (React 15):

* Heavy `MainContent` blocks the render → **UI freezes**

With Fiber (React 16+):

* React renders `Header` quickly
* Pauses before rendering heavy `MainContent`
* User sees partial UI immediately → **perceived performance improves**

---

## 🧩 Bonus: Prioritizing Updates (React 18+ using Fiber + Concurrent Mode)

```tsx
import { startTransition } from 'react';

function SearchBox() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  function handleChange(e) {
    const value = e.target.value;
    setQuery(value);

    startTransition(() => {
      setResults(slowSearch(value));
    });
  }

  return <input onChange={handleChange} value={query} />;
}
```

* Typing stays responsive.
* Search results are low-priority and scheduled in the background.

---

