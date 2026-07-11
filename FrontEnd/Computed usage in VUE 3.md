# Vue 3 Composition API - `computed()`

## What is `computed()`?

A **computed property** is a reactive value that is **derived from other reactive data**.

> **`computed()` automatically recalculates its value whenever its dependencies change.**

Instead of manually updating a variable every time another value changes, Vue does it for you.

---

# Why Do We Need `computed()`?

Suppose you have:

```javascript
const firstName = ref("John");
const lastName = ref("Doe");
```

You want the full name.

Without `computed()`:

```javascript
let fullName = firstName.value + " " + lastName.value;
```

Now if you do:

```javascript
firstName.value = "Rachit";
```

`fullName` is still:

```
John Doe
```

because it was calculated only once.

You would have to manually update it:

```javascript
fullName = firstName.value + " " + lastName.value;
```

This quickly becomes tedious.

---

# Using `computed()`

```javascript
const fullName = computed(() => {
    return firstName.value + " " + lastName.value;
});
```

Now Vue watches both:

- `firstName`
- `lastName`

Whenever either changes, `fullName` is automatically recalculated.

Example:

Initially:

```javascript
firstName.value = "John";
lastName.value = "Doe";
```

```
fullName = "John Doe"
```

Later:

```javascript
firstName.value = "Rachit";
```

Automatically:

```
fullName = "Rachit Doe"
```

No manual update required.

---

# Syntax

```javascript
const variable = computed(() => {
    return someCalculation;
});
```

The function passed to `computed()` is called the **getter**.

Vue executes this function whenever one of its reactive dependencies changes.

---

# Your Code

```typescript
const isCreateMode = computed(() => !props.id);
```

Let's understand it step by step.

---

## Step 1

```typescript
props.id
```

Suppose the component receives:

```javascript
props = {
    id: "123"
}
```

or

```javascript
props = {
    id: undefined
}
```

---

## Step 2

The `!` operator means **NOT**.

Examples:

```javascript
!"123"      // false
!1          // false
!true       // false

!undefined  // true
!null       // true
!""         // true
```

---

## Step 3

So:

```javascript
!props.id
```

means:

> "Return `true` if there is no `id`."

---

# What Does It Represent?

```typescript
const isCreateMode = computed(() => !props.id);
```

is creating a boolean value.

If:

```javascript
props.id = undefined
```

then:

```javascript
isCreateMode.value
```

is

```javascript
true
```

Meaning:

```
We are creating a new record.
```

If:

```javascript
props.id = "10001"
```

then:

```javascript
isCreateMode.value
```

becomes

```javascript
false
```

Meaning:

```
We are editing an existing record.
```

---

# Visual Flow

```
props.id
     │
     ▼
computed(() => !props.id)
     │
     ▼
isCreateMode
```

If `props.id` changes, Vue automatically recalculates `isCreateMode`.

---

# Why Use `computed()` Instead of a Normal Variable?

Suppose you wrote:

```javascript
const isCreateMode = !props.id;
```

This runs only once.

If later:

```javascript
props.id = "500";
```

`isCreateMode` **does not update**.

With:

```javascript
const isCreateMode = computed(() => !props.id);
```

Vue keeps it synchronized automatically.

---

# Computed Properties Are Cached

One of the biggest advantages of `computed()` is **caching**.

Example:

```javascript
const total = computed(() => {
    console.log("Calculating...");
    return price.value * quantity.value;
});
```

Suppose:

```javascript
price = 100
quantity = 5
```

First access:

```javascript
total.value
```

Output:

```
Calculating...
500
```

Second access:

```javascript
total.value
```

Output:

```
500
```

Notice:

```
"Calculating..."
```

does **not** print again.

Vue remembers the result.

Only when:

```javascript
price.value = 200;
```

does Vue recalculate.

---

# `computed()` vs Function

Function:

```javascript
function fullName() {
    return firstName.value + " " + lastName.value;
}
```

Every call:

```javascript
fullName();
fullName();
fullName();
```

runs the calculation again.

---

Computed:

```javascript
const fullName = computed(() => {
    return firstName.value + " " + lastName.value;
});
```

Vue calculates once and reuses the cached result until a dependency changes.

---

# `computed()` vs `watch()`

| `computed()` | `watch()` |
|--------------|-----------|
| Returns a new reactive value | Performs a side effect |
| Cached | Not cached |
| Used for derived data | Used to react to changes (API calls, logging, navigation, etc.) |

Example:

```javascript
const fullName = computed(() => {
    return firstName.value + " " + lastName.value;
});
```

creates a value.

Whereas:

```javascript
watch(firstName, () => {
    console.log("First name changed");
});
```

doesn't create a value—it just performs an action when `firstName` changes.

---

# Real-Life Analogy

Imagine a shopping cart.

```
Price = ₹100
Quantity = 3
```

The total is always:

```
₹300
```

Instead of updating the total manually every time the price or quantity changes, Vue computes it automatically.

```
Price
      \
       \
        ----> Total
       /
Quantity
```

If either input changes, the total updates automatically.

---

# Key Takeaways

- `computed()` creates a **derived reactive value**.
- It automatically updates whenever its reactive dependencies change.
- It is **cached**, so Vue doesn't recompute it unless necessary.
- Use it when one value depends on another.
- In your code:

```typescript
const isCreateMode = computed(() => !props.id);
```

means:

- If `props.id` exists → `isCreateMode` is `false` (Edit Mode).
- If `props.id` doesn't exist → `isCreateMode` is `true` (Create Mode).

This is a common pattern in forms where the same component is used for both creating a new record and editing an existing one.
