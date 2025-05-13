## 3. Reactive Programming Patterns

### 3.1 Working with Electric Tables and e/amb

Electric tables (created with `e/amb`) are fundamental to Electric's reactive model:

- **Use e/amb for concurrent values**: Tables hold multiple values in superposition.
- **Flattening semantics**: Nested `e/amb` calls flatten automatically.
- **Auto-mapping of functions**: Functions automatically map over values in tables.
- **Use e/for for iteration**: When working with tables, use `e/for` for proper isolation.
- **Materialization with e/as-vec**: Convert Electric tables to Clojure vectors with `e/as-vec`.
- **Understand cartesian products**: Be careful with operations that create cross products of tables.

```clojure
;; Creating a table with multiple values
(e/amb 1 2 3) ;; Table with three values in superposition

;; Auto-mapping
(inc (e/amb 1 2 3)) ;; Results in (e/amb 2 3 4)

;; Using e/for for iteration and isolation
(e/for [x (e/amb 1 2 3)]
  (dom/div (dom/text x))) ;; Creates three separate divs

;; Converting to a regular vector
(e/as-vec (e/amb 1 2 3)) ;; [1 2 3]
```

### 3.2 Reactive Flow Control

Control flow in Electric requires special consideration due to reactivity:

- **Use e/for for collection iteration**: Ensures proper lifecycle management for items.
- **Prefer e/for over if with tables**: `e/for` provides better semantics with non-singular values.
- **Use e/for-by for keyed collections**: When items have stable identities, use `e/for-by`.
- **Use e/when for conditional rendering**: For simple cases with a single condition.
- **Careful with product semantics**: Be aware of unintended product semantics with if/when and tables.

```clojure
;; Conditional rendering with e/when
(e/when logged-in?
  (UserDashboard db user-id))

;; Iteration with stable identity
(e/for-by :id [user users]
  (UserCard db user))

;; Avoiding cartesian products with if
(e/for [x (e/amb 1 2 3)]
  (e/when (odd? x)
    (dom/div (dom/text x))))
```

### 3.3 Working with Events

Events in Electric are handled through tokens and event flow:

- **Use dom/On for simple events**: Captures DOM events and their values.
- **Use dom/On-all for concurrent events**: Handles multiple events without waiting for previous ones to complete.
- **Token-based event tracking**: Events return `[t v]` pairs for lifecycle tracking.
- **Command patterns for state changes**: Use tokens to manage command state like pending/complete.

```clojure
;; Simple event handling
(let [value (dom/input (dom/On "input" #(-> % .-target .-value) ""))]
  (dom/div (dom/text "You typed: " value)))

;; Concurrent events with command tracking
(e/for [[t e] (dom/On-all "click")]
  (let [res (e/server (process-click!))]
    (case res
      ::ok (t) ;; Mark command as completed
      ::error (prn "Error!"))))
```

### 3.4 Managing Side Effects

Side effects require careful handling in a reactive system:

- **Use e/on-unmount for cleanup**: Register cleanup functions when components unmount.
- **Mark effects with !**: Use `!` suffix for functions with side effects.
- **Isolate effects from pure logic**: Keep side effects separate from pure computation.
- **Use e/Offload for blocking operations**: Run blocking operations without stalling the reactive system.

```clojure
;; Component with cleanup
(e/defn ResourceComponent []
  (let [resource (e/server (open-resource!))]
    (e/on-unmount #(e/server (close-resource! resource)))
    (dom/div (dom/text "Using resource: " (resource-name resource)))))

;; Offloading blocking work
(e/defn SlowOperation []
  (let [result (e/server (e/Offload #(do-expensive-calculation)))]
    (dom/div (dom/text "Result: " result))))
```

