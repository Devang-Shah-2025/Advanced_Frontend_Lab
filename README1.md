# Lab 19: useReducer Hook — Complete Guide with Redux Comparison

## Overview
This lab demonstrates the **`useReducer`** hook in React. We cover its syntax, workflow, and real-world usage with a Todo App, and then compare it side-by-side with **Redux** so you can understand when to use which.

---

## 📚 What is useReducer?

`useReducer` is a React hook for managing **complex state logic**. Instead of updating state directly (like `useState`), you describe **what happened** (an action) and a **reducer function** decides how the state should change.

> **Think of it like a restaurant:**
> - You (component) place an **order** (dispatch an action)
> - The **kitchen** (reducer) reads the order and prepares the food (new state)
> - The **waiter** brings the food back (React re-renders with new state)

### Basic Syntax

```jsx
import { useReducer } from 'react';

const [state, dispatch] = useReducer(reducerFunction, initialState);
```

| Part | What it is | Example |
|------|-----------|---------|
| `state` | Current state value | `{ count: 0 }` |
| `dispatch` | Function to send actions | `dispatch({ type: 'INCREMENT' })` |
| `reducerFunction` | Pure function: `(state, action) → newState` | See below |
| `initialState` | Starting value of state | `{ count: 0 }` |

---

## 🔄 How useReducer Works — Step-by-Step Workflow

```
┌──────────────────────────────────────────────────────────────────────┐
│                       useReducer WORKFLOW                            │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│   STEP 1: Define Initial State                                       │
│   ┌──────────────────────────────┐                                   │
│   │  const initialState = {      │                                   │
│   │    todos: [                  │                                   │
│   │      { id: 1, text: '...' } │                                   │
│   │    ]                         │                                   │
│   │  }                           │                                   │
│   └──────────────────────────────┘                                   │
│                    ↓                                                 │
│   STEP 2: Define Reducer Function                                    │
│   ┌──────────────────────────────────────┐                           │
│   │  function reducer(state, action) {   │                           │
│   │    switch (action.type) {            │                           │
│   │      case 'ADD':  → return newState  │  ← Pure function         │
│   │      case 'TOGGLE': → return ...     │  ← No side effects       │
│   │      case 'REMOVE': → return ...     │  ← Always returns        │
│   │      default: → return state         │     new state object      │
│   │    }                                 │                           │
│   │  }                                   │                           │
│   └──────────────────────────────────────┘                           │
│                    ↓                                                 │
│   STEP 3: Initialize useReducer in Component                        │
│   ┌───────────────────────────────────────────────────────┐          │
│   │  const [state, dispatch] = useReducer(reducer, init)  │          │
│   └───────────────────────────────────────────────────────┘          │
│                    ↓                                                 │
│   STEP 4: Component Dispatches an Action                             │
│   ┌──────────────────────────────────────────────────┐               │
│   │  dispatch({ type: 'ADD', payload: 'New Todo' })  │               │
│   └──────────────────────────────────────────────────┘               │
│         │                                                            │
│         │  React receives the action                                 │
│         ↓                                                            │
│   STEP 5: Reducer Processes the Action                               │
│   ┌──────────────────────────────────────────────────┐               │
│   │  reducer(currentState, { type:'ADD', payload })  │               │
│   │  → case 'ADD' matches                           │               │
│   │  → returns { ...state, todos: [..., newTodo] }   │               │
│   └──────────────────────────────────────────────────┘               │
│         │                                                            │
│         ↓                                                            │
│   STEP 6: React Updates State & Re-renders                           │
│   ┌──────────────────────────────────────────────────┐               │
│   │  state = newState → Component re-renders         │               │
│   │  → UI shows the updated todo list                │               │
│   └──────────────────────────────────────────────────┘               │
│                                                                      │
└──────────────────────────────────────────────────────────────────────┘
```

---

## 📁 File: Demo.jsx — useRef Basics (Reference file)

This file demonstrates `useRef` use-cases (render counting, input focus, previous value tracking). It is included in the lab for context but is **not** related to `useReducer`.

---

## 📁 File: TodoReducer.jsx — useReducer Todo App

### Complete Code Walkthrough

Let's walk through the `TodoReducer.jsx` file step by step:

#### Step 1: Initial State
```jsx
const initialState = {
  todos: [
    { id: 1, text: 'Learn React Hooks', completed: false },
    { id: 2, text: 'Build a Todo App', completed: false },
  ],
};
```
> This is the starting state — an object with a `todos` array. Each todo has `id`, `text`, and `completed`.

#### Step 2: Reducer Function
```jsx
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return {
        ...state,
        todos: [...state.todos, { id: Date.now(), text: action.payload, completed: false }],
      };
    case 'TOGGLE':
      return {
        ...state,
        todos: state.todos.map((todo) =>
          todo.id === action.payload ? { ...todo, completed: !todo.completed } : todo
        ),
      };
    case 'REMOVE':
      return {
        ...state,
        todos: state.todos.filter((todo) => todo.id !== action.payload),
      };
    default:
      return state;
  }
}
```

**How each action works:**

| Action Type | What `action.payload` contains | What happens |
|-------------|-------------------------------|--------------|
| `'ADD'` | The text of new todo (string) | Spreads existing todos, adds new one at end |
| `'TOGGLE'` | The `id` of the todo to toggle | Maps over all todos, flips `completed` for matching id |
| `'REMOVE'` | The `id` of the todo to delete | Filters out the todo with matching id |
| `default` | — | Returns state unchanged (safety net) |

> ⚠️ **Important:** The reducer NEVER mutates the original state. It always returns a **new object** using spread operator (`...state`).

#### Step 3: Component Using useReducer
```jsx
function TodoReducer() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  const [input, setInput] = useState('');
  //     ↑                    ↑
  //  current state        function to
  //  (has .todos)         send actions
```

#### Step 4: Dispatching Actions
```jsx
// ADD a new todo
dispatch({ type: 'ADD', payload: input.trim() });

// TOGGLE a todo's completed status
dispatch({ type: 'TOGGLE', payload: todo.id });

// REMOVE a todo
dispatch({ type: 'REMOVE', payload: todo.id });
```

### Visual Flow of Adding a Todo

```
User types "Study for exam" and clicks Add
          │
          ↓
dispatch({ type: 'ADD', payload: 'Study for exam' })
          │
          ↓
todoReducer(currentState, { type: 'ADD', payload: 'Study for exam' })
          │
          ↓
case 'ADD' matches → returns:
{
  todos: [
    { id: 1, text: 'Learn React Hooks', completed: false },
    { id: 2, text: 'Build a Todo App', completed: false },
    { id: 1719832800000, text: 'Study for exam', completed: false }  ← NEW
  ]
}
          │
          ↓
React re-renders → UI shows 3 todos
```

---

## 🆚 useReducer vs Redux — Side-by-Side Comparison

### The Fundamental Difference

| Aspect | useReducer | Redux (+ Redux Toolkit) |
|--------|-----------|------------------------|
| **What is it?** | A React **hook** (built into React) | An **external library** (needs `npm install`) |
| **Scope** | Local to **one component** (+ children via props/context) | **Global** — accessible by any component in the app |
| **Setup** | Zero setup — just import from 'react' | Needs: store, slices, Provider, npm packages |
| **Package size** | 0 KB (included in React) | ~11 KB (Redux Toolkit + React-Redux) |
| **Learning curve** | Low — just a function and dispatch | Medium — store, slices, selectors, middleware |

### Architecture Comparison

```
┌──────────────────────────────────────────────────────────────┐
│                    useReducer Architecture                    │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   Component                                                  │
│   ┌────────────────────────────────────────┐                 │
│   │  const [state, dispatch] = useReducer  │                 │
│   │                                        │                 │
│   │  state lives HERE, inside component    │                 │
│   │  dispatch lives HERE, inside component │                 │
│   │                                        │                 │
│   │  To share? → Pass via props or Context │                 │
│   └────────────────────────────────────────┘                 │
│                                                              │
│   ✅ Simple, self-contained                                  │
│   ❌ Hard to share across distant components                 │
│                                                              │
└──────────────────────────────────────────────────────────────┘

┌──────────────────────────────────────────────────────────────┐
│                    Redux Architecture                        │
├──────────────────────────────────────────────────────────────┤
│                                                              │
│   ┌──────────────────────────────┐                           │
│   │   GLOBAL STORE (single)      │  ← Lives outside React   │
│   │   ┌───────────────────────┐  │                           │
│   │   │ slice: cart           │  │                           │
│   │   │ slice: user           │  │                           │
│   │   │ slice: theme          │  │                           │
│   │   └───────────────────────┘  │                           │
│   └────────────┬─────────────────┘                           │
│                │                                             │
│       <Provider store={store}>                               │
│                │                                             │
│   ┌────────────┴─────────────────┐                           │
│   │  ANY component can:          │                           │
│   │  • useSelector(state => ...) │  ← Read from store       │
│   │  • useDispatch()             │  ← Write to store        │
│   └──────────────────────────────┘                           │
│                                                              │
│   ✅ Easy to share across entire app                         │
│   ❌ More boilerplate and setup                              │
│                                                              │
└──────────────────────────────────────────────────────────────┘
```

### Code Comparison — Same Todo App

#### useReducer Version (this lab)
```jsx
// ━━━━━━━━━━━ Everything in ONE file ━━━━━━━━━━━

import { useReducer, useState } from 'react';

// 1. Initial state
const initialState = {
  todos: [{ id: 1, text: 'Learn Hooks', completed: false }],
};

// 2. Reducer function
function todoReducer(state, action) {
  switch (action.type) {
    case 'ADD':
      return { ...state, todos: [...state.todos, { id: Date.now(), text: action.payload, completed: false }] };
    case 'TOGGLE':
      return { ...state, todos: state.todos.map(t => t.id === action.payload ? { ...t, completed: !t.completed } : t) };
    case 'REMOVE':
      return { ...state, todos: state.todos.filter(t => t.id !== action.payload) };
    default:
      return state;
  }
}

// 3. Component
function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  //     ↑ use directly       ↑ dispatch directly

  return (
    <div>
      <button onClick={() => dispatch({ type: 'ADD', payload: 'New' })}>Add</button>
      {state.todos.map(todo => <div key={todo.id}>{todo.text}</div>)}
    </div>
  );
}
```

#### Redux Toolkit Version (for comparison)
```jsx
// ━━━━━━━━━━━ File 1: store/todoSlice.js ━━━━━━━━━━━
import { createSlice } from '@reduxjs/toolkit';

const todoSlice = createSlice({
  name: 'todos',
  initialState: {
    todos: [{ id: 1, text: 'Learn Redux', completed: false }],
  },
  reducers: {
    add: (state, action) => {
      state.todos.push({ id: Date.now(), text: action.payload, completed: false });
      // ↑ Redux Toolkit uses Immer, so you CAN mutate directly (it handles immutability)
    },
    toggle: (state, action) => {
      const todo = state.todos.find(t => t.id === action.payload);
      if (todo) todo.completed = !todo.completed;
    },
    remove: (state, action) => {
      state.todos = state.todos.filter(t => t.id !== action.payload);
    },
  },
});

export const { add, toggle, remove } = todoSlice.actions;
export default todoSlice.reducer;

// ━━━━━━━━━━━ File 2: store/store.js ━━━━━━━━━━━
import { configureStore } from '@reduxjs/toolkit';
import todoReducer from './todoSlice';

export const store = configureStore({
  reducer: { todos: todoReducer },
});

// ━━━━━━━━━━━ File 3: main.jsx ━━━━━━━━━━━
import { Provider } from 'react-redux';
import { store } from './store/store';

<Provider store={store}>
  <App />
</Provider>

// ━━━━━━━━━━━ File 4: TodoApp.jsx ━━━━━━━━━━━
import { useSelector, useDispatch } from 'react-redux';
import { add, toggle, remove } from './store/todoSlice';

function TodoApp() {
  const todos = useSelector(state => state.todos.todos);
  //           ↑ reads from GLOBAL store
  const dispatch = useDispatch();
  //              ↑ gets dispatch from GLOBAL store

  return (
    <div>
      <button onClick={() => dispatch(add('New'))}>Add</button>
      {todos.map(todo => <div key={todo.id}>{todo.text}</div>)}
    </div>
  );
}
```

### Key Differences in Code

| Feature | useReducer | Redux Toolkit |
|---------|-----------|---------------|
| **Immutability** | You MUST return new objects with `...spread` | Immer handles it — you can `state.push()` directly |
| **Action creators** | Write manually: `{ type: 'ADD', payload: ... }` | Auto-generated: `add(payload)` |
| **Accessing state** | Direct: `state.todos` | Via selector: `useSelector(s => s.todos.todos)` |
| **Dispatching** | Direct from `useReducer`: `dispatch(action)` | From hook: `const dispatch = useDispatch()` |
| **Store setup** | None | `configureStore()` + `<Provider>` |
| **Files needed** | 1 file | 3-4 files (slice, store, provider, component) |
| **Middleware** | Not supported | Built-in (thunks for async, logging, etc.) |
| **DevTools** | No | Redux DevTools (time-travel debugging!) |

---

## 🤔 When to Use Which?

### Use `useReducer` when:
- ✅ State logic is complex but **local to one component**
- ✅ Multiple related state values change together
- ✅ You have a simple app or a single form/feature
- ✅ You don't want to install external packages
- ✅ State doesn't need to be shared across many unrelated components

### Use Redux when:
- ✅ **Multiple distant components** need the same state
- ✅ You need **DevTools** for debugging (time-travel!)
- ✅ You need **middleware** (async operations, logging, analytics)
- ✅ The app is **large scale** with many features sharing state
- ✅ You want centralized, predictable state management

### Decision Flowchart

```
Is the state shared across many distant components?
  │
  ├── YES → Use Redux
  │
  └── NO → Is the state logic complex (multiple actions, nested updates)?
            │
            ├── YES → Use useReducer
            │
            └── NO → Use useState (simplest option!)
```

---

## 🏃 How to Run

1. Import the component in `App.jsx`:
   ```jsx
   import TodoReducer from './Labs/lab-19/TodoReducer';
   ```
2. Add it inside the return:
   ```jsx
   <TodoReducer />
   ```
3. Run: `npm run dev`
4. Test: Add todos, toggle completion, delete todos

---

## File Structure

```
lab-19/
  ├── Demo.jsx         (useRef demo — render count, focus, previous value)
  ├── TodoReducer.jsx  (useReducer — full Todo app with ADD/TOGGLE/REMOVE)
  └── README.md        (this file)
```

---

## 📝 Summary Table

| Concept | useState | useReducer | Redux |
|---------|----------|-----------|-------|
| Complexity | Simple | Medium | High |
| State scope | Local | Local (+ Context) | Global |
| Action types | None | Manual strings | Auto-generated |
| Immutability | Manual | Manual | Automatic (Immer) |
| Async support | Manual | Manual | Middleware (Thunks) |
| DevTools | No | No | Yes |
| Install needed | No | No | Yes |
| Best for | Toggles, forms | Complex local logic | Large apps |

---

## 💡 Key Takeaways

1. **useReducer = local state manager** — the reducer pattern without the boilerplate of Redux
2. **dispatch({ type, payload })** is how you communicate "what happened" to the reducer
3. **Never mutate state directly** in useReducer — always return a new object
4. **Redux = useReducer + global store + middleware + DevTools** — it's the same pattern, scaled up
5. Start with `useState`, graduate to `useReducer` when logic gets complex, and use Redux when you need global shared state
