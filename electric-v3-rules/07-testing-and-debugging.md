## 7. Testing and Debugging Electric Clojure

### 7.1 Testing Electric Components

Strategies for testing Electric applications:

- **Unit test pure functions**: Extract and test pure logic separately.
- **Mock external dependencies**: Isolate components by mocking dependencies.
- **Test server components**: Test server-side logic with regular Clojure testing tools.
- **Test client components**: Use ClojureScript testing tools for client components.
- **Integration testing**: Test full client-server flows when necessary.

```clojure
;; Extracting pure functions for testing
(defn calculate-total [items]
  (reduce + (map :price items)))

;; Using in Electric component
(e/defn ShoppingCart [items]
  (let [total (calculate-total items)]
    (dom/div
      (dom/h2 (dom/text "Shopping Cart"))
      (e/for-by :id [item items]
        (CartItem item))
      (dom/div (dom/text "Total: $" total)))))

;; Test the pure function
(deftest calculate-total-test
  (is (= 100 (calculate-total [{:price 50} {:price 50}])))
  (is (= 0 (calculate-total []))))
```

### 7.2 Debugging Techniques

Effective debugging for Electric applications:

- **Use println/prn strategically**: Place print statements for visibility. These will only eval if their values changed.
- **Inspect reactive values**: Use `(prn (e/as-vec reactive-value))` for readable debug output.
- **Check transfer boundaries**: Confirm where code is running.
- **Browser DevTools**: Use browser tools for client-side debugging.
- **Server logs**: Check server logs for server-side issues.
- **Isolate components**: Debug issues by isolating components.

```clojure
;; Debug output for reactive values
(e/defn DebugComponent [data]
  (prn "Data:" (e/as-vec data))
  (dom/div
    (e/for [item data]
      (do
        (prn "Rendering item:" item)
        (dom/div (dom/text (str item)))))))
```

