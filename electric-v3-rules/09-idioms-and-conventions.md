## 9. Idioms and Conventions

### 9.1 Naming and Style

Consistent naming and style practices:

- **Use e/defn compiler macro for Electric functions**: Clearly indicate Electric functions.
- **Use ! prefix for stateful values like atoms**: Indicate reference types with `!`, e.g. `!counter`.
- **Use ! suffix for mutation functions**: Mark functions with side effects, e.g. `save-user!`.
- **Use ? suffix for predicates**: Follow standard Clojure conventions.
- **Clear, descriptive component names**: Use specific, meaningful names.

```clojure
;; Good naming examples
#?(:clj (defonce !app-state (atom {}))) ;; Atom with ! prefix
(e/defn UserProfile [user-id]) ;; Clear component name
(defn valid-email? [email]) ;; Predicate with ? suffix
(defn save-user! [user]) ;; Mutation with ! suffix
```

### 9.2 Common Patterns and Idioms

Recurring patterns in Electric code:

- **e/for-by for keyed collections**: Always use keys for stable identity.
- **e/server/e/client for transfer boundaries**: Keep boundaries clear and explicit.
- **e/watch for connecting atoms**: Standard way to make atoms reactive.
- **e/watch + UI = controlled component**: Pattern for controlled inputs.
- **e/on-unmount for cleanup**: Standard cleanup mechanism.

```clojure
;; Common idioms
(e/defn DataList [items]
  (dom/ul
    (e/for-by :id [item items] ;; Keyed iteration
      (dom/li (dom/text (:name item))))))

(e/defn StateComponent []
  (let [state (e/watch !state)] ;; Connecting atom to reactivity
    (dom/div (dom/text "State: " state))))

(e/defn CleanupComponent []
  (let [resource (init-resource!)]
    (e/on-unmount #(cleanup-resource! resource)) ;; Cleanup on unmount
    (dom/div (dom/text "Resource: " resource))))
```