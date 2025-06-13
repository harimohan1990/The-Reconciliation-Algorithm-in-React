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

In **vanilla JavaScript**, there's **no built-in reconciliation engine** like React’s — you directly manipulate the DOM. However, **some libraries or techniques mimic reconciliation behavior** by implementing **diffing and efficient updates**.

### 🔁 Reconciliation-Like Patterns in JavaScript:

| Tool/Approach       | How It Mimics React's Reconciliation                                    |
| ------------------- | ----------------------------------------------------------------------- |
| **React** (via JSX) | Virtual DOM + Fiber reconciler                                          |
| **Preact**          | Lightweight React alternative with diffing                              |
| **Svelte**          | Compiles away the virtual DOM; updates DOM surgically                   |
| **Mithril.js**      | Has its own virtual DOM + diffing engine                                |
| **Snabbdom**        | A minimalist virtual DOM library with reconciliation                    |
| **Manual Diffing**  | You write your own logic to compare DOM/state and apply minimal updates |

---

### 🔧 Manual Example in Vanilla JS:

```js
// Old virtual DOM
const prev = { tag: 'div', children: ['Hello'] };

// New virtual DOM
const next = { tag: 'div', children: ['Hello World'] };

// Reconcile
if (prev.children[0] !== next.children[0]) {
  document.querySelector('div').textContent = next.children[0];
}
```

🔍 This is the idea behind reconciliation — **compare virtual state → update actual DOM minimally**.

Here’s a **simple mini React-like reconciler** built in **vanilla JavaScript** that demonstrates the core idea behind **reconciliation: diffing virtual DOM trees and updating the real DOM minimally**.

---

## 🧱 Step-by-Step: Mini React-Like Reconciler

### ✅ 1. Virtual DOM Element Creator

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat().map(child =>
      typeof child === 'object' ? child : createTextElement(child)
    )
  };
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: { nodeValue: text },
    children: []
  };
}
```

---

### ✅ 2. Render to Real DOM

```js
function render(vNode, container) {
  const dom =
    vNode.type === 'TEXT_ELEMENT'
      ? document.createTextNode('')
      : document.createElement(vNode.type);

  // Apply props
  Object.entries(vNode.props || {}).forEach(([key, value]) => {
    dom[key] = value;
  });

  // Render children
  vNode.children.forEach(child => render(child, dom));

  container.appendChild(dom);
}
```

---

### ✅ 3. Simple Diffing & Reconciliation

```js
function diff(prev, next, container) {
  if (!prev) {
    render(next, container);
  } else if (!next) {
    container.removeChild(container.firstChild);
  } else if (prev.type !== next.type) {
    container.replaceChild(createDOM(next), container.firstChild);
  } else if (typeof next === 'string') {
    if (prev !== next) container.firstChild.nodeValue = next;
  } else {
    // Update props
    const dom = container.firstChild;
    Object.keys(next.props).forEach(name => {
      dom[name] = next.props[name];
    });

    // Diff children
    for (let i = 0; i < next.children.length; i++) {
      diff(prev.children[i], next.children[i], dom);
    }
  }
}
```

---

### ✅ 4. Use It Like JSX

```js
const v1 = createElement('div', { id: 'root' },
  createElement('h1', null, 'Hello'),
  createElement('p', null, 'Welcome to Mini React')
);

const v2 = createElement('div', { id: 'root' },
  createElement('h1', null, 'Hello World!'),
  createElement('p', null, 'Updated paragraph')
);

const container = document.getElementById('app');
render(v1, container);

// Simulate diff after 2 seconds
setTimeout(() => {
  container.innerHTML = ''; // Clear
  render(v2, container);    // Re-render with updated vDOM
}, 2000);
```

---

## 🧠 Summary

This mini reconciler:

* Creates a **virtual DOM**
* Renders it to real DOM
* **Diffs old vs new VDOM** and applies **minimal changes**

Here’s how to structure the **Mini React-Like Reconciler** into a **reusable GitHub project**. You can copy this into a folder and run it locally or push to GitHub:

---

## 📁 Folder Structure

```
mini-react/
├── index.html
├── reconciler.js
└── main.js
```

---

### ✅ `index.html`

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>Mini React Reconciler</title>
</head>
<body>
  <div id="app"></div>
  <script src="./reconciler.js"></script>
  <script src="./main.js"></script>
</body>
</html>
```

---

### ✅ `reconciler.js`

```js
function createElement(type, props, ...children) {
  return {
    type,
    props: props || {},
    children: children.flat().map(child =>
      typeof child === 'object' ? child : createTextElement(child)
    )
  };
}

function createTextElement(text) {
  return {
    type: 'TEXT_ELEMENT',
    props: { nodeValue: text },
    children: []
  };
}

function render(vNode, container) {
  const dom =
    vNode.type === 'TEXT_ELEMENT'
      ? document.createTextNode('')
      : document.createElement(vNode.type);

  Object.entries(vNode.props || {}).forEach(([key, value]) => {
    if (key !== 'nodeValue') dom[key] = value;
  });

  if (vNode.type === 'TEXT_ELEMENT') {
    dom.nodeValue = vNode.props.nodeValue;
  }

  vNode.children.forEach(child => render(child, dom));
  container.appendChild(dom);
}

function diff(prev, next, container) {
  if (!prev) {
    render(next, container);
  } else if (!next) {
    container.removeChild(container.firstChild);
  } else if (prev.type !== next.type) {
    container.replaceChild(createDOM(next), container.firstChild);
  } else {
    const dom = container.firstChild;
    Object.keys(next.props).forEach(name => {
      if (name !== 'nodeValue') dom[name] = next.props[name];
    });
    if (next.type === 'TEXT_ELEMENT') {
      dom.nodeValue = next.props.nodeValue;
    }
    for (let i = 0; i < next.children.length; i++) {
      diff(prev.children[i], next.children[i], dom);
    }
  }
}
```

---

### ✅ `main.js`

```js
const v1 = createElement('div', { id: 'root' },
  createElement('h1', null, 'Hello'),
  createElement('p', null, 'Welcome to Mini React')
);

const v2 = createElement('div', { id: 'root' },
  createElement('h1', null, 'Hello World!'),
  createElement('p', null, 'Updated paragraph')
);

const container = document.getElementById('app');
render(v1, container);

// Simulate update
setTimeout(() => {
  container.innerHTML = ''; // Clear old
  render(v2, container);    // New virtual DOM
}, 2000);
```

---

## ✅ Next Step: Push to GitHub

1. Create a repo:

   ```sh
   git init
   git add .
   git commit -m "Mini React-like reconciler"
   git remote add origin <your-repo-url>
   git push -u origin main
   ```

2. Add a `README.md` explaining:

   * What this is
   * How to run
   * How reconciliation works

Here are some **interview questions** (basic to advanced) focused on **React Reconciliation and Fiber architecture**, often asked in frontend engineer or React developer interviews:

---

## ✅ **Basic Level**

1. **What is reconciliation in React?**
2. **What is the Virtual DOM and how does it relate to reconciliation?**
3. **Why is reconciliation important for performance in React?**
4. **What is the time complexity of React’s diffing algorithm?**
5. **What role do keys play in React’s reconciliation process?**

---

## ✅ **Intermediate Level**

6. **How does React decide whether to re-render a component?**
7. **What is the difference between the real DOM and virtual DOM?**
8. **What happens during the commit phase of reconciliation?**
9. **Can you explain double buffering in Fiber?**
10. **What are the phases of reconciliation in Fiber?**
11. **How does React 16+ differ from React 15 in terms of rendering?**
12. **What’s the difference between synchronous and asynchronous rendering?**

---

## ✅ **Advanced Level (Fiber & Concurrent Rendering)**

13. **What is the Fiber architecture in React?**
14. **How does Fiber improve rendering performance?**
15. **How does React use linked lists instead of recursive stacks in Fiber?**
16. **What are units of work in Fiber?**
17. **How does Fiber enable time slicing and prioritization?**
18. **What’s the difference between legacy mode and concurrent mode in React 18?**
19. **How does `startTransition()` relate to reconciliation?**
20. **What happens when React pauses rendering in the middle of reconciliation?**
21. **How does Suspense work with the reconciliation process?**
22. **What’s the role of the Scheduler package in React 18’s reconciliation?**

---

## ✅ Bonus Practical Questions

23. **How would you implement a basic virtual DOM diffing algorithm?**
24. **How does improper use of keys affect reconciliation?**
25. **How would you debug a reconciliation-related performance issue in React?**

Great! Below are **detailed answers** to the **top React Reconciliation & Fiber architecture interview questions**:

---

## ✅ Basic Level

### 1. **What is reconciliation in React?**

Reconciliation is the process React uses to update the DOM. It compares the new virtual DOM with the previous one and calculates the minimum number of changes needed to update the real DOM.

---

### 2. **What is the Virtual DOM and how does it relate to reconciliation?**

The Virtual DOM is a lightweight in-memory representation of the actual DOM. During reconciliation, React compares the new virtual DOM with the old one to figure out what changed and updates only those parts in the real DOM.

---

### 3. **Why is reconciliation important for performance in React?**

It avoids full DOM re-renders. React selectively updates only what’s changed, reducing expensive DOM operations and improving rendering performance.

---

### 4. **What is the time complexity of React’s diffing algorithm?**

React optimizes its diffing algorithm to O(n) by making two key assumptions:

* Elements of different types produce different trees.
* Elements with keys are stable between renders and can be reused.

---

### 5. **What role do keys play in React’s reconciliation process?**

Keys help React identify which items in a list changed, were added, or removed. Using stable keys prevents unnecessary re-renders and ensures correct element matching during diffing.

---

## ✅ Intermediate Level

### 6. **How does React decide whether to re-render a component?**

If props or state change, React re-renders the component. It then uses reconciliation to determine if DOM updates are necessary based on changes in the virtual DOM.

---

### 7. **What is the difference between the real DOM and virtual DOM?**

* Real DOM is the actual browser-rendered UI.
* Virtual DOM is a JS object representing the UI.
  React updates the virtual DOM first, then updates the real DOM only where needed.

---

### 8. **What happens during the commit phase of reconciliation?**

In the commit phase, React applies all changes calculated during the render phase to the real DOM. This includes DOM updates, lifecycle methods, and effect hooks.

---

### 9. **Can you explain double buffering in Fiber?**

Double buffering means React maintains two trees:

* The **current tree** shown to users.
* The **work-in-progress tree** being built.
  After reconciliation, the work-in-progress tree becomes the current tree.

---

### 10. **What are the phases of reconciliation in Fiber?**

* **Begin Work:** Build the new fiber tree top-down.
* **Complete Work:** Bottom-up phase to finalize effects.
* **Commit Phase:** Apply changes to the DOM.

---

### 11. **How does React 16+ differ from React 15 in terms of rendering?**

React 15 used a recursive stack reconciler — synchronous and non-interruptible. React 16 introduced Fiber, which is iterative, supports async rendering, and enables features like error boundaries and Suspense.

---

### 12. **What’s the difference between synchronous and asynchronous rendering?**

* **Synchronous:** All work must finish in one go. Blocks UI.
* **Asynchronous (Fiber):** Work can be paused and resumed. Keeps UI responsive.

---

## ✅ Advanced Level

### 13. **What is the Fiber architecture in React?**

Fiber is a reimplementation of the React core algorithm. It uses linked fiber nodes instead of a recursive tree and breaks rendering into units of work to support scheduling and prioritization.

---

### 14. **How does Fiber improve rendering performance?**

It allows React to:

* Pause rendering
* Prioritize high-importance updates
* Resume or cancel in-progress work
* Spread rendering over multiple frames

---

### 15. **How does React use linked lists in Fiber?**

Fiber nodes are linked via `child`, `sibling`, and `return` pointers. This allows React to traverse and manipulate nodes one-by-one in an iterative, non-blocking way.

---

### 16. **What are units of work in Fiber?**

Each fiber node is a unit of work (e.g., rendering a component). React processes these one at a time and yields back control if needed, to avoid UI jank.

---

### 17. **How does Fiber enable time slicing and prioritization?**

React checks the available time during each unit of work. If time runs out, it yields back and resumes later. Updates can be marked with priorities, so urgent updates render before less important ones.

---

### 18. **What’s the difference between legacy mode and concurrent mode in React 18?**

* **Legacy Mode:** React renders everything synchronously.
* **Concurrent Mode:** React renders using time slicing and can interrupt rendering for high-priority updates.

---

### 19. **How does `startTransition()` relate to reconciliation?**

`startTransition()` marks updates as low-priority. This allows React to delay these updates if more urgent updates (like input or clicks) happen, improving responsiveness.

---

### 20. **What happens when React pauses rendering in the middle of reconciliation?**

React saves the current fiber state. It can later resume from that fiber node. This prevents blocking the main thread and keeps animations and input smooth.

---

### 21. **How does Suspense work with the reconciliation process?**

Suspense lets React delay rendering a subtree until data is ready. During reconciliation, React can “suspend” a component and later retry rendering when the promise resolves.

---

### 22. **What’s the role of the Scheduler package in React 18’s reconciliation?**

It decides when and how updates are run based on available time, priority, and deadline. It powers the time-slicing behavior of the concurrent renderer.


