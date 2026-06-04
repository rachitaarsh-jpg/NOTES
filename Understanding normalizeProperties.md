# Understanding `normalizeProperties`

## Code

```groovy
def normalizeProperties = { List props ->
    if (!props) return []

    props.collect { prop ->
        def name = prop?.key
        def value = prop?.value

        [key: name, value: value]
    }.findAll { it.key }
}

def properties = normalizeProperties(lineItem.customAttributes)
```

---

## Purpose

The `normalizeProperties` closure is used to:

* Handle null or empty lists safely.
* Extract only the required fields (`key` and `value`) from each property.
* Convert all properties into a consistent structure.
* Remove invalid properties that do not have a key.
* Return a clean list for further processing.

---

## Step 1: Define a Closure

```groovy
def normalizeProperties = { List props ->
```

This creates a Groovy closure (similar to a function or lambda).

It accepts one parameter:

```groovy
List props
```

which is expected to be a list of property objects/maps.

---

## Step 2: Null/Empty Check

```groovy
if (!props) return []
```

Groovy treats the following as false:

* `null`
* `[]` (empty list)
* `""` (empty string)
* `0`

So if `props` is null or empty, the function immediately returns:

```groovy
[]
```

### Example

Input:

```groovy
normalizeProperties(null)
```

Output:

```groovy
[]
```

---

## Step 3: Transform Each Property Using `collect()`

```groovy
props.collect { prop ->
```

`collect()` is similar to Java Stream's `map()`.

It loops through every item in the list and creates a new list.

### Example Input

```groovy
[
    [key:"Color", value:"Red"],
    [key:"Size", value:"XL"]
]
```

Iterations:

#### First Iteration

```groovy
prop = [key:"Color", value:"Red"]
```

#### Second Iteration

```groovy
prop = [key:"Size", value:"XL"]
```

---

## Step 4: Extract Key and Value

```groovy
def name = prop?.key
def value = prop?.value
```

### Safe Navigation Operator (`?.`)

The `?.` operator prevents `NullPointerException`.

Equivalent logic:

```groovy
name = (prop != null) ? prop.key : null
```

### Example

Input:

```groovy
prop = [key:"Color", value:"Red"]
```

Output:

```groovy
name = "Color"
value = "Red"
```

---

## Step 5: Create a Normalized Map

```groovy
[key: name, value: value]
```

Creates a new map containing only the fields we need.

Example:

```groovy
name = "Color"
value = "Red"
```

becomes:

```groovy
[
    key:"Color",
    value:"Red"
]
```

---

## Step 6: Result After `collect()`

Input:

```groovy
[
    [key:"Color", value:"Red"],
    [key:"Size", value:"XL"]
]
```

Output after `collect()`:

```groovy
[
    [key:"Color", value:"Red"],
    [key:"Size", value:"XL"]
]
```

A new list is generated containing normalized maps.

---

## Step 7: Filter Invalid Records Using `findAll()`

```groovy
.findAll { it.key }
```

`findAll()` acts like a filter.

It keeps only items where:

```groovy
it.key
```

has a value.

### Example

Before filtering:

```groovy
[
    [key:"Color", value:"Red"],
    [key:null, value:"XL"],
    [key:"Size", value:"Large"]
]
```

After filtering:

```groovy
[
    [key:"Color", value:"Red"],
    [key:"Size", value:"Large"]
]
```

Removed:

```groovy
[key:null, value:"XL"]
```

because the key is missing.

---

## Complete Example

### Input

```groovy
lineItem.customAttributes = [
    [key:"Color", value:"Blue"],
    [key:"Size", value:"M"],
    [key:null, value:"Something"]
]
```

### Execution

```groovy
def properties = normalizeProperties(lineItem.customAttributes)
```

### After `collect()`

```groovy
[
    [key:"Color", value:"Blue"],
    [key:"Size", value:"M"],
    [key:null, value:"Something"]
]
```

### After `findAll()`

```groovy
[
    [key:"Color", value:"Blue"],
    [key:"Size", value:"M"]
]
```

---

## Final Value of `properties`

```groovy
[
    [key:"Color", value:"Blue"],
    [key:"Size", value:"M"]
]
```

---

## Flow Diagram

```text
lineItem.customAttributes
            │
            ▼
    normalizeProperties()
            │
            ▼
   Check null/empty
            │
            ▼
      collect()
   (extract key/value)
            │
            ▼
       findAll()
 (remove entries without key)
            │
            ▼
     Clean Property List
```

---

## Real OMS Use Case

Incoming API payload:

```json
{
  "customAttributes": [
    {
      "key": "Color",
      "value": "Blue"
    },
    {
      "key": "Size",
      "value": "M"
    }
  ]
}
```

Before storing these values in entities such as:

* `OrderItemAttribute`
* `ProductFeatureAppl`
* Custom attribute tables

the OMS normalizes the data to ensure:

* No null entries
* Consistent structure
* Only required fields are passed forward

This reduces errors and makes downstream processing simpler.

---

## Key Groovy Concepts Used

| Concept       | Purpose                              |
| ------------- | ------------------------------------ |
| Closure `{}`  | Defines a reusable block of code     |
| `collect()`   | Similar to Java Stream `map()`       |
| `findAll()`   | Similar to Java Stream `filter()`    |
| `?.`          | Safe navigation operator             |
| `[key:value]` | Groovy map syntax                    |
| `!props`      | Checks for null or empty collections |

---

## One-Line Summary

`normalizeProperties()` takes a list of custom attributes, safely extracts only the `key` and `value` fields, removes invalid entries with missing keys, and returns a clean standardized list for further processing.

