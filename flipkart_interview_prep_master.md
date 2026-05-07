# 🚀 Flipkart Interview Master Prep — ThinkifyLabs
### Frontend Engineer (JavaScript + React + System Design)

> **How this doc is organized:**
> - **Part 1 — ⭐ MOST IMPORTANT (LATEST):** Questions from the most recent interview transcripts. These are the freshest, most likely-to-be-asked questions.
> - **Part 2 — Deep Reference Pool:** Additional questions consolidated from multiple Thinkify prep documents (deduplicated).
>
> Every question has a scenario-based answer + tricky-situation handling.

---

## 📋 Master Table of Contents

### Part 1 — ⭐ MOST IMPORTANT (Latest Questions)
1. [JavaScript Core (HOT)](#1-javascript-core-hot)
2. [React Core (HOT)](#2-react-core-hot)
3. [Rendering & Architecture (HOT)](#3-rendering--architecture-hot)
4. [State Management — Redux (HOT)](#4-state-management--redux-hot)
5. [System Design (HOT)](#5-system-design-hot)
6. [Coding Questions w/ Solutions (HOT)](#6-coding-questions-with-solutions-hot)
7. [Project & Behavioral (HOT)](#7-project--behavioral-hot)

### Part 2 — Deep Reference Pool
8. [HTML / CSS Deep Dive](#8-html--css-deep-dive)
9. [JavaScript Advanced Topics](#9-javascript-advanced-topics)
10. [React Advanced Topics](#10-react-advanced-topics)
11. [Browser, Storage & Security](#11-browser-storage--security)
12. [Machine Coding Problems Bank](#12-machine-coding-problems-bank)
13. [DSA Problems for Frontend](#13-dsa-problems-for-frontend)
14. [Hiring Manager Round Playbook](#14-hiring-manager-round-playbook)

---
---

# Part 1 — ⭐ MOST IMPORTANT (Latest Questions)

---

## 1. JavaScript Core (HOT)

### ⭐ Closures — what & why?

A closure is a function that **remembers variables from its outer scope** even after the outer function has finished executing.

**Bank vault analogy:** The bank vault (outer function) closes at end of day, but you (inner function) still carry a key (reference to the variable).

```javascript
function parent() {
  let count = 0;
  return function child() {
    return (count += 1);
  };
}
const increment = parent();
increment(); // 1
increment(); // 2
```

**Why it works:** `child` holds a reference to `count`, so the JS engine doesn't garbage-collect it.

**Real uses:** Data privacy, memoization, event handlers, factory functions, currying.

**🔥 Tricky follow-up — Closure inside a loop with `var`:**
```javascript
for (var i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), i * 1000);
}
// Output: 4, 4, 4 — because `var` is function-scoped, all callbacks reference the same `i`

// Fix 1: use `let` (block-scoped, fresh `i` per iteration)
for (let i = 1; i <= 3; i++) {
  setTimeout(() => console.log(i), i * 1000); // 1, 2, 3
}

// Fix 2: IIFE that captures current `i` in its own scope
for (var i = 1; i <= 3; i++) {
  (function(idx) {
    setTimeout(() => console.log(idx), i * 1000);
  })(i);
}
```

**If asked "why does `var` fail here?":** Say *"`setTimeout` runs after the loop finishes. By then, `i` is 4. Each callback closes over the same single `i` — there's only one `i` in the function scope. With `let`, each iteration creates a new block-scoped binding."*

---

### ⭐ Shallow Copy vs Deep Copy

| | Shallow Copy | Deep Copy |
|---|---|---|
| Copies | Top-level only | All nested levels |
| Nested refs | Shared | Independent |
| Methods | `{...obj}`, `Object.assign()` | `structuredClone()`, `JSON.parse(JSON.stringify())`, Lodash `_.cloneDeep()` |

```javascript
const original = { name: 'Ravi', address: { city: 'Bangalore' } };

const shallow = { ...original };
shallow.address.city = 'Mumbai';
console.log(original.address.city); // 'Mumbai' — MUTATED ⚠️

const deep = structuredClone(original);
deep.address.city = 'Delhi';
console.log(original.address.city); // 'Mumbai' (unchanged from shallow line)
```

**🔥 Tricky:** Why not always `JSON.parse(JSON.stringify())`?
- Loses functions, `undefined`, `Date` objects (becomes string), `Map`/`Set`, circular refs throw.
- Modern answer: **`structuredClone()`** — handles circular refs, but still doesn't clone functions/class methods.

---

### ⭐ Promise States & async/await

**3 states:** Pending → (Fulfilled | Rejected). Once settled, **cannot change.**

```javascript
async function fetchUser() {
  try {
    const res = await fetch('/api/user');
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

**Key rules:**
- `async` functions **always return a Promise**.
- `await` only pauses the current async function (JS thread is non-blocking).
- `Promise.resolve().then(cb)` queues `cb` as a **microtask**.

---

### ⭐ Event Loop, Microtasks vs Macrotasks

```
Call Stack drains (sync code) →
  Microtask Queue drains FULLY (Promises, await continuations) →
    ONE Macrotask runs (setTimeout, setInterval, I/O) →
      Repeat
```

| Microtasks | Macrotasks |
|---|---|
| `Promise.then`, `await`, `queueMicrotask`, `MutationObserver` | `setTimeout`, `setInterval`, DOM events, I/O |

**🔥 Output question (asked at every interview):**
```javascript
console.log('1');
setTimeout(() => console.log('2'), 0);
Promise.resolve().then(() => console.log('3'));
async function transition() {
  console.log('4');
  await Promise.resolve();
  console.log('5');
}
transition();
console.log('6');
```
**Output: `1, 4, 6, 3, 5, 2`**

| Step | Why |
|---|---|
| `1` | Sync log |
| `setTimeout` | Goes to macrotask queue |
| `Promise.then` | Goes to microtask queue |
| `4` | Sync log inside `transition()` |
| `await` | Pauses transition, schedules continuation as **microtask** |
| `6` | Sync log |
| `3` | First microtask drained |
| `5` | Second microtask drained (transition resumes) |
| `2` | Macrotask runs last |

---

### ⭐ let vs const vs var

| | `var` | `let` | `const` |
|---|---|---|---|
| Scope | Function/global | Block | Block |
| Re-declared in same scope | ✅ | ❌ | ❌ |
| Re-assigned | ✅ | ✅ | ❌ |
| Hoisted | ✅ (initialized as `undefined`) | ✅ (in TDZ) | ✅ (in TDZ) |
| Initialization required | ❌ | ❌ | ✅ |

**🔥 Tricky — Temporal Dead Zone (TDZ):**
```javascript
function demo() {
  console.log(a); // undefined  (var is hoisted with undefined)
  console.log(b); // ReferenceError  (TDZ)
  var a = 1;
  let b = 2;
}
```

**`const` does NOT make objects immutable:**
```javascript
const obj = { name: 'A' };
obj.name = 'B'; // ✅ Allowed
obj = {};       // ❌ TypeError
```

---

### ⭐ Hoisting

JavaScript moves **declarations** (not initializations) to the top of their scope.

```javascript
console.log(a); // undefined  ← `var a` hoisted, but `=10` not yet executed
var a = 10;

console.log(b); // ReferenceError ← `let` is in TDZ
let b = 20;

display(); // works! function declarations are fully hoisted
function display() { console.log('hi'); }

func3(); // TypeError: func3 is not a function
var func3 = function() { console.log(3); }; // function expressions: only var is hoisted, not the function
```

**🔥 Tricky output:**
```javascript
var a = 1;
function func() {
  a = 2;          // local `a` declared via hoisting (undefined initially)
  console.log(a); // 2
  var a;          // hoisted to top of func scope
}
func();
console.log(a);   // 1 (outer `a` untouched)
```

---

### ⭐ this — what does it refer to?

**`this` is determined by HOW a function is called, not where it's defined:**

| Call type | `this` is |
|---|---|
| `obj.method()` | `obj` |
| Standalone `func()` | `undefined` (strict) / `window` (non-strict) |
| Arrow function | Inherits from enclosing scope (lexical) |
| `func.call(ctx)` / `apply` / `bind` | Whatever you pass |
| `new Foo()` | The new instance |
| Event handler `el.onclick = fn` | The element |

```javascript
const a = { foo: 'bar' };
function hello() { console.log(this.foo); }

hello();          // undefined / 'baz' (depends on global)
hello.call(a);    // 'bar'
hello.bind(a)();  // 'bar'
```

**🔥 Tricky — Arrow functions don't have their own `this`:**
```javascript
const obj = {
  name: 'Flipkart',
  greet: function() {
    setTimeout(() => console.log(this.name), 100); // 'Flipkart'
    setTimeout(function() { console.log(this.name); }, 100); // undefined
  }
};
```

---

### ⭐ call, apply, bind

All three set the `this` value when calling a function.

| | Arguments | When called |
|---|---|---|
| `call` | Comma-separated | Immediately |
| `apply` | Array | Immediately |
| `bind` | Comma-separated | Returns new function (call later) |

```javascript
function greet(greeting, punct) { return `${greeting}, ${this.name}${punct}`; }
const user = { name: 'Ravi' };

greet.call(user, 'Hi', '!');     // 'Hi, Ravi!'
greet.apply(user, ['Hi', '!']);  // 'Hi, Ravi!'
const bound = greet.bind(user, 'Hi');
bound('!');                       // 'Hi, Ravi!'
```

**🔥 Polyfill for `bind` (very common):**
```javascript
Function.prototype.myBind = function (context, ...args) {
  const fn = this;
  if (typeof fn !== 'function') {
    throw new TypeError('myBind - what is being bound is not callable');
  }
  return function (...args2) {
    return fn.apply(context, [...args, ...args2]);
  };
};

// Test
function show(city, country) { return `${this.name} from ${city}, ${country}`; }
const user2 = { name: 'Anu' };
const bound2 = show.myBind(user2, 'Mumbai');
console.log(bound2('India')); // 'Anu from Mumbai, India'
```

**Concepts the interviewer is checking:** Closure (over `context` and `args`), `apply`, rest/spread, prototype.

---

### ⭐ Difference: Normal Function vs Arrow Function

| | Normal Function | Arrow Function |
|---|---|---|
| `this` binding | Dynamic (caller decides) | Lexical (parent scope) |
| `arguments` object | ✅ Has it | ❌ Use `...args` instead |
| Can be used as constructor (`new`) | ✅ | ❌ |
| Has `prototype` | ✅ | ❌ |
| Hoisted | ✅ (declarations only) | ❌ (assigned to var/let/const) |

**Tricky use case:** In React, arrow functions in class methods are common because they auto-bind `this` to the instance.

---

## 2. React Core (HOT)

### ⭐ Class vs Functional Components

| | Class | Functional |
|---|---|---|
| Syntax | ES6 class | Plain function |
| State | `this.state` + `setState` | `useState` |
| Lifecycle | Lifecycle methods | `useEffect` + others |
| `this` keyword | Required, tricky | Not used |
| Logic reuse | HOC, render props | **Custom hooks** |
| Modern React | Legacy (still supported) | **Recommended** |

---

### ⭐ Lifecycle Methods → Hooks Mapping

```
MOUNT          UPDATE                        UNMOUNT
constructor    getDerivedStateFromProps      componentWillUnmount
render         shouldComponentUpdate
componentDidMount  render
                  getSnapshotBeforeUpdate
                  componentDidUpdate
```

**Mimicking with hooks:**
```javascript
// componentDidMount
useEffect(() => { console.log('mounted'); }, []);

// componentDidUpdate (specific dep)
useEffect(() => { console.log('userId changed'); }, [userId]);

// componentWillUnmount
useEffect(() => {
  return () => console.log('unmounting');
}, []);

// Run on every render (rare)
useEffect(() => { console.log('always'); });
```

---

### ⭐ useEffect vs useLayoutEffect

| | `useEffect` | `useLayoutEffect` |
|---|---|---|
| Timing | After browser paint | Before paint, sync after DOM mutation |
| Blocking | Async (non-blocking) | Sync (blocks paint) |
| Use case | API calls, subscriptions | DOM measurements, prevent flicker |
| SSR | ✅ Safe | ⚠️ Warning on server |

**Default to `useEffect`. Reach for `useLayoutEffect` only when fixing visual flicker.**

---

### ⭐ React Hooks Output Question

```javascript
const ComponentA = () => {
  const [count, setCount] = useState(0);
  console.log('1: Render Start');
  useEffect(() => { console.log('2: useEffect'); }, []);
  useLayoutEffect(() => { console.log('3: useLayoutEffect'); }, []);
  return (
    <div>
      {console.log('4: Render Body')}
      Check Console
    </div>
  );
};
```
**Output: `1, 4, 3, 2`**
- `1` and `4` during render phase
- `3` synchronously after DOM mutation (before paint)
- `2` async after paint

---

### ⭐ Parent/Child useEffect Order

> Parent has `console.log('parent')`, child1 has `console.log('child1')`, grandchild has `console.log('child2')`.
> **Output: `child2, child1, parent`**

**Why:** React commits children before parents. `useEffect` fires after commit, **bottom-up** (deepest first). Child must finish mounting before parent's "post-mount" effect fires.

---

### ⭐ useMemo vs useCallback vs React.memo

| Hook | Memoizes | Use when |
|---|---|---|
| `useMemo` | A computed **value** | Expensive computation (filter/sort large list) |
| `useCallback` | A **function reference** | Passing callbacks to memoized children |
| `React.memo` | A whole **component** | Component re-renders too often with same props |

```javascript
// Expensive computation
const sortedList = useMemo(() => hugeArray.sort(), [hugeArray]);

// Stable callback ref
const handleClick = useCallback(() => doSomething(id), [id]);

// Memoize component
const Row = React.memo(({ item }) => <div>{item.name}</div>);
```

**🔥 Common misconception:** `useMemo`/`useCallback` are NOT free. They have overhead. Only use when you've **measured** a re-render problem.

---

### ⭐ Custom Hook: useDebounce

```javascript
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  useEffect(() => {
    const timer = setTimeout(() => setDebouncedValue(value), delay);
    return () => clearTimeout(timer);
  }, [value, delay]);
  return debouncedValue;
}

// Usage
function SearchApp() {
  const [searchTerm, setSearchTerm] = useState('');
  const debounced = useDebounce(searchTerm, 300);
  useEffect(() => {
    if (debounced) console.log('API Call:', debounced);
  }, [debounced]);
  return <input onChange={e => setSearchTerm(e.target.value)} />;
}
```

**🔥 Tricky follow-up: Debounce vs Throttle?**
- **Debounce**: Wait until user STOPS for N ms, then fire. (Search box.)
- **Throttle**: Fire at most once every N ms, regardless. (Scroll handlers, window resize.)

```javascript
// Throttle implementation
function throttle(fn, delay) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= delay) {
      lastCall = now;
      fn.apply(this, args);
    }
  };
}
```

---

### ⭐ Higher-Order Components (HOC)

A function that takes a component and returns an enhanced component.

```javascript
function withAuth(WrappedComponent) {
  return function AuthGuard(props) {
    const { isLoggedIn, user } = useAuth();
    if (!isLoggedIn) return <Navigate to="/login" />;
    return <WrappedComponent {...props} user={user} />;
  };
}

const ProtectedDashboard = withAuth(Dashboard);
```

**HOC vs Custom Hook:**
- HOC wraps a component (good for cross-cutting like auth, error boundaries).
- Custom Hook shares stateful logic without the wrapper (preferred in modern React).

---

### ⭐ Pure Components & React.memo

A pure component re-renders **only when its props/state change** (shallow compare).

- **Class:** Extend `React.PureComponent`.
- **Functional:** Wrap with `React.memo()`.

```javascript
const Row = React.memo(({ item }) => <div>{item.name}</div>);
```

**🔥 Trap:** `React.memo` does shallow compare. If you pass a new function or object every render, it won't help. Combine with `useCallback`/`useMemo`.

---

## 3. Rendering & Architecture (HOT)

### ⭐ CSR vs SSR

| | CSR | SSR |
|---|---|---|
| Initial HTML | Empty shell | Pre-rendered |
| First paint | Slow (waits for JS) | Fast |
| SEO | Poor (without SSG) | Excellent |
| Server load | Low | High |
| TTI | Fast after JS | Slower (needs hydration) |
| Use case | Dashboards, apps behind login | E-commerce, marketing, blogs |
| Examples | CRA, Vite + React | Next.js, Remix |

**🔥 Bonus — ISR (Next.js):** Pre-render at build time + revalidate on a schedule. Best of both.

---

### ⭐ Hydration

After SSR, React **attaches event listeners** to existing server-rendered HTML to make it interactive.

**Analogy:** Server sends a paper car (looks right, doesn't move). Hydration installs the engine.

**Hydration mismatch:** When server HTML ≠ client HTML, React throws an error. Common causes: `Date.now()`, `Math.random()`, `window` usage during render.

---

### ⭐ Virtual DOM & Reconciliation

**Virtual DOM:** A JS object representation of the real DOM. Diffing happens here, not on the slow real DOM.

**Reconciliation flow:**
1. State changes → React re-renders the component tree (virtually).
2. **Diffing algorithm** compares new VDOM with old VDOM (O(n) using heuristics).
3. Computes minimal patches.
4. Applies them to the real DOM in a batch.

**Why fast:** Real DOM ops are expensive (layout, paint). VDOM ops are pure JS.

**🔥 Diffing rules:**
- Different element types → unmount entire subtree, mount new.
- Same type → only update changed attributes.
- Lists → use `key` to identify which item moved/added/removed.

---

### ⭐ What happens when you type `flipkart.com` and hit enter?

1. **DNS lookup** — Browser checks cache → OS cache → router → ISP DNS → root → TLD → authoritative. Returns IP.
2. **TCP handshake** — 3-way handshake (SYN, SYN-ACK, ACK).
3. **TLS handshake** — for HTTPS, exchange certs and session keys.
4. **HTTP request** — Browser sends GET / with headers (cookies, user-agent).
5. **Server processes** — load balancer → app server → DB → response.
6. **HTML received** — Browser starts parsing.
7. **Critical render path:**
   - HTML → DOM tree
   - CSS → CSSOM
   - DOM + CSSOM → **Render Tree**
   - Layout (compute geometry)
   - Paint (fill pixels)
   - Composite (layers)
8. **JS execution** — Blocks parser unless `async` / `defer`.
9. **Subsequent requests** — Images, fonts, API calls, etc.

---

### ⭐ Performance Optimization Techniques

**Architecture:**
- Use SSR/ISR for SEO-heavy pages (Next.js).
- Code splitting — `React.lazy(() => import('./Heavy'))` + `<Suspense>`.
- CDN for static assets.

**Code:**
- Minimize re-renders → `React.memo`, `useMemo`, `useCallback`.
- Tree shaking — `import _sortBy from 'lodash/sortBy'` not the whole `lodash`.
- Image optimization — WebP, responsive `srcSet`, lazy load with `loading="lazy"`.
- Debounce/throttle expensive handlers.
- Virtualize long lists (`react-window`).
- Web Workers for CPU-heavy computation off the main thread.

**Network:**
- HTTP/2 or HTTP/3, gzip/brotli compression.
- Cache headers, Service Workers (PWA), prefetch/preload critical resources.

**Measurement:** Lighthouse, Web Vitals (LCP, FID, CLS, INP), webpack-bundle-analyzer.

---

## 4. State Management — Redux (HOT)

### ⭐ Why Redux?

Solves **prop drilling** + provides a **single source of truth** for global state with predictable transitions.

**Use Redux when:**
- Many unrelated components share state.
- Complex state transitions need traceability (Redux DevTools).
- Multi-team frontend with clear ownership boundaries.

**Don't use Redux when:**
- Local form state (useState is fine).
- Server state (use React Query / SWR).

---

### ⭐ Core Concepts

```
Action → Reducer → Store → Component (via useSelector)
```

- **Store:** Single object, holds entire state.
- **Action:** `{ type: 'cart/addItem', payload: {...} }`
- **Reducer:** Pure function `(state, action) => newState`. Never mutates.
- **Dispatch:** Sends action to store.

```javascript
function cartReducer(state = [], action) {
  switch (action.type) {
    case 'cart/addItem': return [...state, action.payload];
    case 'cart/removeItem': return state.filter(i => i.id !== action.payload.id);
    default: return state;
  }
}
```

---

### ⭐ Why Immutable Updates?

Redux uses `===` shallow equality to determine if state changed. Mutating in place keeps the same reference → React/Redux thinks nothing changed → no re-render.

```javascript
// ❌ Mutation — won't trigger re-render
state.items.push(newItem);
return state;

// ✅ New reference
return { ...state, items: [...state.items, newItem] };
```

---

### ⭐ Redux Toolkit (RTK) — Modern Redux

```javascript
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

export const fetchUser = createAsyncThunk('user/fetch', async (id) => {
  const res = await fetch(`/api/users/${id}`);
  return res.json();
});

const userSlice = createSlice({
  name: 'user',
  initialState: { data: null, loading: false, error: null },
  reducers: {
    logout: (state) => { state.data = null; }, // Immer lets us "mutate" safely
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUser.pending,   (s)    => { s.loading = true; })
      .addCase(fetchUser.fulfilled, (s, a) => { s.loading = false; s.data = a.payload; })
      .addCase(fetchUser.rejected,  (s, a) => { s.loading = false; s.error = a.error.message; });
  },
});
```

---

## 5. System Design (HOT)

### ⭐ Redis — What & Why

In-memory key-value store. Lightning fast (RAM) with optional disk persistence.

**Use cases:**
- **Caching** — `SET user:123 "{...}" EX 300`
- **Sessions** — Centralized so multiple servers can read.
- **Rate limiting** — `INCR requests:ip:1.2.3.4 EX 60`
- **Pub/Sub** — Real-time messaging between microservices.
- **Leaderboards** — Sorted sets `ZADD`.
- **Job queues** — BullMQ uses Redis.

---

### ⭐ SQL vs MongoDB

| | SQL (Postgres, MySQL) | MongoDB |
|---|---|---|
| Schema | Strict | Flexible |
| Joins | Native | `$lookup` (slow) or embed |
| ACID | Full | Doc-level |
| Scaling | Vertical | Horizontal |
| Best for | Transactions, financial | Catalogs, profiles, evolving schemas |

---

### ⭐ WebSockets

**Persistent, bidirectional** TCP connection. Both sides can push at any time.

**Use cases:** Chat, live scores, collaborative editing, stock tickers, multiplayer games.

**vs HTTP polling:** WS has lower overhead and lower latency for real-time updates.

---

### ⭐ Load Balancer

Distributes incoming traffic across multiple servers.

**Algorithms:** Round Robin, Least Connections, IP Hash (sticky).

**Stateful vs Stateless:**
- **Stateless** servers (REST APIs with JWT) — easy to scale, any server can serve any request.
- **Stateful** services (WebSockets, in-memory sessions) — need sticky sessions OR shared store.

**Sticky sessions problem:** If Server 1 dies, all its users lose state.
**Better solution:** Store session in **Redis** — any server can serve any request.

---

## 6. Coding Questions with Solutions (HOT)

### ⭐ Q1: Event Loop Output (covered in §1)

### ⭐ Q2: Implement Lifecycle Methods in Functional Component

```javascript
function LifecycleDemo({ userId }) {
  const [data, setData] = useState(null);
  const prevUserIdRef = useRef(userId);

  // componentDidMount + componentWillUnmount
  useEffect(() => {
    console.log('MOUNTED');
    return () => console.log('UNMOUNTED');
  }, []);

  // componentDidUpdate (specific dep)
  useEffect(() => {
    if (prevUserIdRef.current !== userId) {
      console.log(`changed from ${prevUserIdRef.current} → ${userId}`);
      prevUserIdRef.current = userId;
    }
  }, [userId]);

  return <div>{data?.name}</div>;
}
export default React.memo(LifecycleDemo); // shouldComponentUpdate
```

### ⭐ Q3: useEffect / useLayoutEffect Output (covered in §2)

### ⭐ Q4: useDebounce Custom Hook (covered in §2)

### ⭐ Q5: Longest Subarray with K Distinct Characters (Sliding Window)

```javascript
function longestSubArr(arr, k) {
  let left = 0, maxLen = 0;
  const charCount = new Map();
  for (let right = 0; right < arr.length; right++) {
    charCount.set(arr[right], (charCount.get(arr[right]) || 0) + 1);
    while (charCount.size > k) {
      const c = arr[left];
      charCount.set(c, charCount.get(c) - 1);
      if (charCount.get(c) === 0) charCount.delete(c);
      left++;
    }
    maxLen = Math.max(maxLen, right - left + 1);
  }
  return maxLen;
}
console.log(longestSubArr(['a','b','a','c','c','b','a','d'], 3)); // 7
```
**Time:** O(n) | **Space:** O(k)

---

## 7. Project & Behavioral (HOT)

### ⭐ Multi-Step Form (Full Solution)

```javascript
const STEPS = [
  { id: 1, title: 'Personal', fields: ['name', 'email'] },
  { id: 2, title: 'Address',  fields: ['city', 'pincode'] },
  { id: 3, title: 'Review',   fields: [] },
];

function MultiStepForm() {
  const [step, setStep] = useState(0);
  const [data, setData] = useState({});
  const [errors, setErrors] = useState({});

  const update = (field, val) => setData(d => ({ ...d, [field]: val }));

  const validate = (s) => {
    const e = {};
    if (s === 0) {
      if (!data.name) e.name = 'Required';
      if (!/\S+@\S+\.\S+/.test(data.email || '')) e.email = 'Invalid';
    }
    if (s === 1) {
      if (!/^\d{6}$/.test(data.pincode || '')) e.pincode = '6 digits';
    }
    return e;
  };

  const next = () => {
    const e = validate(step);
    if (Object.keys(e).length) return setErrors(e);
    setErrors({});
    setStep(s => s + 1);
  };

  const submit = async () => {
    try { await api.submit(data); } catch { alert('Failed — try again'); }
  };

  return (
    <div>
      <ProgressBar current={step} total={STEPS.length} />
      <StepView step={step} data={data} errors={errors} onChange={update} />
      <button disabled={step === 0} onClick={() => setStep(s => s - 1)}>Back</button>
      <button onClick={step === STEPS.length - 1 ? submit : next}>
        {step === STEPS.length - 1 ? 'Submit' : 'Continue'}
      </button>
    </div>
  );
}
```

**Common follow-ups:**
- **State management?** Local `useState` for small forms, `useReducer` for complex, React Hook Form for large with many fields.
- **Persist on refresh?** `useEffect` syncing to `sessionStorage`.
- **API failure?** Don't lose user data — show toast, allow retry.
- **Re-render optimization?** Split into per-field memoized children.
- **Config-driven?** Define schema as JSON, render generically.

---

### ⭐ Code Review Question (Common Trap)

```javascript
const Blog = (props) => {
  const sidebar = (
    <ul>
      {props.posts.map((post, index) =>
        <li key={index}>{post.title}</li>   {/* ❌ key={index} */}
      )}
    </ul>
  );
  // ...
};
```

**Issues to call out:**
1. **`key={index}`** — Anti-pattern. Use stable IDs (`post.id`). Index keys cause bugs when items are reordered/inserted.
2. **No PropTypes / TypeScript types.**
3. **`Blog`** is missing `React.` import in real code.
4. **No error/empty state** for `posts === undefined || posts.length === 0`.
5. **No memoization** — re-renders on every parent render.
6. **No accessibility** — sidebar is `<ul>` but should have `<nav aria-label="...">`.

---

---

# Part 2 — Deep Reference Pool

## 8. HTML / CSS Deep Dive

### CSS Box Model

Every element = **content + padding + border + margin**.

```
┌──────────── margin ────────────┐
│   ┌──────── border ───────┐   │
│   │   ┌── padding ──┐    │   │
│   │   │  content    │    │   │
│   │   └─────────────┘    │   │
│   └───────────────────────┘   │
└────────────────────────────────┘
```

**Total width by default (`box-sizing: content-box`):**
`width + padding + border + margin`

**With `box-sizing: border-box` (preferred):**
`width` includes padding + border. Easier to reason about.

```css
* { box-sizing: border-box; }
```

**🔥 Tricky question:** "If padding=10, margin=10, border=10, height=100, what's total height?"
- `content-box`: `100 + 2*10 + 2*10 + 2*10 = 160px` (margin contributes to outer space, not box height visually)
- Actual rendered box height (excluding margin): `100 + 20 + 20 = 140px`

---

### Block vs Inline vs Inline-Block

| | Block | Inline | Inline-Block |
|---|---|---|---|
| New line | ✅ Forces | ❌ Stays in flow | ❌ Stays in flow |
| Width/Height | Respected | ❌ Ignored | ✅ Respected |
| Margin/Padding (vertical) | ✅ | ❌ Top/Bottom ignored | ✅ |
| Examples | `<div>`, `<p>`, `<h1>` | `<span>`, `<a>`, `<em>` | `<img>`, `<button>`, `<input>` |

**Practical:** Want elements side-by-side but with width/height? → `inline-block` or use Flexbox/Grid.

---

### CSS Positions (must-know)

| Position | Removed from flow? | Positioned relative to |
|---|---|---|
| `static` (default) | ❌ | Normal flow |
| `relative` | ❌ (still takes space) | Its original position |
| `absolute` | ✅ | Nearest positioned ancestor |
| `fixed` | ✅ | Viewport |
| `sticky` | ❌ until threshold, ✅ after | Nearest scrolling ancestor |

**🔥 Trick — Center a div with absolute:**
```css
.parent { position: relative; }
.child {
  position: absolute;
  top: 50%; left: 50%;
  transform: translate(-50%, -50%);
}
```

**Or with Flexbox (cleaner):**
```css
.parent { display: flex; justify-content: center; align-items: center; }
```

---

### CSS Specificity

Order from highest to lowest:
1. `!important` (avoid in production)
2. Inline styles (`style="..."`) — score `1000`
3. ID selectors `#id` — score `100`
4. Class, attribute, pseudo-class `.class`, `[attr]`, `:hover` — score `10`
5. Element, pseudo-element `div`, `::before` — score `1`

```
#nav .item.active a:hover  →  100 + 10 + 10 + 1 + 10 = 131
```

**Rule:** Higher specificity wins. If equal, **last declared** wins.

---

### Reset CSS vs Normalize CSS

| | Reset | Normalize |
|---|---|---|
| Strategy | Removes all default styling | Preserves useful defaults, fixes inconsistencies |
| Result | Blank slate | Consistent baseline |
| Use case | Full design system control | Web apps with quick consistent look |
| Famous | Eric Meyer's reset | normalize.css |

---

### SCSS vs CSS, and PostCSS

**SCSS adds:** Variables, nesting, mixins, partials, imports, functions.

```scss
$primary: #2874f0;
.button {
  color: $primary;
  &:hover { color: darken($primary, 10%); }
}
```

Compiled to CSS via tools (Webpack `sass-loader`, Gulp).

**PostCSS** — A platform for transforming CSS via JS plugins. Used by **Autoprefixer**, Tailwind CSS, CSS Modules. Modern stacks favor PostCSS over SCSS.

---

### Meta Tags & SEO

```html
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<meta name="description" content="India's largest e-commerce site">
<meta name="keywords" content="shopping, deals">
<meta property="og:title" content="...">  <!-- For social sharing -->
<meta property="og:image" content="...">
<meta name="robots" content="index, follow">
<link rel="canonical" href="https://flipkart.com/...">
```

**SEO basics through HTML:**
- One `<h1>` per page, hierarchical headings.
- Semantic tags: `<header>`, `<nav>`, `<main>`, `<article>`, `<footer>`.
- `alt` text on images.
- Descriptive `<title>` (50–60 chars).
- Structured data (JSON-LD).
- Fast LCP, good Core Web Vitals.

---

### display: none vs visibility: hidden vs opacity: 0

| | `display: none` | `visibility: hidden` | `opacity: 0` |
|---|---|---|---|
| Takes space | ❌ | ✅ | ✅ |
| Receives clicks | ❌ | ❌ | ✅ |
| In accessibility tree | ❌ | ❌ | ✅ |
| Animatable | ❌ | ❌ | ✅ |

---

### Pseudo-class vs Pseudo-element

- **Pseudo-class** (single colon `:`): A *state* of the element. `:hover`, `:focus`, `:nth-child(2)`.
- **Pseudo-element** (double colon `::`): A virtual *part* of the element. `::before`, `::after`, `::first-letter`.

```css
a:hover { color: red; }              /* pseudo-class */
p::first-line { font-weight: bold; } /* pseudo-element */
```

---

### Flexbox Cheat Sheet

```css
.container {
  display: flex;
  flex-direction: row | column;
  justify-content: flex-start | center | space-between | space-around;
  align-items: stretch | center | flex-start | flex-end;
  flex-wrap: nowrap | wrap;
  gap: 16px;
}
.item {
  flex: 1;             /* grow shrink basis */
  flex-grow: 1;
  flex-shrink: 0;
  flex-basis: 200px;
  align-self: center;  /* override container's align-items */
}
```

**Quick pattern for the n-children-in-row layout from interviews:**
```css
.parent { display: flex; flex-wrap: wrap; gap: 8px; }
.child { flex: 0 0 33.33%; }
```

---

## 9. JavaScript Advanced Topics

### Strict Mode

```javascript
'use strict';
```

- Throws errors for undeclared variables.
- Disallows duplicate parameter names.
- `this` is `undefined` in standalone functions (not `window`).
- Restricts `eval` scope.
- Reserved keywords protected.

---

### Data Types (8)

`number`, `bigint`, `string`, `boolean`, `null`, `undefined`, `object`, `symbol`.

**`null` vs `undefined`:**
- `null` — Explicitly assigned ("I set it to nothing").
- `undefined` — Never assigned.
- `typeof null === 'object'` (historical bug).
- `null == undefined` is `true`, but `null === undefined` is `false`.

---

### IIFE (Immediately Invoked Function Expression)

```javascript
(function() {
  console.log('runs immediately');
})();
```

Use cases: Avoid global scope pollution (pre-ES6), create private scope for closures.

---

### Generators

```javascript
function* gen() {
  yield 1;
  yield 2;
  yield 3;
}
const g = gen();
g.next(); // { value: 1, done: false }
g.next(); // { value: 2, done: false }
g.next(); // { value: 3, done: false }
g.next(); // { value: undefined, done: true }
```

**Use cases:** Lazy iteration, infinite sequences, redux-saga (async flows), pausable functions.

---

### Prototype & Prototypal Inheritance

Every object has a hidden `[[Prototype]]` (accessible as `__proto__`). When you access a property, JS walks up the prototype chain until found or reaches `null`.

```javascript
function Animal(name) { this.name = name; }
Animal.prototype.eat = function() { console.log(`${this.name} eats`); };

function Dog(name) { Animal.call(this, name); }
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;
Dog.prototype.bark = function() { console.log('woof'); };

const d = new Dog('Rex');
d.eat();  // 'Rex eats'  (via prototype chain)
d.bark(); // 'woof'
```

**`__proto__` vs `prototype`:**
- `prototype` is on **constructor functions**. Used to build new instances.
- `__proto__` is on **instances**. Points to constructor's `prototype`.
- `(new Foo()).__proto__ === Foo.prototype` // true

---

### Object.freeze / seal / preventExtensions

```javascript
const obj = { name: 'A' };

Object.preventExtensions(obj); // Can't ADD new props, can modify/delete existing
Object.seal(obj);              // Can't add/delete, can modify
Object.freeze(obj);            // Can't add/delete/modify (shallow!)

// Selective: defineProperty
Object.defineProperty(obj, 'name', {
  writable: false,
  configurable: false,
});
```

**🔥 Tricky: For one prop modifiable, others not:**
```javascript
const player = { name: 'avi', age: 25 };
Object.defineProperty(player, 'age', { writable: false }); // age locked, name editable
```

---

### First-Class Functions & Higher-Order Functions

**First-class:** Functions are values — can be assigned, passed as args, returned.
**Higher-order:** A function that takes/returns another function. `map`, `filter`, `reduce`, `forEach`.

---

### Currying

Transform `f(a, b, c)` into `f(a)(b)(c)`.

```javascript
function add(a, b, c) { return a + b + c; }

// Manual curry
const addCurry = a => b => c => a + b + c;
addCurry(1)(2)(3); // 6

// Generic curry — handles partial args
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) return fn.apply(this, args);
    return (...more) => curried.apply(this, [...args, ...more]);
  };
}
const cAdd = curry(add);
cAdd(1)(2)(3);  // 6
cAdd(1, 2)(3);  // 6
cAdd(1)(2, 3);  // 6
```

---

### Polyfill: Array.map

```javascript
Array.prototype.myMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback.call(this, this[i], i, this));
  }
  return result;
};
```

### Polyfill: Array.filter

```javascript
Array.prototype.myFilter = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    if (callback.call(this, this[i], i, this)) result.push(this[i]);
  }
  return result;
};
```

### Polyfill: Array.reduce

```javascript
Array.prototype.myReduce = function(callback, initial) {
  let acc = initial;
  let startIdx = 0;
  if (acc === undefined) {
    acc = this[0];
    startIdx = 1;
  }
  for (let i = startIdx; i < this.length; i++) {
    acc = callback(acc, this[i], i, this);
  }
  return acc;
};
```

### Polyfill: Promise.all

```javascript
Promise.myAll = function(promises) {
  return new Promise((resolve, reject) => {
    const results = [];
    let completed = 0;
    if (!promises.length) return resolve([]);
    promises.forEach((p, i) => {
      Promise.resolve(p).then(
        val => {
          results[i] = val;
          if (++completed === promises.length) resolve(results);
        },
        err => reject(err)
      );
    });
  });
};
```

### Polyfill: Debounce

```javascript
function debounce(fn, delay) {
  let timer;
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

### Polyfill: Throttle

```javascript
function throttle(fn, delay) {
  let last = 0;
  return function(...args) {
    const now = Date.now();
    if (now - last >= delay) {
      last = now;
      fn.apply(this, args);
    }
  };
}
```

---

### Memoization

```javascript
function memoize(fn) {
  const cache = new Map();
  return function(...args) {
    const key = JSON.stringify(args);
    if (cache.has(key)) return cache.get(key);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

const slowSquare = memoize(n => {
  for (let i = 0; i < 1e9; i++); // simulate work
  return n * n;
});
slowSquare(5); // slow
slowSquare(5); // instant
```

---

### Map, Filter, Reduce — Real World

```javascript
const users = [
  { name: 'Suhas', age: 20 }, { name: 'Baji', age: 30 },
  { name: 'Iyer',  age: 40 }, { name: 'Anil', age: 20 },
];

// Map — names array
users.map(u => u.name);

// Filter — adults > 25
users.filter(u => u.age > 25);

// Reduce — group by age
users.reduce((acc, u) => {
  acc[u.age] = (acc[u.age] || 0) + 1;
  return acc;
}, {});
// { 20: 2, 30: 1, 40: 1 }
```

---

### Functional Programming Concepts

- **Pure functions** — Same input → same output, no side effects.
- **Immutability** — Don't mutate; return new values.
- **First-class & higher-order functions.**
- **Function composition** — `compose(f, g)(x) = f(g(x))`.
- **Currying** — Partial application.
- **Recursion** — Iterative loops replaced with self-calls (with base case).

---

### Promise Patterns

```javascript
// Sequential
async function seq() {
  const a = await fetchA();
  const b = await fetchB(a); // depends on a
  return b;
}

// Parallel
async function par() {
  const [a, b] = await Promise.all([fetchA(), fetchB()]);
  return { a, b };
}

// First to finish wins
const fastest = await Promise.race([fetchA(), fetchB()]);

// First successful
const first = await Promise.any([fetchA(), fetchB()]);

// Wait for all (don't fail fast)
const results = await Promise.allSettled([fetchA(), fetchB()]);
```

---

### Sleep Function

```javascript
const sleep = ms => new Promise(res => setTimeout(res, ms));

async function demo() {
  console.log('start');
  await sleep(2000);
  console.log('after 2s');
}
```

---

### `==` vs `===`

`===` checks type AND value. `==` does type coercion.

```javascript
0 == false      // true
0 === false     // false
1 == '1'        // true
1 === '1'       // false
null == undefined   // true
null === undefined  // false
[] == []        // false (different references)
NaN == NaN      // false (NaN compared to nothing equals NaN)
```

**Rule:** Always use `===` unless you specifically need coercion.

---

### Negation Quirks

```javascript
console.log(![]);  // false  ([] is truthy)
console.log(!{});  // false  ({} is truthy)
console.log(![] == [])  // true (!!! bizarre coercion)
```

---

### Comparison Output Trap

```javascript
console.log(5 < 6 < 7);   // true   →  (5<6)<7 → true<7 → 1<7 → true
console.log(7 > 6 > 5);   // false  →  (7>6)>5 → true>5 → 1>5 → false
```

---

## 10. React Advanced Topics

### Controlled vs Uncontrolled Components

| | Controlled | Uncontrolled |
|---|---|---|
| Source of truth | React state | DOM (refs) |
| Value | `value={state}` | Default value via `defaultValue` |
| Update | `onChange` handler | Read via `ref.current.value` |
| Use case | Validation, conditional UI | Quick file inputs, simple forms |

```javascript
// Controlled
<input value={name} onChange={e => setName(e.target.value)} />

// Uncontrolled
const ref = useRef();
<input defaultValue="abc" ref={ref} />
// later: ref.current.value
```

---

### Context API

For passing data deeply without prop drilling.

```javascript
const ThemeContext = createContext('light');

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Page />
    </ThemeContext.Provider>
  );
}

function Page() {
  const theme = useContext(ThemeContext); // 'dark'
  return <div className={theme}>...</div>;
}
```

**Tradeoff:** Every consumer re-renders when value changes. For frequent updates, use Redux/Zustand.

---

### React.lazy + Suspense

```javascript
const Heavy = React.lazy(() => import('./Heavy'));

<Suspense fallback={<Spinner />}>
  <Heavy />
</Suspense>
```

Code splits the bundle. Good for routes, admin panels, modals shown rarely.

---

### Error Boundaries

Catch errors during **rendering, lifecycle, constructors**. Do NOT catch errors in event handlers, async code, SSR.

```javascript
class ErrorBoundary extends React.Component {
  state = { hasError: false };
  static getDerivedStateFromError() { return { hasError: true }; }
  componentDidCatch(error, info) { logger.log(error, info); }
  render() {
    if (this.state.hasError) return <h1>Something went wrong</h1>;
    return this.props.children;
  }
}

// Wrap routes or risky components
<ErrorBoundary><Page /></ErrorBoundary>
```

For event handlers / async, use plain `try/catch`.

---

### Why setState is Asynchronous & Why You Shouldn't Do `count = 10`

```javascript
// ❌ Direct mutation
this.state.count = 10; // React doesn't re-render!

// ✅ Tells React to schedule re-render
this.setState({ count: 10 });

// ❌ Relying on current state in setState (may be stale due to batching)
this.setState({ count: this.state.count + 1 });

// ✅ Functional updater — guaranteed latest state
this.setState(prev => ({ count: prev.count + 1 }));
```

---

### When does a React Component Re-render?

1. State change (`setState`, `useState` setter).
2. Parent re-renders (and child isn't memoized).
3. Context value changes (only consumers).
4. `forceUpdate()` (avoid).

**Won't re-render:**
- New state shallow-equal to old (React bails out).
- Component wrapped in `React.memo` and props haven't changed (shallow).

---

### Composition vs Inheritance

React docs explicitly say: **prefer composition.**

```javascript
// Specialization via composition
function Dialog({ title, children }) {
  return (
    <div className="dialog">
      <h2>{title}</h2>
      {children}
    </div>
  );
}

function WelcomeDialog() {
  return <Dialog title="Welcome">Hi there!</Dialog>;
}
```

---

### Keys in Lists

```javascript
{items.map(item => <Row key={item.id} item={item} />)} // ✅
{items.map((item, i) => <Row key={i} item={item} />)}  // ❌ index key
```

**Why index keys are bad:** Reordering or inserting at the front causes React to misidentify items, leading to stale state in inputs and unnecessary re-renders.

---

## 11. Browser, Storage & Security

### localStorage vs sessionStorage vs Cookies

| | localStorage | sessionStorage | Cookies |
|---|---|---|---|
| Capacity | ~5–10 MB | ~5 MB | ~4 KB |
| Lifetime | Until cleared | Tab session | Configurable expiry |
| Sent with requests | ❌ | ❌ | ✅ Auto with HTTP requests |
| Accessible by JS | ✅ | ✅ | Optional (`HttpOnly` flag) |
| Use case | User prefs, drafts | Multi-step form data | Auth tokens, server-set sessions |

---

### JWT — How It Works

JWT = `header.payload.signature` (base64url encoded).

**Flow:**
1. User logs in → server validates → issues JWT signed with server secret.
2. Client stores JWT (memory / `httpOnly` cookie / localStorage).
3. Client sends `Authorization: Bearer <jwt>` on subsequent requests.
4. Server verifies signature with same secret → trusts the payload.

**Where to store JWT (security tradeoff):**

| Storage | Pros | Cons |
|---|---|---|
| `localStorage` | Easy | Vulnerable to XSS |
| `httpOnly` cookie | Safer from XSS | Vulnerable to CSRF (mitigate with `SameSite=strict` + CSRF token) |
| In-memory (state) | Most secure | Lost on refresh |

**Where do session expiry details live?** Inside JWT payload as `exp` claim (UNIX timestamp). Server rejects expired tokens.

---

### Same-Origin Policy & CORS

**Same-origin policy:** JS on `siteA.com` can't read responses from `siteB.com` by default.

**CORS** (Cross-Origin Resource Sharing): Server explicitly allows certain origins via `Access-Control-Allow-Origin` header.

---

### Service Workers & PWA

A Service Worker is a JS script that runs in the background, separate from the page. Enables:
- **Offline support** (cache responses).
- **Push notifications.**
- **Background sync.**

PWA = Progressive Web App = Service Worker + Web App Manifest + HTTPS.

---

### Browser Rendering Pipeline

1. **Parse HTML** → DOM tree.
2. **Parse CSS** → CSSOM.
3. **DOM + CSSOM** → **Render Tree** (only visible nodes).
4. **Layout** — Calculate position/size.
5. **Paint** — Fill pixels.
6. **Composite** — Combine layers (GPU).

**Reflow** = recalculating layout (expensive). Triggered by changing geometry (width, height, margin, font-size).
**Repaint** = recalculating colors only (cheaper).

**To optimize:** Animate `transform` and `opacity` (GPU-accelerated, no layout/paint).

---

## 12. Machine Coding Problems Bank

### 1. Modal Component (Common!)

**Requirements:**
- Button opens modal with transparent black overlay.
- Close button (X) inside modal.
- Modal centered horizontally + vertically.
- Click on overlay closes.
- Long content scrolls inside modal.
- Fixed title at top, fixed footer at bottom.

```jsx
import { useState, useEffect } from 'react';

function Modal({ isOpen, onClose, title, footer, children }) {
  useEffect(() => {
    const handleEsc = (e) => { if (e.key === 'Escape') onClose(); };
    if (isOpen) document.addEventListener('keydown', handleEsc);
    return () => document.removeEventListener('keydown', handleEsc);
  }, [isOpen, onClose]);

  if (!isOpen) return null;

  return (
    <div
      className="fixed inset-0 z-50 flex items-center justify-center bg-black/60"
      onClick={onClose}
    >
      <div
        className="relative bg-white rounded-lg w-[90%] max-w-2xl max-h-[80vh] flex flex-col"
        onClick={e => e.stopPropagation()}
      >
        <div className="flex justify-between items-center p-4 border-b">
          <h2 className="font-bold">{title}</h2>
          <button onClick={onClose} aria-label="Close">✕</button>
        </div>
        <div className="overflow-y-auto p-4 flex-1">{children}</div>
        {footer && <div className="p-4 border-t">{footer}</div>}
      </div>
    </div>
  );
}

// Usage
function App() {
  const [open, setOpen] = useState(false);
  return (
    <>
      <button onClick={() => setOpen(true)}>Open</button>
      <Modal
        isOpen={open}
        onClose={() => setOpen(false)}
        title="Terms"
        footer={<button onClick={() => setOpen(false)}>Accept</button>}
      >
        {/* long content scrolls */}
      </Modal>
    </>
  );
}
```

**Key tricks:**
- `e.stopPropagation()` on inner div prevents overlay-click from firing.
- `max-h-[80vh]` + `overflow-y-auto` makes content scrollable.
- ESC key support is bonus points.
- For real apps: use a Portal (`createPortal`) so modal escapes parent stacking contexts.

---

### 2. Autosuggest / Autocomplete with Debouncing

```jsx
function Autocomplete() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [showDropdown, setShowDropdown] = useState(false);
  const debounced = useDebounce(query, 300);
  const ref = useRef();

  useEffect(() => {
    if (!debounced) return setResults([]);
    fetch(`/api/search?q=${debounced}`)
      .then(r => r.json())
      .then(data => { setResults(data); setShowDropdown(true); });
  }, [debounced]);

  // Click outside to close
  useEffect(() => {
    const handler = (e) => {
      if (ref.current && !ref.current.contains(e.target)) setShowDropdown(false);
    };
    document.addEventListener('mousedown', handler);
    return () => document.removeEventListener('mousedown', handler);
  }, []);

  return (
    <div ref={ref} className="relative">
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        onFocus={() => setShowDropdown(true)}
      />
      {showDropdown && results.length > 0 && (
        <ul className="absolute bg-white border w-full">
          {results.map(r => <li key={r.id}>{r.name}</li>)}
        </ul>
      )}
    </div>
  );
}
```

**Bonus features they ask for:** keyboard navigation (↑↓ Enter), highlighting matched text, caching prior queries, loading state.

---

### 3. Counter App with useMemo / useCallback

```jsx
function Counter() {
  const [count, setCount] = useState(0);
  const [other, setOther] = useState('');

  const expensive = useMemo(() => {
    let sum = 0;
    for (let i = 0; i < 1e7; i++) sum += i;
    return sum + count; // depends on count
  }, [count]);

  const increment = useCallback(() => setCount(c => c + 1), []);

  return (
    <>
      <p>Count: {count}, Expensive: {expensive}</p>
      <Button onClick={increment} />
      <input value={other} onChange={e => setOther(e.target.value)} />
    </>
  );
}

const Button = React.memo(({ onClick }) => {
  console.log('Button rendered');
  return <button onClick={onClick}>+</button>;
});
```

---

### 4. Job Board — Hacker News API

```jsx
function JobBoard() {
  const [ids, setIds] = useState([]);
  const [jobs, setJobs] = useState([]);
  const [page, setPage] = useState(0);
  const [loading, setLoading] = useState(false);
  const PAGE_SIZE = 6;

  // Fetch all IDs once
  useEffect(() => {
    fetch('https://hacker-news.firebaseio.com/v0/jobstories.json')
      .then(r => r.json()).then(setIds);
  }, []);

  // Load next page
  const loadMore = async () => {
    setLoading(true);
    const nextIds = ids.slice(page * PAGE_SIZE, (page + 1) * PAGE_SIZE);
    const newJobs = await Promise.all(
      nextIds.map(id => fetch(`https://hacker-news.firebaseio.com/v0/item/${id}.json`).then(r => r.json()))
    );
    setJobs(prev => [...prev, ...newJobs]);
    setPage(p => p + 1);
    setLoading(false);
  };

  useEffect(() => { if (ids.length) loadMore(); }, [ids.length]);

  const hasMore = page * PAGE_SIZE < ids.length;

  return (
    <div>
      {jobs.map(j => (
        <div key={j.id}>
          {j.url ? <a href={j.url} target="_blank" rel="noopener">{j.title}</a> : <span>{j.title}</span>}
          <small> by {j.by}, {new Date(j.time * 1000).toLocaleDateString()}</small>
        </div>
      ))}
      {hasMore && <button onClick={loadMore} disabled={loading}>{loading ? 'Loading...' : 'Load more'}</button>}
    </div>
  );
}
```

---

### 5. Country/City Weather (Common machine-coding round)

```jsx
function WeatherApp() {
  const [country, setCountry] = useState('IN');
  const [selectedCity, setSelectedCity] = useState(null);
  const [weather, setWeather] = useState(null);

  const cityMap = {
    IN: [{ id: 1, name: 'Mumbai' }, { id: 2, name: 'Delhi' }, /* ... */],
    US: [{ id: 11, name: 'NYC' }, /* ... */],
    PK: [{ id: 21, name: 'Karachi' }, /* ... */],
  };

  useEffect(() => {
    if (!selectedCity) return;
    fetch(`/api/weather/${selectedCity.id}`).then(r => r.json()).then(setWeather);
  }, [selectedCity]);

  return (
    <div className="flex">
      <div className="w-1/4">
        {['IN', 'US', 'PK'].map(c => (
          <button key={c} onClick={() => { setCountry(c); setSelectedCity(null); }}>{c}</button>
        ))}
      </div>
      <div className="w-1/4">
        {cityMap[country].map(city => (
          <div key={city.id} onClick={() => setSelectedCity(city)}>{city.name}</div>
        ))}
      </div>
      <div className="flex-1">
        {weather ? <pre>{JSON.stringify(weather, null, 2)}</pre> : 'Select a city'}
      </div>
    </div>
  );
}
```

---

### 6. Progress Bar (Resumable)

```jsx
function ProgressBar() {
  const [progress, setProgress] = useState(0);
  const [running, setRunning] = useState(false);

  useEffect(() => {
    if (!running) return;
    const id = setInterval(() => {
      setProgress(p => {
        if (p >= 100) { clearInterval(id); return 100; }
        return p + 1;
      });
    }, 50);
    return () => clearInterval(id);
  }, [running]);

  return (
    <div>
      <div className="w-full bg-gray-200 h-4 rounded">
        <div className="bg-blue-500 h-4 rounded transition-all" style={{ width: `${progress}%` }} />
      </div>
      <button onClick={() => setRunning(r => !r)}>{running ? 'Pause' : 'Resume'}</button>
      <button onClick={() => { setProgress(0); setRunning(false); }}>Reset</button>
    </div>
  );
}
```

---

### 7. Memory Game

```jsx
function MemoryGame() {
  const [cards, setCards] = useState([1, 2, 3, 4, 5]);
  const [target, setTarget] = useState(0);
  const [phase, setPhase] = useState('show'); // show | shuffle | guess | result
  const [result, setResult] = useState('');

  useEffect(() => {
    setTarget(Math.floor(Math.random() * cards.length));
    const t1 = setTimeout(() => setPhase('shuffle'), 2000);
    const t2 = setTimeout(() => {
      setCards(prev => [...prev].sort(() => Math.random() - 0.5));
      setPhase('guess');
    }, 3000);
    return () => { clearTimeout(t1); clearTimeout(t2); };
  }, []);

  const handleGuess = (idx) => {
    setResult(cards[idx] === cards[target] ? 'You won!' : 'You lost!');
    setPhase('result');
  };

  return (
    <div>
      {phase === 'show' && <p>Remember card #{target + 1}: {cards[target]}</p>}
      {phase === 'shuffle' && <p>Shuffling...</p>}
      <div className="flex gap-2">
        {cards.map((c, i) => (
          <div key={i} onClick={() => phase === 'guess' && handleGuess(i)} className="w-16 h-16 bg-gray-300 flex items-center justify-center cursor-pointer">
            {phase === 'show' ? c : '?'}
          </div>
        ))}
      </div>
      {phase === 'result' && <p>{result}</p>}
    </div>
  );
}
```

---

### 8. Render Table + Search + Role Filter

```jsx
function UserTable({ users }) {
  const [search, setSearch] = useState('');
  const [role, setRole] = useState('All');

  const filtered = users.filter(u =>
    u.name.toLowerCase().includes(search.toLowerCase()) &&
    (role === 'All' || u.role === role)
  );

  const roles = ['All', ...new Set(users.map(u => u.role))];

  return (
    <div>
      <input placeholder="Search name" value={search} onChange={e => setSearch(e.target.value)} />
      <select value={role} onChange={e => setRole(e.target.value)}>
        {roles.map(r => <option key={r}>{r}</option>)}
      </select>
      <table>
        <thead><tr><th>ID</th><th>Name</th><th>Age</th><th>Role</th><th>Status</th></tr></thead>
        <tbody>
          {filtered.map(u => (
            <tr key={u.id}>
              <td>{u.id}</td><td>{u.name}</td><td>{u.age}</td><td>{u.role}</td>
              <td>{u.isActive ? 'Active' : 'Inactive'}</td>
            </tr>
          ))}
        </tbody>
      </table>
    </div>
  );
}
```

---

### 9. Carousel

```jsx
function Carousel({ images }) {
  const [idx, setIdx] = useState(0);
  const next = () => setIdx(i => (i + 1) % images.length);
  const prev = () => setIdx(i => (i - 1 + images.length) % images.length);

  useEffect(() => {
    const id = setInterval(next, 3000); // auto-play
    return () => clearInterval(id);
  }, []);

  return (
    <div className="relative w-full">
      <img src={images[idx]} alt="" className="w-full" />
      <button onClick={prev} className="absolute left-2 top-1/2">‹</button>
      <button onClick={next} className="absolute right-2 top-1/2">›</button>
      <div className="flex justify-center gap-2 mt-2">
        {images.map((_, i) => (
          <span key={i} onClick={() => setIdx(i)}
            className={`w-2 h-2 rounded-full ${i === idx ? 'bg-black' : 'bg-gray-400'}`} />
        ))}
      </div>
    </div>
  );
}
```

---

### 10. Infinite Scroll with Search (College Metrics-style)

```jsx
function InfiniteList() {
  const [items, setItems] = useState([]);
  const [page, setPage] = useState(1);
  const [search, setSearch] = useState('');
  const [hasMore, setHasMore] = useState(true);
  const debounced = useDebounce(search, 300);
  const observer = useRef();

  // Reset on search
  useEffect(() => { setItems([]); setPage(1); setHasMore(true); }, [debounced]);

  // Fetch
  useEffect(() => {
    fetch(`/api/items?q=${debounced}&page=${page}`)
      .then(r => r.json())
      .then(data => {
        setItems(prev => page === 1 ? data.items : [...prev, ...data.items]);
        setHasMore(data.hasMore);
      });
  }, [debounced, page]);

  // Intersection Observer for last item
  const lastItemRef = useCallback(node => {
    if (observer.current) observer.current.disconnect();
    observer.current = new IntersectionObserver(entries => {
      if (entries[0].isIntersecting && hasMore) setPage(p => p + 1);
    });
    if (node) observer.current.observe(node);
  }, [hasMore]);

  return (
    <div>
      <input value={search} onChange={e => setSearch(e.target.value)} />
      {items.map((item, i) => (
        <div key={item.id} ref={i === items.length - 1 ? lastItemRef : null}>
          {item.name}
        </div>
      ))}
    </div>
  );
}
```

---

### 11. Lazy Image with Intersection Observer

```jsx
function LazyImage({ src, alt }) {
  const [loaded, setLoaded] = useState(false);
  const ref = useRef();

  useEffect(() => {
    const obs = new IntersectionObserver(([entry]) => {
      if (entry.isIntersecting) { setLoaded(true); obs.disconnect(); }
    });
    if (ref.current) obs.observe(ref.current);
    return () => obs.disconnect();
  }, []);

  return <img ref={ref} src={loaded ? src : ''} alt={alt} loading="lazy" />;
}
```

---

### 12. Todo List

```jsx
function TodoApp() {
  const [items, setItems] = useState([]);
  const [text, setText] = useState('');

  const add = () => {
    if (!text.trim()) return;
    setItems([...items, { id: Date.now(), text, done: false }]);
    setText('');
  };

  const toggle = (id) => setItems(items.map(i => i.id === id ? { ...i, done: !i.done } : i));
  const remove = (id) => setItems(items.filter(i => i.id !== id));

  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} onKeyDown={e => e.key === 'Enter' && add()} />
      <button onClick={add}>Add</button>
      <ul>
        {items.map(i => (
          <li key={i.id}>
            <span onClick={() => toggle(i.id)} style={{ textDecoration: i.done ? 'line-through' : 'none' }}>
              {i.text}
            </span>
            <button onClick={() => remove(i.id)}>×</button>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

### 13. Toggle Message on Button Click

```jsx
function Toggle() {
  const [visible, setVisible] = useState(false);
  return (
    <>
      <button onClick={() => setVisible(v => !v)}>Toggle</button>
      {visible && <p>Hello world!</p>}
    </>
  );
}
```

---

### 14. Editable Name Fields

```jsx
function NameForm() {
  const [first, setFirst] = useState('');
  const [last, setLast] = useState('');
  return (
    <div>
      <input placeholder="First" value={first} onChange={e => setFirst(e.target.value)} />
      <input placeholder="Last" value={last} onChange={e => setLast(e.target.value)} />
      <p>Hello, {first} {last}</p>
    </div>
  );
}
```

---

## 13. DSA Problems for Frontend

### 1. Flatten Deeply Nested Array

```javascript
function flatten(arr, depth = Infinity) {
  return depth > 0
    ? arr.reduce((acc, val) =>
        acc.concat(Array.isArray(val) ? flatten(val, depth - 1) : val), [])
    : arr.slice();
}
flatten([1, [2, [3, [4, [5]]]]]); // [1, 2, 3, 4, 5]
```

---

### 2. Square + Sort Array (Two-Pointer for Sorted Input)

```javascript
// For ALREADY-SORTED input: O(n)
function sortedSquares(nums) {
  const result = new Array(nums.length);
  let l = 0, r = nums.length - 1, i = nums.length - 1;
  while (l <= r) {
    if (Math.abs(nums[l]) > Math.abs(nums[r])) {
      result[i--] = nums[l] * nums[l]; l++;
    } else {
      result[i--] = nums[r] * nums[r]; r--;
    }
  }
  return result;
}
sortedSquares([-2, -1, 0, 1, 2]); // [0, 1, 1, 4, 4]

// Or simply: nums.map(x => x*x).sort((a,b) => a-b)  — O(n log n)
```

---

### 3. Merge Two Sorted Arrays

```javascript
function merge(a, b) {
  const result = [];
  let i = 0, j = 0;
  while (i < a.length && j < b.length) {
    if (a[i] <= b[j]) result.push(a[i++]);
    else result.push(b[j++]);
  }
  while (i < a.length) result.push(a[i++]);
  while (j < b.length) result.push(b[j++]);
  return result;
}
merge([1, 3, 5], [2, 4, 6]); // [1, 2, 3, 4, 5, 6]
```

---

### 4. Remove Duplicates — Max 2 Allowed

```javascript
function removeDuplicates(arr) {
  let i = 0; // write pointer
  for (const num of arr) {
    if (i < 2 || arr[i - 2] !== num) arr[i++] = num;
  }
  return arr.slice(0, i);
}
removeDuplicates([1, 1, 1, 2, 2, 2, 3, 3, 3, 4]); // [1, 1, 2, 2, 3, 3, 4]
```
**Time:** O(n) | **Space:** O(1)

---

### 5. add(1)(2)(3) → 6 (Currying)

```javascript
function add(a) {
  return function(b) {
    return function(c) {
      return a + b + c;
    };
  };
}
add(1)(2)(3); // 6
```

---

### 6. Tree Path Finding (Common DSA in interviews)

Find path from root to target node in a binary tree.

```javascript
function findPath(root, target, path = []) {
  if (!root) return null;
  path.push(root.val);
  if (root.val === target) return [...path];
  const left = findPath(root.left, target, path);
  if (left) return left;
  const right = findPath(root.right, target, path);
  if (right) return right;
  path.pop(); // backtrack
  return null;
}
```

---

### 7. Sum Property on Array Prototype

```javascript
Array.prototype.sum = function() {
  return this.reduce((a, b) => a + b, 0);
};
[1, 2, 3, 4].sum(); // 10
```

---

## 14. Hiring Manager Round Playbook

### Tell me about yourself

**Structure (Past → Present → Future, 90 sec):**
> "I'm a frontend engineer with X years building React-based products. Currently at [Company], I work on [most impressive project], where I [specific accomplishment with metric]. Before this, I [past role with growth]. I'm looking for [why Flipkart] because [specific to them]."

---

### Most challenging project — STAR format

- **Situation:** Scope, team, why it mattered.
- **Task:** Your role & responsibility.
- **Action:** Concrete steps you took.
- **Result:** Quantified outcome (latency reduced X%, conversion up Y%).

---

### How do you handle conflicts with the design team?

- Frame the disagreement around the user, not personal preferences.
- Bring data — A/B tests, accessibility audits, performance budgets.
- Suggest a small experiment to settle it instead of a long debate.
- Always escalate via email summary so both sides have alignment in writing.

---

### How do you ensure collaborative work in a team?

- Daily standups, weekly demos.
- Shared design system / component library to reduce friction.
- PRs with templates: what + why + screenshots + test plan.
- Pair-programming on complex stuff.
- Document decisions (ADRs).

---

### What new skill have you learned in the last 6 months?

Pick something genuine and specific. Be ready to explain a real example. Example: "I picked up React Server Components — built a side project to migrate from Pages Router to App Router, learned about streaming SSR and the trade-offs."

---

### What's a feature you built and how did you take it to production?

**Mention concretely:**
1. Requirements gathering — PM brought spec → I asked clarifying questions about edge cases.
2. Design review with designer + engineering lead.
3. Tech doc — components needed, API contracts, state management.
4. Implementation — small atomic PRs with reviews.
5. Testing — unit (Jest), integration (RTL), E2E (Playwright/Cypress).
6. Feature flags — rolled out to 1% → 10% → 100%.
7. Monitoring — Sentry/Datadog dashboards, watched for errors / latency spikes.
8. Post-launch — analyzed metrics, wrote retro for the team.

---

### Mentoring experience

> "I mentored two juniors on our team — set up a weekly 1:1, paired on their first 2-3 features, walked them through code review feedback patiently, and gave them progressively bigger tickets. One of them became a primary owner of our checkout module within 6 months."

---

### Questions YOU should ask the interviewer

- "What does success look like for this role in 6 months / 1 year?"
- "How does the team handle technical debt?"
- "What's the deployment cadence — daily, weekly?"
- "How are product priorities balanced against engineering improvements?"
- "What's the team's growth path — tech-lead vs deeper IC?"
- "What's your favorite thing about working here?"

---

## 🎯 Pre-Interview Day Checklist

**JavaScript:**
- [ ] Closures, loop bug fix (var → let), TDZ
- [ ] Event loop output question
- [ ] this binding rules
- [ ] bind/call/apply + bind polyfill
- [ ] Promise.all polyfill
- [ ] Currying (add(1)(2)(3))
- [ ] Flatten nested array
- [ ] Debounce + throttle implementations

**React:**
- [ ] useEffect / useLayoutEffect output trace
- [ ] Parent/child useEffect ordering
- [ ] useMemo vs useCallback vs React.memo (when to use which)
- [ ] useDebounce custom hook
- [ ] Lifecycle methods → hooks mapping
- [ ] Error boundaries (class component, can't catch event handlers/async)
- [ ] Controlled vs uncontrolled
- [ ] Why setState is async, functional updater

**HTML/CSS:**
- [ ] Box model + box-sizing
- [ ] Flexbox properties
- [ ] Position values + center a div 3 ways
- [ ] Specificity
- [ ] display:none vs visibility:hidden vs opacity:0

**System Design:**
- [ ] What happens when you type URL (full pipeline)
- [ ] CSR vs SSR vs ISR
- [ ] Hydration
- [ ] localStorage vs sessionStorage vs cookies
- [ ] JWT — where to store, expiry handling
- [ ] Redis use cases
- [ ] Load balancer + sticky sessions + Redis sessions

**Coding Practice:**
- [ ] Build a Modal in 15 mins
- [ ] Build Autocomplete with debounce in 20 mins
- [ ] Build Infinite Scroll in 25 mins
- [ ] Multi-step form with validation
- [ ] Memory game

**Behavioral:**
- [ ] 90-sec elevator pitch
- [ ] 2 STAR stories ready (challenge / conflict)
- [ ] 3 questions ready to ask interviewer

---

*Master prep document — Last updated May 2026 | Flipkart via ThinkifyLabs*
