## 1. Electric Clojure Philosophy and Fundamentals

### 1.1 Core Principles

#### Reactivity

Electric is fundamentally a reactive programming system, where changes to data automatically propagate through the application.

- **Values flowing through a DAG**: Electric programs are directed acyclic graphs (DAGs) of computations, with values flowing from sources to sinks.
- **Differential updates**: Only recompute what has changed, with fine-grained reactivity.
- **Signals, not streams**: Electric models signals (like mouse coordinates) rather than streams (like mouse clicks).
- **Work-skipping**: If an argument to a function hasn't changed, the function doesn't need to be recomputed.
- **Backpressure**: Electric automatically handles backpressure throughout the application, ensuring components aren't overwhelmed with data.

```clojure
;; A reactive timer that automatically updates
(e/defn Timer []
  (let [seconds (/ (e/System-time-ms) 1000)]
    (dom/div (dom/text "Time: " seconds))))
```

#### Client-Server Transfer

Electric allows for seamless client-server programming within the same codebase.

- **Transfer boundaries are explicit**: Use `e/server` and `e/client` to specify where code runs.
- **Values transfer, references don't**: Only serializable values can cross network boundaries.
- **Platform interop forces site**: Interop with platform-specific code (like DOM or Java) forces expressions to a specific site.
- **Edges aren't sited**: Shared bindings pass through function calls symbolically, not forcing transfer.
- **Dynamic siting**: Site inheritance follows dynamic scope in v3.

```clojure
;; Server-side query with client-side rendering
(e/defn UserProfile [user-id]
  (let [user-data (e/server (db/get-user user-id))]
    (e/client
      (dom/div
        (dom/h1 (dom/text (:name user-data)))
        (dom/p (dom/text (:bio user-data)))))))
```

#### Differential Collections

To send only changes over the wire, the Electric `e/for` looping construct expects a *differential collection*, calculated via `(e/diff-by predicate collection)`.

`(e/for-by :id [uid user-ids] ...)` is equivalent to `(e/for [uid (e/diff-by :id user-ids)] ...)`.

Prefer to name differential collections explicitly by prefixing with `diffed-`, e.g. `diffed-users`. 

#### Unified Programming Model

Electric unifies client and server code into a single program and mental model.

- **Single language across stack**: Write your entire application in Clojure(Script).
- **Shared syntax and semantics**: Same language constructs work on both client and server.
- **Automatic serialization**: Electric handles value transfer between environments.
- **Single-pass rendering**: Traverse and render server data directly to client DOM in one pass.
- **Bidirectional reactivity**: Changes flow in both directions through the DAG.

```clojure
;; Combining server data access with client input in one function
(e/defn SearchUsers []
  (let [search-term (e/client (dom/input (dom/On "input" #(-> % .-target .-value) "")))
        results (e/server 
                  (when (seq search-term)
                    (db/search-users search-term)))]
    (e/client
      (dom/div
        (dom/h2 (dom/text "Search Results"))
        (e/for [user results]
          (dom/div (dom/text (:name user))))))))
```

### 1.2 Electric vs. Traditional Clojure

Electric Clojure extends regular Clojure with reactive semantics, which creates some important differences:

- **Reactivity as default**: Every expression is potentially reactive, recalculating when inputs change.
- **Concurrent evaluation**: Expressions in different sites run concurrently, not sequentially.
- **Tables (amb) vs. collections**: Electric uses tables (amb values) as its primary collection abstraction.
- **Differential evaluation**: Functions are called on changes to their inputs, not just on initial evaluation.
- **Reactive lifecycles**: Components have mount/unmount lifecycle tied to their place in the DAG.
- **Transfer semantics**: Need to reason about what moves between client and server.

```clojure
;; In regular Clojure, this would print once
;; In Electric, it prints whenever x changes
(let [x (e/watch !state)]
  (println x))
```

