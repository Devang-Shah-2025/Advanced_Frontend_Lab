# Lab 17: React Hooks - useState, useEffect, useContext

## What are Hooks?

Hooks are functions that allow functional components to use React features like state and lifecycle methods. Before hooks, only class components could use these features.

---

## 🎯 useState Hook

### What is useState?

`useState` is a React hook that adds state management to functional components. It returns an array with two elements: the current state value and a function to update it.

### Syntax

```jsx
const [state, setState] = useState(initialValue);
```

**Parameters:**
- `initialValue`: The initial value of the state variable

**Returns:**
- `state`: Current state value
- `setState`: Function to update the state

### Example Syntax

```jsx
import { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0);
  
  return (
    <button onClick={() => setCount(count + 1)}>
      Count: {count}
    </button>
  );
}
```

### Why Use useState?

**Problem without useState:**
- Functional components cannot store data between renders
- Variables reset to initial values on every render
- UI cannot respond to user interactions

**Solution with useState:**
- State persists between re-renders
- Updating state triggers component re-render
- UI automatically reflects state changes

### Use Cases in This Lab

**Program 1: Counter.jsx**
- **Use case:** Increment/decrement a numeric value
- **State variable:** `count` (number)
- **Operations:** `setCount(count + 1)` and `setCount(count - 1)`

**Program 2: ToggleText.jsx**
- **Use case:** Show/hide content based on boolean flag
- **State variable:** `show` (boolean)
- **Operation:** `setShow(!show)` toggles between true/false

**Program 3: TypeAndShow.jsx**
- **Use case:** Store and display user input in real-time
- **State variable:** `text` (string)
- **Operation:** `setText(e.target.value)` updates on every keystroke

---

## ⏰ useEffect Hook

### What is useEffect?

`useEffect` is a React hook that handles side effects in functional components. Side effects are operations that interact with external systems (API calls, timers, subscriptions, DOM manipulation).

### Syntax

```jsx
useEffect(() => {
  // Effect code here
  
  return () => {
    // Cleanup code (optional)
  };
}, [dependencies]);
```

**Parameters:**
- **Effect function:** Code to run
- **Cleanup function (optional):** Code to run before component unmounts or before effect runs again
- **Dependency array:** Controls when effect runs

### Dependency Array Behavior

| Dependency Array | When Effect Runs |
|-----------------|------------------|
| `[]` (empty) | Only once after first render |
| `[var1, var2]` | After first render + whenever var1 or var2 changes |
| No array | After every render (usually avoided) |

### Example Syntax

```jsx
import { useState, useEffect } from 'react';

function DataFetcher() {
  const [data, setData] = useState([]);
  
  useEffect(() => {
    fetch('https://api.example.com/data')
      .then(res => res.json())
      .then(data => setData(data));
  }, []); // Empty array = run once on mount
  
  return <div>{data.length} items loaded</div>;
}
```

### Why Use useEffect?

**Problem without useEffect:**
- No way to run code after component renders
- Cannot fetch data on component mount
- Cannot set up subscriptions or timers properly
- No cleanup mechanism for resources

**Solution with useEffect:**
- Run code after render completes
- Fetch data when component mounts
- Set up and clean up subscriptions/timers
- Synchronize component with external systems

### Use Cases in This Lab

**Program 4: UsersList.jsx**
- **Use case:** Fetch data from API when component loads
- **Effect:** Calls MockAPI to get users
- **Dependency:** `[]` (empty) - runs once on mount
- **Syntax:**
  ```jsx
  useEffect(() => {
    fetch('https://6978487fcd4fe130e3d8602a.mockapi.io/users')
      .then(response => response.json())
      .then(data => setUsers(data));
  }, []);
  ```

**Program 5: AutoCounter.jsx**
- **Use case:** Increment counter every second using setInterval
- **Effect:** Sets up interval timer
- **Cleanup:** Clears interval when component unmounts
- **Dependency:** `[]` (empty) - set up once
- **Syntax:**
  ```jsx
  useEffect(() => {
    const interval = setInterval(() => {
      setCount((prev) => prev + 1);
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  ```

### Technical Deep Dive: Cleanup Functions

**Why Cleanup is Required:**

When `useEffect` runs, it can set up resources (timers, listeners, subscriptions) that continue running even after the component unmounts. Without cleanup, these resources remain active in memory, causing **memory leaks**.

**What Happens Without clearInterval:**

```jsx
// WITHOUT CLEANUP - Memory Leak
useEffect(() => {
  const interval = setInterval(() => {
    setCount((prev) => prev + 1);
  }, 1000);
  // No cleanup function returned
}, []);
```

**Problems:**
1. **Interval keeps running** even after component is removed from DOM
2. **Multiple intervals accumulate** if component remounts (navigating back to page)
3. **setState called on unmounted component** causes React warnings
4. **Memory leak** - intervals consume memory indefinitely
5. **Performance degradation** - browser becomes slower over time

**Technical Breakdown:**

```jsx
useEffect(() => {
  // SETUP PHASE - Runs when component mounts
  const interval = setInterval(() => {
    setCount((prev) => prev + 1);  // Executes every 1000ms
  }, 1000);
  
  // setInterval returns an interval ID (number)
  // This ID is needed to stop the interval later
  
  // CLEANUP PHASE - Runs when component unmounts
  return () => {
    clearInterval(interval);  // Stops the interval using its ID
  };
}, []);  // Empty dependency = setup once, cleanup on unmount
```

**How clearInterval Works:**

- `setInterval(callback, delay)` returns an **interval ID** (unique number)
- Browser maintains an internal registry of active intervals
- `clearInterval(intervalID)` removes the interval from registry
- Once cleared, callback stops executing and memory is freed

**Cleanup Function Execution Flow:**

1. Component mounts → Effect runs → setInterval starts
2. User navigates away → Component unmounts → Cleanup function runs → clearInterval stops timer
3. Memory is freed, no more setState calls, no warnings

**Always Return Cleanup For:**
- `setInterval` → `clearInterval(intervalID)`
- `setTimeout` → `clearTimeout(timeoutID)`
- `addEventListener` → `removeEventListener`
- WebSocket → `websocket.close()`
- Subscriptions → `subscription.unsubscribe()`

**Memory Leak Example:**

Without cleanup, if user visits and leaves the page 10 times:
- **10 intervals running simultaneously**
- Each calling setState every second
- Trying to update components that no longer exist
- Browser console filled with warnings
- Application becomes sluggish

---

## 🌍 useContext Hook

### What is useContext?

`useContext` is a React hook that allows components to consume context values without prop drilling. It provides a way to share data across the component tree without passing props manually at every level.

### The Fundamental Problem: Prop Drilling

**Scenario:** You have data at the top level that needs to be accessed deep in the component tree.

```jsx
function App() {
  const userName = "John Doe";
  return <Header userName={userName} />;
}

function Header({ userName }) {
  return <Navigation userName={userName} />; // Header doesn't need userName
}

function Navigation({ userName }) {
  return <UserMenu userName={userName} />; // Navigation doesn't need userName
}

function UserMenu({ userName }) {
  return <div>Welcome, {userName}</div>; // Finally used here!
}
```

**Problems:**
1. `Header` and `Navigation` don't use `userName` but must accept and pass it
2. If you need to add more data (email, role), you must update every component
3. Code becomes cluttered with unnecessary props
4. Refactoring is difficult and error-prone

### Context Solution: Direct Data Access

Context creates a **global data store** that any component can access directly, without props.

### Step-by-Step: How Context Works

**Step 1: Create the Context (Create the Storage)**

```jsx
import { createContext } from 'react';

const UserContext = createContext();
```

**What this does:**
- Creates a context object that will hold data
- `UserContext` is now a container that can store and provide data
- Optional: `createContext(defaultValue)` sets a fallback value if no Provider is found

**Step 2: Provide Data (Put Data in Storage)**

```jsx
function App() {
  const userName = "John Doe";
  
  return (
    <UserContext.Provider value={userName}>
      <Header />
      <MainContent />
      <Footer />
    </UserContext.Provider>
  );
}
```

**What this does:**
- `<UserContext.Provider>` wraps components that need access to data
- `value={userName}` is the data being shared
- All components inside `<UserContext.Provider>` can access `userName`
- Components outside the Provider cannot access the data

**Step 3: Consume Data (Get Data from Storage)**

```jsx
import { useContext } from 'react';

function UserMenu() {
  const userName = useContext(UserContext);
  return <div>Welcome, {userName}</div>;
}
```

**What this does:**
- `useContext(UserContext)` retrieves the value from the nearest Provider
- No props needed - direct access to data
- Works at any nesting level

### Complete Working Example

```jsx
import { createContext, useContext, useState } from 'react';

// Step 1: Create Context
const ThemeContext = createContext();

// Step 2: Provider Component
function Parent() {
  const [theme, setTheme] = useState('light');
  
  return (
    <ThemeContext.Provider value={theme}>
      <Child />
    </ThemeContext.Provider>
  );
}

// Step 3: Consumer Component
function Child() {
  const theme = useContext(ThemeContext);
  return <div>Current theme: {theme}</div>;
}
```

### Technical Deep Dive: How Context Works Internally

**1. Context Object Structure:**
```jsx
const MyContext = createContext(defaultValue);
// MyContext = {
//   Provider: Component,  // Used to provide data
//   Consumer: Component,  // Old way to consume (before hooks)
//   _currentValue: value  // Internal: stores current value
// }
```

**2. Provider Mechanism:**
```jsx
<MyContext.Provider value={someData}>
  <ComponentTree />
</MyContext.Provider>
```
- Provider creates a **subscription system**
- When `value` changes, all consumers re-render
- Provider must wrap consumers in component tree
- Can be nested: inner Provider overrides outer Provider

**3. useContext Hook Mechanism:**
```jsx
const value = useContext(MyContext);
```
- React walks up the component tree
- Finds the nearest `<MyContext.Provider>`
- Returns its `value` prop
- If no Provider found, returns `defaultValue` from `createContext(defaultValue)`
- Subscribes component to context changes (auto re-render on updates)

### Context Flow Diagram

```
App (has data: theme = "dark")
  │
  └─ <ThemeContext.Provider value="dark">
       │
       ├─ Header
       │    └─ Logo
       │         └─ useContext(ThemeContext) → "dark" ✓
       │
       ├─ Sidebar
       │    └─ Menu
       │         └─ MenuItem
       │              └─ useContext(ThemeContext) → "dark" ✓
       │
       └─ Footer
            └─ useContext(ThemeContext) → "dark" ✓
```

**Key Points:**
- Provider at top wraps entire tree
- Any component at any level can call `useContext(ThemeContext)`
- No props passed through intermediate components
- Data flows from Provider to all consumers

### Prop Drilling Problem

**Without useContext (Prop Drilling):**

Every intermediate component must receive and pass props, even if they don't use them.

```jsx
function App() {
  const user = { name: "John" };
  return <Parent user={user} />;
}

function Parent({ user }) {
  // Parent doesn't use 'user', just passes it down
  return <Child user={user} />;
}

function Child({ user }) {
  // Child doesn't use 'user', just passes it down
  return <GrandChild user={user} />;
}

function GrandChild({ user }) {
  // Finally used here!
  return <div>{user.name}</div>;
}
```

**Issues:**
- 3 components modified to pass 1 prop
- Add `email`? Update all 3 components again
- Insert new component? Update prop chain again
- Difficult to maintain and refactor

**With useContext (Direct Access):**

Only components that need data access it directly.

```jsx
const UserContext = createContext();

function App() {
  const user = { name: "John" };
  return (
    <UserContext.Provider value={user}>
      <Parent />
    </UserContext.Provider>
  );
}

function Parent() {
  // No user prop needed!
  return <Child />;
}

function Child() {
  // No user prop needed!
  return <GrandChild />;
}

function GrandChild() {
  const user = useContext(UserContext); // Direct access!
  return <div>{user.name}</div>;
}
```

**Benefits:**
- Only `App` (Provider) and `GrandChild` (Consumer) deal with `user`
- Intermediate components are clean and simple
- Add `email`? Only update Provider value, consumers automatically get it
- Insert new components? No changes needed

### Advanced Context Patterns

**Pattern 1: Multiple Context Values**

You can provide multiple pieces of data in one context using an object:

```jsx
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: "John", email: "john@example.com" });
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Profile />
    </UserContext.Provider>
  );
}

function Profile() {
  const { user, setUser } = useContext(UserContext);
  
  return (
    <div>
      <p>{user.name} - {user.email}</p>
      <button onClick={() => setUser({ ...user, name: "Jane" })}>
        Change Name
      </button>
    </div>
  );
}
```

**Pattern 2: Nested Contexts**

Multiple contexts can be used together:

```jsx
const ThemeContext = createContext();
const UserContext = createContext();

function App() {
  return (
    <ThemeContext.Provider value="dark">
      <UserContext.Provider value={{ name: "John" }}>
        <Dashboard />
      </UserContext.Provider>
    </ThemeContext.Provider>
  );
}

function Dashboard() {
  const theme = useContext(ThemeContext);    // Gets "dark"
  const user = useContext(UserContext);      // Gets { name: "John" }
  
  return <div className={theme}>Welcome, {user.name}</div>;
}
```

**Pattern 3: Context with Custom Hook**

Create a custom hook to simplify context usage and add validation:

```jsx
const ThemeContext = createContext();

// Custom hook
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within ThemeProvider');
  }
  return context;
}

// Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  return (
    <ThemeContext.Provider value={{ theme, setTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Usage
function App() {
  return (
    <ThemeProvider>
      <Button />
    </ThemeProvider>
  );
}

function Button() {
  const { theme, setTheme } = useTheme(); // Clean and validated!
  return <button onClick={() => setTheme('dark')}>{theme}</button>;
}
```

### When to Use Context vs Props

**Use Props when:**
- ✅ Data is only needed by direct children
- ✅ Component relationship is clear (parent → child)
- ✅ Data is specific to that component
- ✅ Only 1-2 levels of passing required

**Use Context when:**
- ✅ Data needed by many components at different levels
- ✅ Prop drilling creates messy code
- ✅ Data is truly global (theme, auth, language)
- ✅ Multiple unrelated components need same data

**Avoid Context for:**
- ❌ All state management (use props when appropriate)
- ❌ Frequently changing data (can cause performance issues)
- ❌ Data only used by 1-2 components

### Performance Considerations

**Problem: Context updates re-render all consumers**

```jsx
function App() {
  const [user, setUser] = useState({ name: "John", count: 0 });
  
  return (
    <UserContext.Provider value={user}>
      <NameDisplay />  {/* Re-renders when count changes */}
      <CountDisplay /> {/* Re-renders when name changes */}
    </UserContext.Provider>
  );
}
```

**Solution 1: Split contexts**

```jsx
function App() {
  const [name, setName] = useState("John");
  const [count, setCount] = useState(0);
  
  return (
    <NameContext.Provider value={name}>
      <CountContext.Provider value={count}>
        <NameDisplay />  {/* Only re-renders when name changes */}
        <CountDisplay /> {/* Only re-renders when count changes */}
      </CountContext.Provider>
    </NameContext.Provider>
  );
}
```

**Solution 2: Memoize context value**

```jsx
import { useMemo } from 'react';

function App() {
  const [user, setUser] = useState({ name: "John" });
  
  const value = useMemo(() => ({ user, setUser }), [user]);
  
  return (
    <UserContext.Provider value={value}>
      <Profile />
    </UserContext.Provider>
  );
}
```

### Use Cases in This Lab

**Program 6: ThemeDemo.jsx**
- **Use case:** Share theme preference (dark/light) across multiple components
- **Context:** `ThemeContext`
- **Provider:** Wraps components and provides theme state
- **Consumers:** `ThemeButton` and `ThemeBox` both access theme directly
- **Syntax:**
  ```jsx
  const ThemeContext = createContext();
  
  function ThemeDemo() {
    const [theme, setTheme] = useState('light');
    return (
      <ThemeContext.Provider value={theme}>
        <ThemeButton />
        <ThemeBox />
      </ThemeContext.Provider>
    );
  }
  
  function ThemeButton() {
    const theme = useContext(ThemeContext);
    return <button>Theme: {theme}</button>;
  }
  ```

**Program 7: UserProfile.jsx**
- **Use case:** Share logged-in user details (name, email) across components
- **Context:** `UserContext`
- **Data structure:** `{ name: 'John Doe', email: 'john@example.com' }`
- **Consumers:** `Header` displays welcome message, `ProfileCard` shows full details
- **Syntax:**
  ```jsx
  const UserContext = createContext();
  
  function UserProfile() {
    const user = { name: 'John', email: 'john@example.com' };
    return (
      <UserContext.Provider value={user}>
        <Header />
        <ProfileCard />
      </UserContext.Provider>
    );
  }
  ```

**Program 8: LanguageSelector.jsx**
- **Use case:** Share selected language and display translated messages
- **Context:** `LanguageContext`
- **Data:** Language code ('english', 'spanish', 'french')
- **Messages object:** Contains translations for each language
- **Consumer:** `MessageDisplay` reads language and shows appropriate messages
- **Syntax:**
  ```jsx
  const LanguageContext = createContext();
  
  const messages = {
    english: { greeting: 'Hello!' },
    spanish: { greeting: 'Hola!' }
  };
  
  function LanguageSelector() {
    const [language, setLanguage] = useState('english');
    return (
      <LanguageContext.Provider value={language}>
        <MessageDisplay />
      </LanguageContext.Provider>
    );
  }
  
  function MessageDisplay() {
    const language = useContext(LanguageContext);
    return <h2>{messages[language].greeting}</h2>;
  }
  ```

---

## 📚 Quick Reference Table

| Hook | Purpose | Syntax | Common Use Cases |
|------|---------|--------|------------------|
| **useState** | Manage component state | `const [state, setState] = useState(initial)` | Form inputs, counters, toggles, storing user data |
| **useEffect** | Handle side effects | `useEffect(() => { /* code */ }, [deps])` | API calls, timers, subscriptions, DOM updates |
| **useContext** | Share data across components | `const value = useContext(Context)` | Global state, theme, auth, language settings |

---

## 🔧 Programs Implementation Guide

### useState Programs

| File | State Variable | Type | Initial Value | Update Triggers |
|------|---------------|------|---------------|-----------------|
| Counter.jsx | `count` | number | `0` | Button clicks (+/-) |
| ToggleText.jsx | `show` | boolean | `true` | Button click (toggle) |
| TypeAndShow.jsx | `text` | string | `''` | Input onChange event |

### useEffect Programs

| File | Effect Purpose | Dependency Array | Cleanup Required |
|------|---------------|------------------|------------------|
| UsersList.jsx | Fetch users from API | `[]` (run once) | No |
| AutoCounter.jsx | Increment counter every 1s | `[]` (run once) | Yes (clearInterval) |

### useContext Programs

| File | Context Name | Data Type | Provider Location | Consumers |
|------|-------------|-----------|-------------------|-----------|
| ThemeDemo.jsx | ThemeContext | string | ThemeDemo | ThemeButton, ThemeBox |
| UserProfile.jsx | UserContext | object | UserProfile | Header, ProfileCard |
| LanguageSelector.jsx | LanguageContext | string | LanguageSelector | MessageDisplay |

---

## 🎯 Key Technical Points

### useState Rules
1. **State updates are asynchronous** - React batches updates for performance
2. **Use functional updates** when new state depends on old state: `setState(prev => prev + 1)`
3. **Never mutate state directly** - always create new values
4. **State is isolated** - each component instance has its own state

### useEffect Rules
1. **Always specify dependencies** - empty array `[]` or list all used variables
2. **Cleanup functions prevent memory leaks** - return cleanup for timers/subscriptions
3. **Effects run after render** - DOM is already updated when effect runs
4. **Infinite loops** - avoid missing dependencies or setting state without conditions

### useContext Rules
1. **Provider must wrap consumers** - components must be inside Provider tree
2. **Context updates trigger re-renders** - all consumers re-render when value changes
3. **Default value** - used only when no Provider is found
4. **Don't overuse** - not a replacement for all prop passing

---

## 📝 Common Patterns

### Pattern 1: Form Input Binding (useState)
```jsx
const [value, setValue] = useState('');
<input value={value} onChange={(e) => setValue(e.target.value)} />
```

### Pattern 2: Data Fetching (useEffect + useState)
```jsx
const [data, setData] = useState([]);
useEffect(() => {
  fetch(url).then(res => res.json()).then(setData);
}, []);
```

### Pattern 3: Timer with Cleanup (useEffect)
```jsx
useEffect(() => {
  const interval = setInterval(() => {
    // Timer logic
  }, 1000);
  return () => clearInterval(interval);
}, []);
```

**Why This Pattern:**
- `setInterval` returns interval ID stored in `interval` variable
- Cleanup function uses this ID to stop the timer
- Prevents memory leaks when component unmounts
- Without cleanup: timer continues running in background indefinitely

### Pattern 4: Context Provider Pattern (useContext)
```jsx
const Context = createContext();

function Provider({ children }) {
  const [value, setValue] = useState(initial);
  return <Context.Provider value={value}>{children}</Context.Provider>;
}
```

---

## ⚠️ Common Errors and Solutions

### useState Errors

**Error:** State not updating immediately
```jsx
setCount(count + 1);
console.log(count); // Still shows old value!
```
**Solution:** State updates are asynchronous. Use useEffect to react to changes.

**Error:** Mutating state directly
```jsx
const [items, setItems] = useState([1, 2, 3]);
items.push(4); // Wrong!
```
**Solution:** Create new array
```jsx
setItems([...items, 4]); // Correct
```

### useEffect Errors

**Error:** Infinite loop
```jsx
useEffect(() => {
  setCount(count + 1); // Runs infinitely!
});
```
**Solution:** Add dependency array
```jsx
useEffect(() => {
  setCount(count + 1);
}, []); // Runs once
```

**Error:** Memory leak from timer
```jsx
useEffect(() => {
  setInterval(() => {
    setCount((prev) => prev + 1);
  }, 1000); // Never cleared!
}, []);
```
**Solution:** Return cleanup function
```jsx
useEffect(() => {
  const interval = setInterval(() => {
    setCount((prev) => prev + 1);
  }, 1000);
  return () => clearInterval(interval); // Cleanup
}, []);
```

**Technical Explanation:**
- **Problem:** `setInterval` creates a timer that runs forever
- **Without cleanup:** Timer keeps running even after component unmounts
- **Result:** Multiple timers accumulate, causing memory leaks and performance issues
- **Solution:** Store interval ID and clear it in cleanup function
- **When cleanup runs:** Component unmounts or before effect re-runs
- **Browser behavior:** Each interval consumes ~50KB memory; 100 uncleaned intervals = 5MB leak

### useContext Errors

**Error:** useContext returns undefined
```jsx
const value = useContext(MyContext); // undefined
```
**Solution:** Ensure component is wrapped in Provider
```jsx
<MyContext.Provider value={data}>
  <YourComponent />
</MyContext.Provider>
```

---

## 🚀 Running the Programs

1. Import the component in your App.jsx:
```jsx
import Counter from './Labs/lab-17/Counter';
```

2. Use it in your render:
```jsx
function App() {
  return <Counter />;
}
```

3. Each program is self-contained and ready to run independently.

---

## 📖 Additional Resources

- **React Official Docs:** https://react.dev/reference/react
- **useState:** https://react.dev/reference/react/useState
- **useEffect:** https://react.dev/reference/react/useEffect
- **useContext:** https://react.dev/reference/react/useContext

---

## Summary

- **Hooks enable functional components** to use React features previously only available in class components
- **useState** manages component-local state and triggers re-renders
- **useEffect** handles side effects and lifecycle events with optional cleanup
- **useContext** provides global state sharing without prop drilling
- **All hooks must be called** at the top level of functional components (not in loops/conditions)
- **Each hook solves specific problems** - use the appropriate hook for each use case
