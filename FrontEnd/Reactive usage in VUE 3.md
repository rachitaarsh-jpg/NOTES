# Vue 3 Composition API - `reactive()` vs `ref()`

## What is Reactivity?

Reactivity is one of Vue's core features.

> **Reactivity means whenever the data changes, Vue automatically updates everything that depends on it (UI, computed properties, watchers, etc.).**

Instead of manually updating the DOM, Vue tracks your data and updates the UI whenever that data changes.

---

# Without Reactivity

```javascript
let name = "John";

name = "Rachit";
```

The variable changes, but Vue has no way of knowing that it changed.

If the UI was displaying:

```
John
```

It will continue displaying:

```
John
```

unless we manually update it.

---

# With Reactivity

Vue provides reactive APIs like:

```javascript
reactive()
```

or

```javascript
ref()
```

These APIs allow Vue to track changes automatically.

---

# `reactive()`

`reactive()` is used to make **objects** reactive.

Example:

```javascript
const person = reactive({
    name: "John",
    age: 25
});
```

Now Vue watches every property inside this object.

When you update:

```javascript
person.name = "Mike";
```

Vue immediately knows that `name` has changed and updates:

- UI
- Watchers
- Computed properties

without writing any DOM manipulation code.

---

## How `reactive()` Works Internally

Internally Vue wraps your object inside a **JavaScript Proxy**.

```
Your Object
      │
      ▼
   Vue Proxy
      │
Vue tracks every property
```

Whenever any property changes, Vue detects it automatically.

---

# Example

```javascript
const person = reactive({
    name: "John",
    age: 25
});
```

Template:

```vue
<p>{{ person.name }}</p>
```

Initially:

```
John
```

Later:

```javascript
person.name = "Mike";
```

The browser automatically updates to:

```
Mike
```

No manual refresh is required.

---

# Understanding the Given Code

```typescript
const form = reactive({
    username: {
        label: "Username",
        type: "text"
    },

    password: {
        label: "Password",
        type: "password"
    },

    description: {
        label: "Description",
        type: "textarea"
    }
});
```

Here `form` is one large object containing multiple fields.

Structure:

```
form
│
├── username
├── password
├── description
└── ...
```

Since the object is reactive, Vue tracks every property.

Example:

```javascript
form.username.label = "Login Username";
```

Vue instantly updates every place where

```vue
{{ form.username.label }}
```

is being displayed.

---

# Adding New Properties

Reactive objects can also receive new properties.

```javascript
form.email = {
    label: "Email",
    type: "text"
};
```

Vue also tracks these newly added properties.

---

# `ref()`

`ref()` is generally used for **single values** such as:

- String
- Number
- Boolean

Example:

```javascript
const count = ref(0);
```

Reading value:

```javascript
count.value
```

Updating value:

```javascript
count.value++;
```

Notice the use of `.value`.

---

# Why `.value`?

A `ref()` wraps a single value inside an object.

Conceptually:

```javascript
const count = {
    value: 0
};
```

So:

```javascript
count.value++;
```

---

# Why Doesn't `reactive()` Use `.value`?

Example:

```javascript
const user = reactive({
    name: "John",
    age: 25
});
```

Access directly:

```javascript
user.name
```

Update directly:

```javascript
user.name = "Mike";
```

No `.value` is required because the object itself is already reactive.

---

# Your Code

## Using `ref()`

```typescript
const selectedStatusId = ref("");
```

This stores **one string value**.

Later:

```javascript
selectedStatusId.value = "ACTIVE";
```

Since it is a single value, `ref()` is the correct choice.

---

## Using `reactive()`

```typescript
const form = reactive({
    ...
});
```

This contains many related properties.

```
form
├── username
├── password
├── description
├── receiveUrl
├── sendUrl
├── remoteId
└── ...
```

Using `reactive()` makes working with object properties much cleaner.

Instead of:

```javascript
form.value.username
form.value.password
```

You simply write:

```javascript
form.username
form.password
```

---

# Why Not Use `ref()` for Objects?

You can.

```javascript
const form = ref({
    username: "",
    password: ""
});
```

But then every access becomes:

```javascript
form.value.username
```

Whereas with `reactive()`:

```javascript
form.username
```

This is cleaner and easier to read.

---

# Understanding `Record<string, any>`

Your code:

```typescript
reactive<Record<string, any>>(...)
```

This is a TypeScript type.

It means:

- Keys are strings.
- Values can be of any type.

Equivalent to:

```typescript
type Form = {
    [key: string]: any;
};
```

So these are all valid:

```javascript
form.username
form.password
form.description
```

---

# `ref()` vs `reactive()`

| Feature | `ref()` | `reactive()` |
|----------|----------|---------------|
| Best for | Primitive values | Objects |
| Access | `value` | Direct property access |
| Update | `count.value = 5` | `user.name = "Mike"` |
| Uses Proxy | Yes | Yes |
| Common use | Single values | Forms, state objects, configurations |

---

# When to Use Which?

### Use `ref()` when:

- Managing a single value.
- String
- Number
- Boolean

Example:

```javascript
const isLoading = ref(false);
const username = ref("");
const age = ref(25);
```

---

### Use `reactive()` when:

- Managing multiple related values.
- Forms
- State objects
- Configuration objects

Example:

```javascript
const user = reactive({
    name: "",
    age: 0,
    email: ""
});
```

---

# Real-Life Analogy

Imagine a hospital registration system.

Without reactivity:

```
Receptionist changes patient's name

↓

Every department must be informed manually.
```

With Vue's reactivity:

```
Receptionist
      │
      ▼
Reactive Object
      │
      ├── Billing updated
      ├── Doctor updated
      ├── Pharmacy updated
      └── UI updated automatically
```

One change automatically propagates everywhere.

---

# Key Takeaways

- Vue's reactivity automatically updates the UI when data changes.
- `reactive()` is best suited for objects.
- `ref()` is best suited for primitive values.
- `reactive()` allows direct property access (`user.name`).
- `ref()` requires using `.value`.
- Vue internally uses JavaScript **Proxy** objects to track changes.
- `Record<string, any>` is a TypeScript type indicating an object with string keys and values of any type.
- Forms, configuration objects, and application state are commonly implemented using `reactive()`.
- Single values such as counters, IDs, and flags are typically implemented using `ref()`.

