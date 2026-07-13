# Understanding `reduce()` in JavaScript/TypeScript

## What is `reduce()`?

`reduce()` is an array method that **iterates over every element** of an array and **builds one final result**.

That final result can be:

- A Number
- A String
- An Array
- An Object

Think of it as:

```text
Start with an initial value
        │
        ▼
Process first element
        │
        ▼
Update accumulator
        │
        ▼
Process second element
        │
        ▼
Update accumulator
        │
        ▼
...
        │
        ▼
Return final accumulator
```

The accumulator keeps growing after every iteration.

---

# Simple Example

Suppose we have

```ts
const numbers = [1, 2, 3, 4];
```

Normally we'd calculate the sum like this:

```ts
let sum = 0;

for (const num of numbers) {
    sum += num;
}
```

Using `reduce()`:

```ts
const sum = numbers.reduce((total, num) => {
    return total + num;
}, 0);
```

Notice the `0` at the end.

```ts
.reduce(callback, 0)
```

This means:

> Start the accumulator (`total`) with 0.

### Execution Flow

|Iteration|Current Number|Accumulator Before|Return Value|Accumulator After|
|---------|--------------|-----------------|------------|----------------|
|Start|-|0|-|0|
|1|1|0|1|1|
|2|2|1|3|3|
|3|3|3|6|6|
|4|4|6|10|10|

Final result

```ts
sum = 10;
```

---

# Understanding `reduce()` in My Code

My code:

```ts
const payload = Object.entries(form).reduce(
    (params, [key, field]) => {
        ...
    },
    {}
);
```

The important part is the last argument.

```ts
{}
```

This tells `reduce()`:

> Start with an empty object.

Initially

```ts
params = {}
```

Think of `params` as the payload that is being built.

---

# Step 1 : Object.entries()

Suppose my form looks like this

```ts
const form = {
    description: {
        value: "FTP Server"
    },
    password: {
        value: null
    },
    hostName: {
        value: "192.168.1.10"
    }
};
```

When I do

```ts
Object.entries(form)
```

It converts the object into an array.

```ts
[
    ["description", { value: "FTP Server" }],
    ["password", { value: null }],
    ["hostName", { value: "192.168.1.10" }]
]
```

Now `reduce()` iterates over this array one item at a time.

---

# Step 2 : Understanding the Callback

```ts
(params, [key, field]) => {

}
```

`params`

- The accumulator.
- It stores the object being built.

`[key, field]`

Destructures each array element.

For example,

```ts
["description", { value: "FTP Server" }]
```

becomes

```ts
key = "description"

field = {
    value: "FTP Server"
}
```

---

# Step 3 : Initial State

```ts
params = {}
```

Nothing has been added yet.

---

# Iteration 1

Current Entry

```ts
["description", { value: "FTP Server" }]
```

After destructuring

```ts
key = "description"

field.value = "FTP Server"
```

### First Condition

```ts
if (
    CREDENTIAL_FIELDS.has(key) &&
    field.value === null
)
```

Evaluate

```ts
CREDENTIAL_FIELDS.has("description")
```

returns

```text
false
```

Condition fails.

---

### Second Condition

```ts
if (
    field.value !== null &&
    field.value !== undefined
)
```

Evaluate

```ts
"FTP Server" !== null
```

↓

true

and

```ts
"FTP Server" !== undefined
```

↓

true

Execute

```ts
params["description"] = "FTP Server";
```

Now

```ts
params = {
    description: "FTP Server"
}
```

Return

```ts
return params;
```

This returned object becomes the accumulator for the next iteration.

---

# Iteration 2

Current Entry

```ts
["password", { value: null }]
```

Current accumulator

```ts
params = {
    description: "FTP Server"
}
```

Notice

The accumulator is **not recreated**.

It still contains the previous data.

---

Destructuring

```ts
key = "password"

field.value = null
```

### First Condition

```ts
CREDENTIAL_FIELDS.has("password")
```

↓

true

```ts
field.value === null
```

↓

true

Whole condition

```ts
true && true
```

↓

true

Execute

```ts
return params;
```

Nothing is added.

Accumulator remains

```ts
params = {
    description: "FTP Server"
}
```

---

# Iteration 3

Current Entry

```ts
["hostName", { value: "192.168.1.10" }]
```

Current accumulator

```ts
params = {
    description: "FTP Server"
}
```

First condition

```text
false
```

Second condition

```text
true
```

Execute

```ts
params["hostName"] = "192.168.1.10";
```

Accumulator becomes

```ts
params = {
    description: "FTP Server",
    hostName: "192.168.1.10"
}
```

Return it.

---

# Loop Ends

Reduce returns

```ts
{
    description: "FTP Server",
    hostName: "192.168.1.10"
}
```

Therefore

```ts
const payload = {
    description: "FTP Server",
    hostName: "192.168.1.10"
}
```

---

# Visual Flow

```text
Initial

params = {}

        │
        ▼

Iteration 1

description

params["description"] = "FTP Server"

↓

params =
{
    description: "FTP Server"
}

        │
        ▼

Iteration 2

password

Credential?
YES

Value == null?
YES

Skip

↓

params =
{
    description: "FTP Server"
}

        │
        ▼

Iteration 3

hostName

params["hostName"] = "192.168.1.10"

↓

params =
{
    description: "FTP Server",
    hostName: "192.168.1.10"
}

        │
        ▼

Loop Finished

payload =
{
    description: "FTP Server",
    hostName: "192.168.1.10"
}
```

---

# Equivalent `for...of` Loop

The `reduce()` implementation is equivalent to:

```ts
const params: Record<string, any> = {};

for (const [key, field] of Object.entries(form)) {

    if (CREDENTIAL_FIELDS.has(key) && field.value === null) {
        continue;
    }

    if (field.value !== null && field.value !== undefined) {
        params[key] = field.value;
    }
}

const payload = params;
```

Sometimes the `for...of` version is easier to understand because it explicitly shows that we start with an empty object and gradually add properties to it.

---

# Key Takeaways

- `reduce()` processes every element of an array.
- The **accumulator** (`params`) carries data from one iteration to the next.
- In this code, the accumulator is an **object** that becomes the request payload.
- `Object.entries(form)` converts the form object into an array of `[key, value]` pairs.
- Credential fields with a value of `null` are intentionally skipped.
- Every valid field is added to `params`.
- After the final iteration, `params` is returned as the completed payload.
