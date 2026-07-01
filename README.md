# Lab 20: useFormStatus, useImperativeHandle, useDeferredValue & useTransition Hooks

## Overview
This lab demonstrates four React hooks across two categories:
1. **Form & Ref Hooks** → `useFormStatus` (react-dom) and `useImperativeHandle`
2. **Concurrency Hooks** → `useDeferredValue` and `useTransition`

---

# Part 1: useFormStatus & useImperativeHandle

---

## useFormStatus Hook

### What is it?
`useFormStatus` provides information about form submission status, allowing you to track whether a form is currently being submitted.

### Basic Syntax
```jsx
import { useFormStatus } from 'react-dom';

function SubmitButton() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? 'Submitting...' : 'Submit'}</button>;
}
```

### Key Rules
- **MUST be called inside a component that's rendered INSIDE a `<form>` tag**
- Cannot be used in the same component that renders the `<form>`
- The form must have an `action` prop with an async function

### Returns
- `pending` - Boolean indicating if form is submitting
- `data` - FormData object with form values
- `method` - HTTP method (GET/POST)
- `action` - Function/URL form is submitting to

---

## useImperativeHandle Hook

### What is it?
`useImperativeHandle` allows child components to expose specific methods to parent components through refs, providing controlled imperative access.

### Basic Syntax
```jsx
import { useImperativeHandle, forwardRef, useRef } from 'react';

const ChildComponent = forwardRef((props, ref) => {
  const internalRef = useRef();
  
  useImperativeHandle(ref, () => ({
    methodName: () => { /* logic */ }
  }));
  
  return <div ref={internalRef}>...</div>;
});
```

### Key Rules
- **MUST be used with `forwardRef`**
- Exposes only the methods you define
- Parent calls methods via `ref.current.methodName()`

---

## Programs (Part 1)

### Program A: ProgramA.jsx

**Purpose**: Disable submit button while form is submitting

**What it does**:
- Disables submit button when form is being submitted
- Changes button text to "Submitting..." during submission
- Re-enables button after submission completes

**Implementation**:
```jsx
function SubmitButton() {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'Submitting...' : 'Submit'}
    </button>
  );
}
```

**How it works**:
1. `SubmitButton` uses `useFormStatus()` to get `pending` state
2. When form submits, `pending` becomes `true`
3. Button gets disabled and text changes
4. After async action completes, `pending` becomes `false`

---

### Program B: ProgramB.jsx

**Purpose**: Display success message from server after form submission

**What it does**:
- Submits form data to server (simulated)
- Displays success message after server responds
- Shows message returned from server

**Implementation**:
```jsx
function ProgramB() {
  const [message, setMessage] = useState('');

  async function handleSubmit() {
    const response = await submitForm();
    setMessage(response);
  }

  return (
    <div>
      {message && <p>{message}</p>}
      <form action={handleSubmit}>
        <input type="text" name="email" required />
        <SubmitButton />
      </form>
    </div>
  );
}
```

**How it works**:
1. Form submits with async `handleSubmit` function
2. `submitForm()` simulates server delay (2 seconds)
3. Server returns success message
4. Message is stored in state and displayed
5. `useFormStatus` tracks submission state in button

---

### Program A: InputFocusDemo.jsx

**Purpose**: Demonstrates useImperativeHandle with input focus

**What it does**:
1. Child component (ChildInput) exposes `.focus()` method to parent
2. Parent (ParentInput) can imperatively focus the input by calling the method
3. Demonstrates controlled access to child component internals

**Files**:
- `ChildInput.jsx` - Child component with input and useImperativeHandle
- `ParentInput.jsx` - Parent component that controls the child

**Implementation**:
```jsx
// Child Component
const ChildInput = forwardRef((props, ref) => {
  const inputRef = useRef();
  
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus()
  }));
  
  return <input ref={inputRef} />;
});

// Parent Component
function ParentInput() {
  const inputRef = useRef();
  return (
    <div>
      <ChildInput ref={inputRef} />
      <button onClick={() => inputRef.current.focus()}>Focus Input</button>
    </div>
  );
}
```

**How it works**:
1. `ChildInput` uses `forwardRef` to receive ref from parent
2. `useImperativeHandle` exposes only the `focus()` method
3. Parent calls `inputRef.current.focus()` when button clicked
4. Internal input ref remains hidden from parent

---

### Program C: VideoPlayerDemo.jsx

**Purpose**: Demonstrates useImperativeHandle with media controls

**What it does**:
1. Child component (ChildVideo) exposes `play()` and `pause()` methods
2. Parent (ParentVideo) controls video playback through custom buttons
3. Separates video element from UI controls

**Files**:
- `ChildVideo.jsx` - Child component with video and useImperativeHandle
- `ParentVideo.jsx` - Parent component with control buttons

**Implementation**:
```jsx
// Child Component
const ChildVideo = forwardRef((props, ref) => {
  const videoRef = useRef();
  
  useImperativeHandle(ref, () => ({
    play: () => videoRef.current.play(),
    pause: () => videoRef.current.pause()
  }));
  
  return <video ref={videoRef} src="video.mp4" />;
});

// Parent Component
function ParentVideo() {
  const playerRef = useRef();
  return (
    <div>
      <ChildVideo ref={playerRef} />
      <button onClick={() => playerRef.current.play()}>Play</button>
      <button onClick={() => playerRef.current.pause()}>Pause</button>
    </div>
  );
}
```

**How it works**:
1. `ChildVideo` wraps `<video>` element and uses `forwardRef`
2. `useImperativeHandle` exposes `play()` and `pause()` methods
3. Parent buttons call these methods to control playback
4. Video element ref remains encapsulated in child

---
---

# Part 2: useDeferredValue & useTransition (Concurrency Hooks)

---

## 🧠 Why Do We Need Concurrency Hooks?

**The Problem:** When you type in a search box and React tries to filter 10,000+ items on every keystroke, the UI **freezes** because React treats ALL state updates as equally urgent.

**The Solution:** React 18 introduced **concurrent features** that let you tell React: _"This update is NOT urgent — do it in the background."_

| Hook | What it defers | Who controls it |
|------|---------------|-----------------|
| `useDeferredValue` | A **value** — the rendered output lags behind | React decides when to update |
| `useTransition` | A **state update** — you wrap `setState` in `startTransition` | You decide what to defer |

---

## ❌ WITHOUT Hooks — See The Problem First!

Before learning the hooks, **run these 4 files first** to experience the problem yourself:

### WithoutDeferredSearch.jsx — Search Filter WITHOUT useDeferredValue

**What happens:** You type in the search box but the input **freezes and lags** on every keystroke. You can't type smoothly at all.

**Why it freezes:** The filtering of 10,000 items runs on EVERY keystroke. React can't update the input until the filtering finishes. So typing one letter = wait for filtering = then see the letter appear.

```
Type "R" → React starts filtering 10,000 items → UI FROZEN → filtering done → "R" appears
Type "e" → React starts filtering again → UI FROZEN → filtering done → "Re" appears
Type "a" → React starts filtering again → UI FROZEN → filtering done → "Rea" appears
```

**Code (super simple — just useState, NO hook):**
```jsx
const [text, setText] = useState('');

<input value={text} onChange={(e) => setText(e.target.value)} />
<FilteredList query={text} />    ← text updates AND filtering runs together = FREEZE
```

**Compare with DeferredSearchFilter.jsx** which uses `useDeferredValue` — input stays smooth because filtering is deferred.

---

### WithoutDeferredPreview.jsx — Preview WITHOUT useDeferredValue

**What happens:** You type in the textarea but it **freezes** because the slow preview re-renders on every single keystroke.

**Why it freezes:** Both the textarea AND the preview use the same `text` state. Since the preview takes 200ms to render, every keystroke is blocked by that 200ms delay.

```
Type "H" → textarea updates + preview renders (200ms) → UI FROZEN → "H" appears
Type "e" → textarea updates + preview renders (200ms) → UI FROZEN → "He" appears
```

**Code:**
```jsx
const [text, setText] = useState('');

<textarea value={text} onChange={(e) => setText(e.target.value)} />
<SlowPreview text={text} />    ← same value for both = textarea waits for preview
```

**Compare with DeferredPreview.jsx** which uses `useDeferredValue` — textarea updates instantly, preview catches up later.

---

### WithoutTransitionFilter.jsx — Filter WITHOUT useTransition

**What happens:** Same as above — the input **freezes** while filtering 20,000 items. No loading indicator, no feedback, just a frozen UI.

**Why it freezes:** There's only ONE state variable (`text`). Updating it triggers BOTH the input re-render AND the heavy filtering at the same time. React can't prioritize one over the other.

```jsx
const [text, setText] = useState('');

// ONE state does EVERYTHING — input + filtering happen together
const filtered = items.filter(item => item.includes(text));
```

**Compare with TransitionFilter.jsx** which uses `useTransition` — it splits into TWO states (`inputValue` for typing, `filterQuery` for filtering) and wraps the filter update in `startTransition`.

---

### WithoutTransitionTabs.jsx — Tabs WITHOUT useTransition

**What happens:** You click a tab and the UI **completely freezes** for ~500ms. No "Loading..." message, no feedback. The user doesn't know if the click worked.

**Why it freezes:** `setTab('About')` triggers a re-render of `SlowTab` which takes 500ms. During that 500ms, the entire UI is frozen — buttons don't respond, nothing updates.

```
Click "About" → setTab('About') → SlowTab renders (500ms) → UI FROZEN → tab appears
```

**Code:**
```jsx
const [tab, setTab] = useState('Home');

<button onClick={() => setTab('About')}>About</button>
<SlowTab name={tab} />    ← no loading state, just freeze
```

**Compare with TransitionTabs.jsx** which uses `useTransition` — shows "Loading…" while the tab renders in the background.

---

### Side-by-Side Summary: Without vs With

| Scenario | ❌ Without Hook | ✅ With Hook |
|----------|----------------|---------------|
| Search filter | Input freezes on every keystroke | Input stays smooth, list updates later |
| Editor + Preview | Textarea freezes while preview renders | Textarea instant, preview catches up |
| Filter huge list | No loading indicator, UI frozen | Shows "Filtering..." while processing |
| Tab switching | UI freezes, no feedback | Shows "Loading…" then switches |

---

## useDeferredValue Hook

### What is it?
`useDeferredValue` accepts a value and returns a **deferred copy** of it. React will first render with the old deferred value (keeping interactions responsive), then re-render in the background with the new value.

### Basic Syntax
```jsx
import { useDeferredValue } from 'react';

function MyComponent() {
  const [text, setText] = useState('');
  const deferredText = useDeferredValue(text);
  //    ^^^^^^^^^^^^
  //    This lags behind `text` during heavy renders
}
```

### Workflow Diagram
```
User types "R" → "Re" → "Rea" → "Reac" → "React"

WITHOUT useDeferredValue:
  Type "R"  → [FREEZE while filtering] → Type "e" → [FREEZE] → ...
  (each keystroke blocks the UI)

WITH useDeferredValue:
  Type "R"  → Input updates instantly ✅
  Type "Re" → Input updates instantly ✅
  Type "Rea" → Input updates instantly ✅
  ... React filters with "Rea" in background (deferred)
  → Filtered list appears smoothly
```

### Key Points
- `useDeferredValue(value)` returns a copy that may lag behind `value`
- React prioritizes keeping the UI responsive (input updates)
- The deferred re-render can be interrupted if a new value arrives
- Use `value !== deferredValue` to detect if the deferred value is stale

---

### Program A (useDeferredValue): DeferredSearchFilter.jsx

**Purpose**: Filter a large array (10,000 items) with smooth typing

**Workflow:**
```
┌──────────────────────────────────────────────────────────┐
│                    User types "react"                     │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  1. onChange fires → setSearchText("react")              │
│     ↓                                                    │
│  2. searchText = "react" (input re-renders IMMEDIATELY)  │
│     ↓                                                    │
│  3. deferredSearch = useDeferredValue(searchText)        │
│     → Still holds OLD value (e.g., "reac")               │
│     → FilteredList re-renders with OLD data              │
│     ↓                                                    │
│  4. React finishes urgent work, then updates             │
│     deferredSearch → "react"                             │
│     → FilteredList re-renders with NEW filtered data     │
│                                                          │
│  Result: Input never freezes, list updates smoothly      │
└──────────────────────────────────────────────────────────┘
```

**Key Code:**
```jsx
const [searchText, setSearchText] = useState('');
const deferredSearch = useDeferredValue(searchText);
const isStale = searchText !== deferredSearch;

// Input uses searchText (instant)
<input value={searchText} onChange={(e) => setSearchText(e.target.value)} />

// List uses deferredSearch (deferred) — fades while stale
<div style={{ opacity: isStale ? 0.6 : 1 }}>
  <FilteredList query={deferredSearch} />
</div>
```

---

### Program B (useDeferredValue): DeferredPreview.jsx

**Purpose**: Editor text updates immediately; preview text updates slowly

**Workflow:**
```
┌──────────────────────────────────────────────────────────┐
│              User types in the Editor textarea            │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ┌─────────────┐         ┌─────────────────────┐        │
│  │   EDITOR     │         │     PREVIEW          │        │
│  │  uses: text  │         │  uses: deferredText  │        │
│  │  (instant)   │         │  (slow / deferred)   │        │
│  └──────┬──────┘         └──────────┬──────────┘        │
│         │                           │                    │
│    User types                  React defers              │
│    "Hello" →                   "Hello" update            │
│    textarea shows              until it's ready          │
│    "Hello" IMMEDIATELY         to re-render the          │
│                                expensive preview         │
│                                                          │
│  Result: Typing is never blocked by preview rendering    │
└──────────────────────────────────────────────────────────┘
```

**Key Code:**
```jsx
const [text, setText] = useState('');
const deferredText = useDeferredValue(text);

// Editor uses `text` → updates on every keystroke (fast)
<textarea value={text} onChange={(e) => setText(e.target.value)} />

// Preview uses `deferredText` → updates when React is ready (slow)
<SlowPreview text={deferredText} />
```

---

## useTransition Hook

### What is it?
`useTransition` lets you mark a **state update** as a non-urgent transition. While the transition is in progress, React gives you an `isPending` flag to show loading states.

### Basic Syntax
```jsx
import { useTransition } from 'react';

function MyComponent() {
  const [isPending, startTransition] = useTransition();
  
  const handleClick = () => {
    startTransition(() => {
      setSomeState(newValue);  // This update is non-urgent
    });
  };
  
  if (isPending) return <p>Loading...</p>;
}
```

### Workflow Diagram
```
WITHOUT useTransition:
  Click Tab → [UI FREEZES while rendering heavy content] → Tab shows

WITH useTransition:
  Click Tab → isPending = true → Show "Loading..."
            → React renders new tab content in background
            → isPending = false → Show actual content
  (UI stays interactive the entire time)
```

### Key Difference from useDeferredValue

| Feature | useDeferredValue | useTransition |
|---------|------------------|---------------|
| What you pass | A **value** | A **setState call** |
| Who controls timing | React | You (via `startTransition`) |
| Loading indicator | `value !== deferredValue` | `isPending` boolean |
| Best for | Deferring renders of received data | Deferring state updates you initiate |

---

### Program A (useTransition): TransitionFilter.jsx

**Purpose**: Filter huge list (20,000 items) without blocking UI

**Workflow:**
```
┌──────────────────────────────────────────────────────────┐
│              User types "Java" in search box              │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  handleChange(e) runs:                                   │
│                                                          │
│  1. setInputValue("Java")          ← URGENT (immediate) │
│     → Input shows "Java" instantly                       │
│                                                          │
│  2. startTransition(() => {        ← NON-URGENT          │
│       setFilterQuery("Java")                             │
│     })                                                   │
│     → isPending = true (show "⏳ Filtering...")          │
│     → React filters list in BACKGROUND                   │
│     → isPending = false (show filtered results)          │
│                                                          │
│  Key: Two separate state variables!                      │
│  • inputValue → for the input (always instant)           │
│  • filterQuery → for the list (wrapped in transition)    │
└──────────────────────────────────────────────────────────┘
```

**Key Code:**
```jsx
const [inputValue, setInputValue] = useState('');
const [filterQuery, setFilterQuery] = useState('');
const [isPending, startTransition] = useTransition();

const handleChange = (e) => {
  setInputValue(e.target.value);          // Urgent: input updates NOW
  startTransition(() => {
    setFilterQuery(e.target.value);       // Non-urgent: filter updates LATER
  });
};
```

---

### Program B (useTransition): TransitionTabs.jsx

**Purpose**: Show "Loading…" while switching tabs

**Workflow:**
```
┌──────────────────────────────────────────────────────────┐
│              User clicks "About" tab                      │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  handleTabClick("about") runs:                           │
│                                                          │
│  startTransition(() => {                                 │
│    setActiveTab("about")                                 │
│  })                                                      │
│     │                                                    │
│     ├── isPending = true                                 │
│     │   → UI shows: ⏳ "Loading…"                       │
│     │   → User can still click other tabs!               │
│     │                                                    │
│     ├── React renders <SlowTabContent tab="about" />     │
│     │   (heavy computation in background)                │
│     │                                                    │
│     └── isPending = false                                │
│         → UI shows: About tab content                    │
│                                                          │
│  Result: UI never freezes, user sees loading feedback    │
└──────────────────────────────────────────────────────────┘
```

**Key Code:**
```jsx
const [activeTab, setActiveTab] = useState('home');
const [isPending, startTransition] = useTransition();

const handleTabClick = (tabKey) => {
  startTransition(() => {
    setActiveTab(tabKey);      // Non-urgent: wrapped in transition
  });
};

// Show loading or content based on isPending
{isPending ? <div>⏳ Loading…</div> : <SlowTabContent tab={activeTab} />}
```

---

## How to Run

1. Components are already exported from `index.js`
2. Import any component in `App.jsx` and uncomment it
3. Run: `npm run dev`
4. Test each program:
   - **ProgramA**: Fill input, click Submit, watch button disable
   - **ProgramB**: Fill input, click Submit, success message appears
   - **InputFocus**: Click "Focus Input" button
   - **VideoPlayer**: Click Play/Pause buttons
   - **Try WITHOUT first** (to see the problem):
     - **WithoutDeferredSearch**: Type fast — notice input freezing
     - **WithoutDeferredPreview**: Type fast — notice textarea freezing
     - **WithoutTransitionFilter**: Type fast — notice input freezing
     - **WithoutTransitionTabs**: Click tabs — notice UI freezing
   - **Then try WITH hooks** (to see the solution):
     - **DeferredSearchFilter**: Type in search box — smooth typing!
     - **DeferredPreview**: Type in editor — preview lags but typing is smooth!
     - **TransitionFilter**: Type in search — see "Filtering..." indicator!
     - **TransitionTabs**: Click tabs — see "Loading..." between switches!

## File Structure

```
lab-20/
  │
  │── ProgramA.jsx                  (useFormStatus - disable button)
  │── ProgramB.jsx                  (useFormStatus - success message)
  │── ChildInput.jsx                (useImperativeHandle - child input)
  │── ParentInput.jsx               (useImperativeHandle - parent input)
  │── ChildVideo.jsx                (useImperativeHandle - child video)
  │── ParentVideo.jsx               (useImperativeHandle - parent video)
  │
  │── DeferredSearchFilter.jsx      ✅ useDeferredValue - smooth search
  │── DeferredPreview.jsx           ✅ useDeferredValue - instant editor + slow preview
  │── TransitionFilter.jsx          ✅ useTransition - filter with startTransition
  │── TransitionTabs.jsx            ✅ useTransition - tab switch with Loading...
  │
  │── WithoutDeferredSearch.jsx     ❌ WITHOUT useDeferredValue - search FREEZES
  │── WithoutDeferredPreview.jsx    ❌ WITHOUT useDeferredValue - preview FREEZES
  │── WithoutTransitionFilter.jsx   ❌ WITHOUT useTransition - filter FREEZES
  │── WithoutTransitionTabs.jsx     ❌ WITHOUT useTransition - tabs FREEZE
  │
  │── index.js                      (all exports)
  └── README.md
```

---

## Important Concepts

### useFormStatus
- Automatically tracks form submission state
- No manual state management needed
- Works with React Server Actions
- `pending` becomes true when form action starts executing
- Returns to false when async action completes

### useImperativeHandle
- Provides "escape hatch" for imperative code
- Encapsulates internal implementation
- Parent only sees methods you expose
- Useful for DOM operations (focus, scroll, play/pause)
- Should be used sparingly - prefer declarative patterns when possible

### useDeferredValue
- Returns a deferred copy of a value that lags behind the original
- React prioritizes urgent updates (typing) over deferred updates (filtering)
- The deferred render can be interrupted by new inputs
- Great for search filters, previews, autocomplete results

### useTransition
- Gives you `[isPending, startTransition]`
- Wrap non-urgent `setState` calls inside `startTransition`
- `isPending` lets you show loading indicators
- Great for tab switches, navigation, heavy computations

---

## Quick Cheat Sheet: When to Use What?

| Scenario | Use This Hook |
|----------|--------------|
| Filter a big list while typing | `useDeferredValue` or `useTransition` |
| Show loading while tab switches | `useTransition` |
| Editor + live preview | `useDeferredValue` |
| Need `isPending` flag | `useTransition` |
| Don't control the state update | `useDeferredValue` |
| You trigger the state update | `useTransition` |

---

## Troubleshooting

### useFormStatus not working?
- Check: Is `useFormStatus()` called INSIDE a component rendered inside `<form>`?
- Check: Does form have `action` prop with async function?
- Check: Are you using react-dom version that supports this hook?

### useImperativeHandle not working?
- Check: Is component wrapped with `forwardRef`?
- Check: Is ref being passed from parent?
- Check: Are you calling methods after component mounts?
- Use: `ref.current?.method()` for safety

### useDeferredValue not showing any lag?
- Check: Is the child component doing expensive work? (Add artificial delay for demo)
- Check: Wrap the expensive component in `useMemo` to prevent unnecessary re-renders
- Check: Make sure the deferred value is passed to the expensive child, NOT the original

### useTransition isPending always false?
- Check: Is the state update inside `startTransition()` doing enough work to be noticeable?
- Check: Wrap heavy computation in `useMemo` so React actually defers it
- Check: Make sure you're not accidentally using the immediate value instead of the transitioned one

---

## Key Takeaways

1. **useFormStatus** simplifies form submission UI without manual state
2. **useImperativeHandle** gives controlled access to child component methods
3. **useDeferredValue** defers a value — React decides when to update the deferred copy
4. **useTransition** defers a state update — you decide what is non-urgent
5. Both concurrency hooks prevent UI blocking during heavy computations
6. Use `isPending` or `value !== deferredValue` to show loading/stale states
