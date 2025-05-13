## 2. Code Organization and Structure

### 2.1 Namespace Organization

1. **One component per namespace**: For complex components, define them in their own namespace.
2. **Group related components**: Keep related components in the same namespace.
3. **Reuse ns conventions**: Follow standard Clojure namespace conventions.
4. **Separate client/server only code**: Code that runs exclusively on one platform should be in separate namespaces with appropriate reader conditionals.
5. **Use cljc for shared code**: Write most Electric code in .cljc files for cross-platform compatibility.

```clojure
(ns my-app.user-profile
  (:require [hyperfiddle.electric3 :as e]
            [hyperfiddle.electric-dom3 :as dom]
            #?(:clj [my-app.db :as db]) ; server-side only dependency
            #?(:cljs [my-app.ui.client :as ui-client]) ; client-side only dependency
            [my-app.ui.components :as ui])) ; shared UI components
```

### 2.2 Component Structure

1. **Single responsibility**: Components should have a clear, focused purpose.
2. **Explicit transfer boundaries**: Make client/server boundaries clear and deliberate.
3. **Keep components small**: Break large components into smaller, composable pieces.
4. **Clear input/output contract**: Define what data flows in and out of components.
5. **Pure rendering**: Keep rendering logic separate from data fetching when possible.

```clojure
;; Good: component with clear responsibilities
(e/defn UserCard [db user-id]
  (let [user (e/server (db/get-user db user-id))]
    (e/client
      (dom/div (dom/props {:class "user-card"})
        (dom/h2 (dom/text (:name user)))
        (dom/p (dom/text (:email user)))))))

;; Better: separated data and UI components
(e/defn UserData [db user-id]
  (e/server (db/get-user db user-id)))

(e/defn UserCard [db user-id]
  (let [user (UserData db user-id)]
    (dom/div (dom/props {:class "user-card"})
      (dom/h2 (dom/text (:name user)))
      (dom/p (dom/text (:email user))))))
```

### 2.3 State Management

1. **Server state in atoms**: Use atoms for shared server-side state.
2. **Reactive state with e/watch**: Connect atoms to the reactive system with `e/watch`.
3. **Optimize state granularity**: Balance between too many small atoms and too few large atoms.
4. **Shared vs. local state**: Decide deliberately what state should be global vs. component-local.
5. **Client-side state**: Use atoms for UI-only state that doesn't need server persistence.

```clojure
;; Server-side shared state
#?(:clj (defonce !app-state (atom {:users {} :settings {}})))

;; Reactive component using shared state
(e/defn UserList []
  (e/server
    (let [users        (:users (e/watch !app-state))
          diffed-users (e/diff-by :id users)]
     (e/client
       (dom/div
         (e/for [user diffed-users]
           (UserCard (:id user))))))))
```

