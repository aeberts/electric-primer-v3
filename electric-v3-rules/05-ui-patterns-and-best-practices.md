## 5. UI Patterns and Best Practices

### 5.1 DOM Manipulation

Effective DOM manipulation in Electric:

- **Use electric-dom3 macros**: Leverage the reactive DOM API.
- **Props for attributes**: Use `dom/props` for element attributes.
- **Dynamic text with dom/text**: Create reactive text nodes.
- **Event handling with dom/On**: Capture DOM events reactively.
- **Element reference with dom/node**: Access the DOM element for imperative operations.

```clojure
;; Complete DOM example
(e/defn Button [label on-click]
  (dom/button
    (dom/props {:class "btn btn-primary"
                :disabled (nil? on-click)})
    (dom/On "click" (fn [e] 
                      (.preventDefault e)
                      (when on-click (on-click))))
    (dom/text label)))
```

### 5.2 Forms and User Input

Building interactive forms in Electric:

- **Controlled inputs**: Use reactive values to control input state.
- **Form validation**: Validate forms on both client and server.
- **Input debouncing**: Debounce rapidly changing inputs when necessary.
- **Multi-step forms**: Break complex forms into manageable stages.
- **Optimistic updates**: Provide immediate feedback before server confirmation.

```clojure
;; Simple controlled input
(e/defn InputField [label value on-change]
  (dom/div
    (dom/label (dom/text label))
    (dom/input
      (dom/props {:value value
                 :type "text"})
      (dom/On "input" #(on-change (-> % .-target .-value))))))

;; Form with validation
(e/defn LoginForm []
  (let [!email    (atom "")
        !password (atom "")
        errors (e/server
                 (cond-> {}
                   (not (valid-email? (e/watch !email)))
                   (assoc :email "Invalid email")
                   
                   (< (count (e/watch !password)) 8)
                   (assoc :password "Password too short")))]
    (dom/form
      (InputField "Email" (e/watch !email) #(reset! !email %))
      (when (:email errors)
        (dom/div (dom/props {:class "error"}) (dom/text (:email errors))))
      
      (InputField "Password" (e/watch !password) #(reset! !password %))
      (when (:password errors)
        (dom/div (dom/props {:class "error"}) (dom/text (:password errors))))
      
      (dom/button
        (dom/props {:type "submit"
                   :disabled (not-empty errors)})
        (dom/text "Login")))))
```

### 5.3 List Rendering and Collection Handling

Efficiently rendering collections:

- **Use e/for-by for keyed collections**: Provide stable identity for efficient updates.
- **Process collections server-side**: Sort, filter, and limit collections on the server when possible.
- **Virtual rendering for large lists**: Consider virtual rendering for very large collections.
- **Paginated loading**: Implement pagination for large datasets.
- **Optimistic collection updates**: Update collections optimistically for better UX.

```clojure
;; Efficient list rendering with keyed iteration
(e/defn UserList [users]
  (dom/ul
    (e/for-by :id [user users]
      (dom/li
        (dom/text (:name user))))))

;; Server-side processing before rendering
(e/defn SearchResults [db query]
  (let [results (e/server
                  (-> (db/search db query)
                      (sort-by :relevance)
                      (take 20)))]
    (dom/div
      (dom/h2 (dom/text "Results for: " query))
      (UserList results))))
```

### 5.4 Performance Optimization

Techniques for optimizing Electric UI performance:

- **Minimize state changes**: Only update state when necessary.
- **Use e/memo for expensive calculations**: Memoize results of expensive computations.
- **Control reactivity granularity**: Balance between too fine and too coarse reactivity.
- **Implement virtualization**: Use virtual rendering for large lists.
- **Profile and optimize**: Use browser dev tools to identify bottlenecks.

```clojure
;; Memoizing expensive calculations
(e/defn ExpensiveCalculation [data]
  (e/memo
    (slow-calculation data)))

;; Virtualized list rendering (simplified)
(e/defn VirtualList [items]
  (let [visible-range (calculate-visible-range)]
    (dom/div
      (dom/props {:style {:height (str (* (count items) item-height) "px")}})
      (e/for-by :id [item (subvec items (:start visible-range) (:end visible-range))]
        (dom/div
          (dom/props {:style {:position "absolute"
                             :top (str (* (:index item) item-height) "px")}})
          (RenderItem item))))))
```

