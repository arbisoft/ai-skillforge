---
name: class-to-functional-migration
description: Use when converting a React class component into a functional component with hooks. Handles state migration, lifecycle-to-effect conversion, refs, callbacks, memoization, and iterative quality passes. Trigger phrases — "convert class component", "class to functional", "refactor to hooks", "modernize React component".
origin: community
---

# Class To Functional Migration

A behavior-preserving workflow for converting React class components into functional components that use hooks. The skill enforces a strict mapping from class concepts to hook equivalents, then iterates over the result for correctness, performance, and maintainability.

## When to Activate

- Modernizing a class component to use hooks
- Eliminating `this`, lifecycle methods, and HOC ceremony
- Preparing a component for use with newer React features (Suspense, concurrent rendering)
- Splitting a large class component into smaller hook-based units
- Reviewing an already-converted component for hook idioms and stale-closure bugs

## Core Principles

1. **Behavior parity first.** Do not change props, exports, render output, or business logic during the conversion pass.
2. **Strict mapping.** Every class construct maps to a documented hook equivalent — see [Conversion Map](#conversion-map).
3. **Effect dependencies are not optional.** Each `useEffect` must list every reactive value it reads.
4. **Iterate after converting.** Run correctness, performance, and maintainability passes once the component compiles and behaves correctly.
5. **Do not redesign mid-migration.** Refactor scope (extracting hooks, splitting components) is allowed only after parity is confirmed.

## Pre-Migration Audit

Before writing a single hook, list these from the source class:

- All `this.state` fields and which `setState` calls touch each one
- All lifecycle methods used and what each one does
- All `React.createRef()` and any mutable instance fields (`this.foo = ...`)
- All class methods, including which are passed as props or event handlers
- Any HOCs (`withRouter`, `connect`, `withStyles`) wrapping the export
- Any `getDerivedStateFromProps`, `shouldComponentUpdate`, or `componentDidCatch`
- Any `forwardRef` or imperative handle requirements

Write this audit as a short comment block while reading; discard it after migration. It catches subtle behavior that is easy to miss during conversion.

## Conversion Map

### State

```jsx
// BEFORE
class Counter extends React.Component {
  state = { count: 0, label: 'idle' };

  increment = () => {
    this.setState((prev) => ({ count: prev.count + 1 }));
  };
}

// AFTER
const Counter = () => {
  const [count, setCount] = useState(0);
  const [label, setLabel] = useState('idle');

  const increment = () => setCount((prev) => prev + 1);
};
```

Group related state into a single `useState` only when the fields update together. Otherwise prefer one hook per concern — it makes effect dependencies cleaner.

For complex state with branching transitions, use `useReducer`:

```jsx
const [state, dispatch] = useReducer(reducer, initialState);
```

### Lifecycle Methods

```jsx
// BEFORE
componentDidMount() {
  this.subscription = api.subscribe(this.handleData);
}
componentWillUnmount() {
  this.subscription.unsubscribe();
}

// AFTER
useEffect(() => {
  const subscription = api.subscribe(handleData);
  return () => subscription.unsubscribe();
}, [handleData]);
```

```jsx
// BEFORE
componentDidUpdate(prevProps) {
  if (prevProps.userId !== this.props.userId) {
    this.fetchUser(this.props.userId);
  }
}

// AFTER
useEffect(() => {
  fetchUser(userId);
}, [userId, fetchUser]);
```

Lifecycle to hook mapping at a glance:

| Class lifecycle              | Hook equivalent                                          |
| ---------------------------- | -------------------------------------------------------- |
| `componentDidMount`          | `useEffect(() => { ... }, [])`                           |
| `componentDidUpdate`         | `useEffect(() => { ... }, [dep1, dep2])`                 |
| `componentWillUnmount`       | Cleanup function returned from `useEffect`               |
| `getDerivedStateFromProps`   | Compute during render, or sync via `useEffect` if needed |
| `shouldComponentUpdate`      | Wrap export in `React.memo` with a custom comparator     |
| `componentDidCatch`          | Keep as a class `ErrorBoundary` — hooks cannot replace it|
| `getSnapshotBeforeUpdate`    | `useLayoutEffect` reading the DOM before paint           |

### Refs and Instance Fields

```jsx
// BEFORE
class Form extends React.Component {
  inputRef = React.createRef();
  pollTimer = null;

  componentDidMount() {
    this.inputRef.current.focus();
    this.pollTimer = setInterval(this.poll, 1000);
  }
  componentWillUnmount() {
    clearInterval(this.pollTimer);
  }
}

// AFTER
const Form = () => {
  const inputRef = useRef(null);
  const pollTimerRef = useRef(null);

  useEffect(() => {
    inputRef.current.focus();
    pollTimerRef.current = setInterval(poll, 1000);
    return () => clearInterval(pollTimerRef.current);
  }, [poll]);
};
```

Use `useRef` for any value that must persist across renders without triggering re-renders (DOM nodes, timers, subscriptions, mutable counters).

### Class Methods

Plain helpers become arrow functions inside the component. Wrap with `useCallback` only when:

- The function is passed to a memoized child (`React.memo`)
- The function is a dependency of another hook
- Identity stability matters for an effect cleanup

```jsx
// BEFORE
handleSubmit = (event) => { ... };

// AFTER (no memoization needed)
const handleSubmit = (event) => { ... };

// AFTER (passed to memoized child)
const handleSubmit = useCallback((event) => { ... }, [userId]);
```

### Derived Values

```jsx
// BEFORE
render() {
  const sorted = this.props.items.slice().sort(byDate);
  return <List items={sorted} />;
}

// AFTER
const sorted = useMemo(
  () => items.slice().sort(byDate),
  [items],
);
return <List items={sorted} />;
```

Only memoize when the computation is measurably expensive or stable identity is required downstream. Do not blanket-wrap every derived value in `useMemo`.

### HOCs and Context

| Class pattern                        | Hook replacement                          |
| ------------------------------------ | ----------------------------------------- |
| `connect(mapState, mapDispatch)`     | `useSelector` + `useDispatch`             |
| `withRouter`                         | `useNavigate`, `useParams`, `useLocation` |
| `static contextType = MyContext`     | `const value = useContext(MyContext)`     |
| `withTranslation()`                  | `useTranslation()`                        |
| `withTheme` / `withStyles`           | `useTheme()` / styled API                 |

Drop the HOC wrapper from the export once each consumer is rewritten as a hook.

### forwardRef and Imperative Handles

```jsx
// BEFORE
class TextInput extends React.Component {
  inputRef = React.createRef();
  focus() { this.inputRef.current.focus(); }
  render() { return <input ref={this.inputRef} {...this.props} />; }
}

// AFTER
const TextInput = forwardRef((props, ref) => {
  const inputRef = useRef(null);
  useImperativeHandle(ref, () => ({
    focus: () => inputRef.current.focus(),
  }));
  return <input ref={inputRef} {...props} />;
});
```

## Workflow

### Step 1 — Audit

Run the [Pre-Migration Audit](#pre-migration-audit). Note any audit item with no obvious hook mapping (for example `componentDidCatch`) and call it out before editing.

### Step 2 — Convert

Rewrite the class as an arrow-function component that:

- Preserves the props contract, default exports, and JSX output
- Replaces every audited construct using the [Conversion Map](#conversion-map)
- Keeps method order roughly aligned with the original for easier diffing

Do not extract sub-components or new hooks during this step.

### Step 3 — Correctness Pass

Walk through the converted component asking:

- Does every `useEffect` declare every reactive value it reads in the deps array?
- Are all cleanup paths covered (subscriptions, timers, listeners, abort controllers)?
- Are setters that read previous state using the functional updater form?
- Are stale closures possible? If a callback captures state but is registered once, use a ref or move state into the callback.
- Does any effect run on every render because a non-memoized object or function is in the deps?

### Step 4 — Performance Pass

- Wrap callbacks passed to memoized children in `useCallback`
- Wrap genuinely expensive derivations in `useMemo`
- Replace cascading `useEffect` chains with a single async function or a query library where appropriate
- Parallelize independent async calls with `Promise.all` instead of awaiting in series
- Add `React.memo` to the export only if measured re-renders are a problem

### Step 5 — Maintainability Pass

- Extract reusable logic into custom hooks (`useFooData`, `useDebouncedValue`)
- Split the component if it now exceeds the project's file-length budget
- Move static helpers outside the component to avoid recreating them each render
- Confirm naming, prop types, and exports match the rest of the codebase

### Step 6 — Validate

- No `class`, `this`, `setState`, or `componentXxx` references remain
- Run the project's lint, type check, and test suite
- Manually exercise the component for any behavior the test suite does not cover (focus management, scroll restoration, animation timing)

## Common Pitfalls

### Stale Closures

```jsx
// BUG: count is captured once and never updates
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, []);

// FIX: include count, or use the functional setter form
useEffect(() => {
  const id = setInterval(() => console.log(count), 1000);
  return () => clearInterval(id);
}, [count]);
```

### Missing Effect Dependencies

The lint rule `react-hooks/exhaustive-deps` exists for a reason. If a value is in the dependency array because the linter asked, do not silence it without understanding why — that is the most common source of post-migration bugs.

### Object and Function Identity Churn

```jsx
// BUG: options is a new object every render, so the effect runs every render
const options = { signal: controller.signal };
useEffect(() => fetchData(options), [options]);

// FIX: memoize or inline the dependency
useEffect(() => fetchData({ signal: controller.signal }), [controller]);
```

### Using `useEffect` for Derived State

```jsx
// BUG: an effect that only sets state from props produces an extra render
useEffect(() => setFullName(`${first} ${last}`), [first, last]);

// FIX: derive during render
const fullName = `${first} ${last}`;
```

### Treating `useRef` Like State

A ref change does not trigger a re-render. Use it only for values the render output does not depend on. Anything visible in the JSX should live in `useState` (or `useReducer`).

### Forgetting Cleanup

Subscriptions, timers, IntersectionObservers, AbortControllers — all of them must be torn down in the effect's cleanup function. A converted component without cleanup leaks just like the class version did.

## Output Expectations

When this skill finishes, deliver:

1. The converted functional component
2. A short mapping summary listing each class concept and the hook that replaced it
3. A list of behavior-critical risks or assumptions made during conversion (especially for ambiguous lifecycle behavior)
4. Optional follow-up suggestions — extracting a custom hook, splitting the component, replacing manual data fetching with a query library

## Constraints

- Do not modify unrelated modules
- Do not introduce TypeScript into a JavaScript file (or vice versa)
- Do not remove existing functionality, even if it appears unused
- Do not change API payload shapes, business rules, or rendered output
- Do not silently alter prop names, default values, or exported identifiers
