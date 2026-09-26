# State Management & Client Data Flow Rules

> **Scope**: Client-side state models, server cache synchronization, URL state persistence, and reactivity.

---

## State Categorization Matrix
Frontend data MUST be strictly categorized into one of three distinct state domains:

| State Domain | Definition | Correct Management Mechanism | Prohibited Mechanism |
| :--- | :--- | :--- | :--- |
| **Server State** | Remote data owned by the backend (user profiles, product lists) | Dedicated async cache (TanStack Query, SWR, Apollo) | Global Redux / Zustand stores with manual dispatch |
| **Client UI State** | Ephemeral, local interface state (modal open, dropdown active) | Local component state (`useState`) or lightweight store | Persisting in global state or server databases |
| **Shareable View State** | Active filters, search queries, pagination, sorted column | URL search parameters (`?q=query&page=2&sort=asc`) | Hidden component state that vanishes on refresh |

---

## Rule STM-01: Separate Server Cache from Global Client State [MUST]
- **Rule**: Asynchronous server data (API responses) MUST NOT be copied into global client state managers (Redux, Zustand, Pinia) using manual fetch effects. Use dedicated asynchronous caching libraries (TanStack Query, SWR, RTK Query).
- **Reason**: Re-inventing caching in Redux requires manual deduplication, manual polling, manual stale-while-revalidate logic, and manual cache invalidation. It inevitably leads to stale UI bugs.
- **Risk**: Desynchronized views, race conditions during rapid re-renders, and hundreds of lines of boilerplate.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Manual Redux/Zustand fetching boilerplate
  useEffect(() => {
    fetch('/api/users').then(r => r.json()).then(users => setUsers(users));
  }, []);

  // ✅ CORRECT: TanStack Query manages deduplication, caching, and retry
  const { data: users, isLoading, error } = useQuery({
    queryKey: ['users'],
    queryFn: () => fetchUsers(),
    staleTime: 5 * 60 * 1000 // 5 minutes
  });
  ```
- **Exceptions**: Local offline-first applications utilizing SQLite (e.g. via WASM) in the browser.

---

## Rule STM-02: Persist Shareable Filter & Navigation State into URL Parameters [MUST]
- **Rule**: Search terms, active filters, page numbers, sorting preferences, and selected tabs MUST be persisted to URL query parameters (`searchParams`).
- **Reason**: Storing filter criteria in local component state means the view resets when the user refreshes the page, and prevents users from bookmarking or sending links to colleagues.
- **Risk**: Frustrated users who lose their filtered view upon page refresh or when clicking the browser "Back" button.
- **Correct Approach**:
  ```tsx
  // ✅ CORRECT: Synced directly to URL search parameters
  import { useSearchParams } from 'react-router-dom';

  function CustomerDirectory() {
    const [searchParams, setSearchParams] = useSearchParams();
    const query = searchParams.get('q') || '';
    const page = parseInt(searchParams.get('page') || '1', 10);

    const updateFilter = (newQuery: string) => {
      setSearchParams(prev => {
        if (newQuery) prev.set('q', newQuery);
        else prev.delete('q');
        prev.set('page', '1'); // Reset to page 1 on search change
        return prev;
      });
    };
    // ...
  }
  ```
- **Exceptions**: Sensitive search queries containing PII or passwords, which must never appear in URLs or browser history.

---

## Rule STM-03: Zero In-Place State Mutation [MUST]
- **Rule**: State variables MUST be treated as strictly immutable. Developers and AI agents MUST NOT use array mutating methods (`push`, `pop`, `splice`, `reverse`, `sort`) or object property assignments (`state.prop = value`) directly on state variables.
- **Reason**: React and modern declarative frameworks rely on shallow referential equality checks (`prev !== next`) to trigger re-renders. Mutating an object in place preserves the same memory reference, causing the framework to skip rendering.
- **Risk**: Ghost UI states where the internal data changed but the screen does not update.
- **Correct Approach**:
  ```typescript
  // ❌ BAD: Direct mutation
  const handleAdd = (item: string) => {
    items.push(item);
    setItems(items); // Same reference! React skips re-render!
  };

  // ✅ CORRECT: New immutable reference via spread operator
  const handleAdd = (item: string) => {
    setItems(prev => [...prev, item]);
  };
  ```
- **Exceptions**: Stores wrapped in **Immer** (e.g., Zustand `immer` middleware or Redux Toolkit `createSlice`) where direct mutations are safely proxied into immutable patches.

---

## Rule STM-04: Derive Values During Render; Avoid Synced Effect State [MUST]
- **Rule**: Values that can be calculated from existing state or props MUST be computed in-line during rendering (or memoized with `useMemo`), rather than synchronized into a second state variable via `useEffect`.
- **Reason**: Synchronizing state via effects creates a second render cycle, introduces temporary visual inconsistencies, and invites infinite render loops.
- **Risk**: Desynchronized counters, sluggish rendering, and high CPU usage.
- **Correct Approach**:
  ```tsx
  // ❌ BAD: Synchronizing state via effect
  const [items, setItems] = useState<Item[]>([]);
  const [totalPrice, setTotalPrice] = useState<number>(0);
  useEffect(() => {
    setTotalPrice(items.reduce((sum, i) => sum + i.price, 0));
  }, [items]);

  // ✅ CORRECT: Inline derived calculation (Instant, zero lag)
  const totalPrice = useMemo(
    () => items.reduce((sum, i) => sum + i.price, 0),
    [items]
  );
  ```
- **Exceptions**: Heavy async transformations requiring worker thread calculation.
