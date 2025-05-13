## 6. Advanced Patterns

### 6.1 Optimistic Updates

Implementing responsive UIs with optimistic updates:

- **Apply changes locally first**: Update UI before server confirmation.
- **Track in-flight commands**: Use tokens to track command state.
- **Reconcile server responses**: Merge server updates with optimistic ones.
- **Handle failures gracefully**: Provide visual feedback for failed operations.
- **Use with-cycle for edit loops**: Loop in-flight edits through the component.

```clojure
;; Optimistic todo creation
(e/defn TodoList []
  (let [server-todos (e/server (e/watch !todos))
        !local-edits (atom {})
        all-todos (concat server-todos 
                          (vals (e/watch !local-edits)))]
    (dom/div
      (e/for-by :id [todo all-todos]
        (TodoItem todo))
      (dom/form
        (dom/input (dom/On "submit" 
          (fn [e]
            (.preventDefault e)
            (let [value (-> e .-target .-elements .-todo .-value)
                  temp-id (random-uuid)
                  temp-todo {:id temp-id :text value :status :pending}]
              ;; Add optimistic todo
              (swap! !local-edits assoc temp-id temp-todo)
              ;; Send to server
              (e/server
                (let [result (db/add-todo! value)]
                  ;; On success, remove temporary todo
                  (e/client
                    (swap! !local-edits dissoc temp-id)))))))))))))
```

### 6.2 Error Handling

Robust error handling in Electric applications:

- *try..catch is not yet supported in Electric v3. You can only `try..catch` in Clojure-land functions.
- **Provide fallback UI**: Show useful fallback content when errors occur.
- **Log errors server-side**: Record errors for debugging.
- **Distinguish between different error types**: Handle different errors appropriately.
- **Retry mechanisms**: Implement retry logic for transient failures.

### 6.3 Loading States and Suspense

Managing loading states effectively:

- **Show loading indicators**: Provide visual feedback during loading.
- **Use progressive loading**: Load and display content incrementally.
- **Skeleton screens**: Show skeleton UI while content loads.
- **Control loading granularity**: Balance between fine and coarse-grained loading states.
- **Loading timeouts**: Consider timeouts for loading indicators.

```clojure
;; Component with loading state
(e/defn AsyncContent [db query]
  (let [!loading? (atom true)
        result (e/server
                 (try
                   (let [data (slow-query db query)]
                     (e/client (reset! !loading? false))
                     data)
                   (catch Exception e
                     (e/client (reset! !loading? false))
                     nil)))]
    (e/if (e/watch !loading?)
      (LoadingSpinner)
      (if result
        (ContentView result)
        (ErrorMessage)))))
```

### 6.4 Custom Reactivity

Working with custom reactive patterns:

- **Custom event sources**: Create reactive sources from external events.
- **Manual reactivity with e/watch/e/write!**: Direct control when needed.
- **Interop with external reactive systems**: Adapter patterns for external reactivity.
- **Custom differential collections**: Build custom reactive collection abstractions.
- **Controlled reactivity flow**: Fine-tune when and how reactivity propagates.

```clojure
;; Creating a reactive source from external events
(e/defn ExternalEventSource []
  (let [!value (atom nil)]
    ;; Connect to external event source
    (js/externalAPI.onEvent #(reset! !value %))
    (e/on-unmount #(js/externalAPI.removeEventListener))
    
    ;; Return reactive value
    (e/watch !value)))

;; Using the external event source
(e/defn ExternalEventConsumer []
  (let [event-value (ExternalEventSource)]
    (dom/div (dom/text "Latest event: " event-value))))
```

