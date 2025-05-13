## 4. Client-Server Patterns

### 4.1 Transfer Boundaries

Understanding and managing transfer boundaries is crucial in Electric:

- **Platform interop forces site**: Interop with platform-specific code forces expressions to a specific site.
- **Values transfer, references don't**: Only serializable values can cross network boundaries.
- **Site determines where code runs**: The site of an expression determines where it executes.
- **Dynamic site inheritance**: Site is inherited through dynamic scope.
- **DOM operations are auto-sited**: Electric DOM macros auto-site their contents to the client.

```clojure
;; Server interop forces server site
(e/server (java.io.File. "example.txt")) 

;; Client interop forces client site
(e/client (js/alert "Hello"))

;; Value transfers across boundary
(e/client (dom/div (dom/text (e/server (db/get-username)))))

;; Reference doesn't transfer
(e/server (let [conn (db/connect!)] ;; conn stays on server
            (e/client (dom/div (dom/text "Connected")))))
```

### 4.2 Optimizing Network Traffic

Minimizing unnecessary network transfers:

- **Batch related queries**: Fetch related data in one operation.
- **Use e/server sparingly**: Don't create unnecessary round trips.
- **Transfer minimal data**: Only send what's needed across the network.
- **Use client-side filtering**: When possible, filter large datasets on the server.
- **Watch for accidental transfers**: Be careful about unintended data movement.

```clojure
;; Bad: Multiple round trips
(e/defn UserProfile [db user-id]
  (e/server
    (let [user    (db/get-user db user-id)
          posts   (db/get-user-posts db user-id) ;; Separate trip
          friends (db/get-user-friends db user-id)] ;; Separate trip
      (e/client (RenderUserProfile user posts friends)))))

;; Better: Batched query
(e/defn UserProfile [db user-id]
  (let [{:as user-data :keys [user posts friends]}
        (e/server (db/get-user-with-posts-and-friends db user-id))]
    (RenderUserProfile user posts friends)))
```

### 4.3 Authentication and Security

Security considerations for Electric applications:

- **Validate on server**: Always validate user input on the server.
- **Never trust client data**: All client-provided data must be validated.
- **Use server-side permissions**: Check permissions on the server before operations.
- **Secure sensitive operations**: Place security-sensitive code in `e/server` blocks.
- **Expose only necessary data**: Only send the client data it needs.

```clojure
;; Secure server-side validation
(e/defn UpdateUser [user-id user-data]
  (e/server
    (when (authorized? user-id) ;; Server-side permission check
      (when (valid-user-data? user-data) ;; Server-side validation
        (db/update-user! conn user-id user-data)))))

;; Only sending necessary data to client
(e/defn UserProfile [db user-id]
  (let [user (e/server 
               (-> (db/get-user db user-id)
                   (select-keys [:name :bio :public-info]))) ;; Exclude sensitive data
    (RenderUserProfile user)))
```

