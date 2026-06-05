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






---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------




# Understanding Shipment Method Mapping Logic

## Code

```groovy
if (shipmentMethodDesc) {
    def shopCarrierShipmentMethod = ec.entity.find("co.hotwax.shopify.ShopifyShopCarrierShipment")
            .condition("shopId", shopId)
            .useCache(true).list()
            ?.find { it.shopifyShippingMethod?.toString()?.equalsIgnoreCase(shipmentMethodDesc) }

    if (shopCarrierShipmentMethod) {
        shipmentMethodTypeId = shopCarrierShipmentMethod.shipmentMethodTypeId
        carrierPartyId = shopCarrierShipmentMethod.carrierPartyId

    } else if (productStoreId) {

        def productStoreShipmentMeth = ec.entity.find("co.hotwax.product.store.ProductStoreShipmentMethView")
                .condition("productStoreId", productStoreId)
                .condition("partyId", "_NA_")
                .condition("roleTypeId", "CARRIER")
                .condition("description", "like", "%${shipmentMethodDesc}%")
                .useCache(true).one()

        if (productStoreShipmentMeth) {
            shipmentMethodTypeId = productStoreShipmentMeth.shipmentMethodTypeId
        }
    }
}
```

---

# Purpose

This code attempts to determine the correct:

* `shipmentMethodTypeId`
* `carrierPartyId`

based on a shipping method description received from Shopify.

### Example

Shopify sends:

```text
Standard Shipping
```

or

```text
Express Delivery
```

The OMS needs to convert that text into internal shipment method records.

---

# Overall Flow

```text
Received shipmentMethodDesc
            │
            ▼
Search ShopifyShopCarrierShipment mapping
            │
     Found? ───── Yes ──► Set shipmentMethodTypeId
            │             Set carrierPartyId
            │
            No
            │
            ▼
Search ProductStoreShipmentMethView
            │
     Found? ───── Yes ──► Set shipmentMethodTypeId
            │
            No
            │
            ▼
Leave values unset
```

---

# Step 1: Check if Shipment Method Description Exists

```groovy
if (shipmentMethodDesc)
```

Only execute the logic if a shipping method was received.

Example:

```groovy
shipmentMethodDesc = "Standard Shipping"
```

If:

```groovy
shipmentMethodDesc = null
```

the entire block is skipped.

---

# Step 2: Find Shopify-Specific Mapping

```groovy
def shopCarrierShipmentMethod = ec.entity.find(
        "co.hotwax.shopify.ShopifyShopCarrierShipment")
```

Queries the entity:

```text
ShopifyShopCarrierShipment
```

This entity stores mappings between:

| Shopify Shipping Method | Internal Shipment Method |
| ----------------------- | ------------------------ |
| Standard Shipping       | GROUND                   |
| Express Shipping        | NEXT_DAY                 |
| Free Shipping           | STANDARD                 |

---

## Filter by Shop

```groovy
.condition("shopId", shopId)
```

Example:

```groovy
shopId = "SHOPIFY_US"
```

Only mappings for that shop are loaded.

---

## Use Cache

```groovy
.useCache(true)
```

Avoids repeated database queries.

If data is already cached, it is fetched from memory.

---

## Fetch All Matching Records

```groovy
.list()
```

Returns multiple rows.

Example:

```groovy
[
    [shopifyShippingMethod:"Standard Shipping"],
    [shopifyShippingMethod:"Express Shipping"]
]
```

---

## Find Matching Shipping Method

```groovy
.find {
    it.shopifyShippingMethod
      ?.toString()
      ?.equalsIgnoreCase(shipmentMethodDesc)
}
```

Looks for a record whose Shopify shipping method matches the incoming description.

Example:

Incoming:

```groovy
shipmentMethodDesc = "standard shipping"
```

Database:

```groovy
shopifyShippingMethod = "Standard Shipping"
```

`equalsIgnoreCase()` returns:

```groovy
true
```

So a match is found.

---

# Step 3: Mapping Found

```groovy
if (shopCarrierShipmentMethod)
```

Example record:

```groovy
[
    shipmentMethodTypeId : "GROUND",
    carrierPartyId       : "UPS"
]
```

---

## Set Shipment Method

```groovy
shipmentMethodTypeId =
    shopCarrierShipmentMethod.shipmentMethodTypeId
```

Result:

```groovy
shipmentMethodTypeId = "GROUND"
```

---

## Set Carrier

```groovy
carrierPartyId =
    shopCarrierShipmentMethod.carrierPartyId
```

Result:

```groovy
carrierPartyId = "UPS"
```

At this point the lookup is complete.

---

# Step 4: Fallback Logic

If Shopify mapping is not found:

```groovy
else if (productStoreId)
```

Try another lookup.

Example:

```groovy
productStoreId = "STORE100"
```

---

# Query ProductStoreShipmentMethView

```groovy
ec.entity.find(
    "co.hotwax.product.store.ProductStoreShipmentMethView")
```

This view contains shipment methods configured for a product store.

Example:

| Product Store | Shipment Method |
| ------------- | --------------- |
| STORE100      | Ground          |
| STORE100      | Express         |
| STORE100      | Pickup          |

---

## Filter by Product Store

```groovy
.condition("productStoreId", productStoreId)
```

Example:

```groovy
STORE100
```

---

## Carrier Placeholder

```groovy
.condition("partyId", "_NA_")
```

Looks for shipment methods not tied to a specific carrier.

`_NA_` typically means:

```text
Not Applicable
```

or

```text
Generic shipment method
```

---

## Carrier Role

```groovy
.condition("roleTypeId", "CARRIER")
```

Restricts results to carrier-related shipment methods.

---

## Description Matching

```groovy
.condition(
    "description",
    "like",
    "%${shipmentMethodDesc}%"
)
```

SQL equivalent:

```sql
WHERE description LIKE '%Standard Shipping%'
```

Example:

Database:

```text
Standard Shipping (3-5 Days)
```

Incoming:

```text
Standard Shipping
```

Match found because description contains the text.

---

## Fetch One Record

```groovy
.one()
```

Returns the first matching record.

Example:

```groovy
[
    shipmentMethodTypeId:"GROUND"
]
```

---

# Step 5: Set Shipment Method Type

```groovy
if (productStoreShipmentMeth)
```

If a matching record exists:

```groovy
shipmentMethodTypeId =
    productStoreShipmentMeth.shipmentMethodTypeId
```

Result:

```groovy
shipmentMethodTypeId = "GROUND"
```

Notice:

```groovy
carrierPartyId
```

is NOT assigned in this fallback path.

Only the shipment method type is determined.

---

# Real OMS Example

Shopify Order:

```json
{
  "shipping_method": "Express Shipping"
}
```

### First Attempt

Search:

```text
ShopifyShopCarrierShipment
```

Found:

```text
Express Shipping -> NEXT_DAY
Carrier -> FEDEX
```

Result:

```groovy
shipmentMethodTypeId = "NEXT_DAY"
carrierPartyId = "FEDEX"
```

---

### If No Shopify Mapping Exists

Search:

```text
ProductStoreShipmentMethView
```

Found:

```text
Description:
Express Shipping (1 Day)

shipmentMethodTypeId:
NEXT_DAY
```

Result:

```groovy
shipmentMethodTypeId = "NEXT_DAY"
```

Carrier remains unset.

---

# Why Two Lookups?

The code follows a priority order:

### Priority 1

Use explicit Shopify mappings.

```text
Shopify Shipping Method
        ↓
Internal Shipment Method
```

Most accurate.

---

### Priority 2

Use Product Store shipment configuration.

```text
Description Match
        ↓
Shipment Method Type
```

Fallback option when no Shopify-specific mapping exists.

---

# One-Line Summary

This code converts a Shopify shipping method description into the OMS shipment method type and carrier by first checking explicit Shopify-to-OMS mappings (`ShopifyShopCarrierShipment`) and, if no mapping exists, falling back to shipment methods configured for the product store (`ProductStoreShipmentMethView`).





---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------






# Understanding `getTypeMapping()` in Shopify OMS Integration

## Code

```groovy
def getTypeMapping = { String mappedTypeId, String mappedKey, String defaultValue = null ->
    if (!mappedKey) return defaultValue

    def mappings = ec.entity.find("co.hotwax.shopify.ShopifyShopTypeMapping")
            .condition("shopId", shopId)
            .condition("mappedTypeId", mappedTypeId)
            .useCache(true)
            .list()

    def match = mappings?.find {
        it.mappedKey?.toString()?.equalsIgnoreCase(mappedKey.toString())
    }

    return match?.mappedValue ?: defaultValue
}

def channelSource = order.sourceName

def channelId = getTypeMapping(
        "SHOPIFY_ORDER_SOURCE",
        channelSource?.toString(),
        "UNKNWN_SALES_CHANNEL"
)
```

---

# Purpose

This function converts an external Shopify value into an internal OMS/OFBiz value using a database mapping table.

Instead of hardcoding values, mappings are stored in the database and looked up dynamically.

---

# Example Mapping Table

Entity:

```text
ShopifyShopTypeMapping
```

Sample records:

| shopId  | mappedTypeId         | mappedKey | mappedValue |
| ------- | -------------------- | --------- | ----------- |
| GORJANA | SHOPIFY_ORDER_SOURCE | web       | WEB_STORE   |
| GORJANA | SHOPIFY_ORDER_SOURCE | pos       | POS_STORE   |
| GORJANA | SHOPIFY_ORDER_SOURCE | mobile    | MOBILE_APP  |

---

# Step 1: Read Shopify Source

```groovy
def channelSource = order.sourceName
```

Shopify Order:

```json
{
  "sourceName": "web"
}
```

So:

```groovy
channelSource = "web"
```

---

# Step 2: Function Call

```groovy
def channelId = getTypeMapping(
        "SHOPIFY_ORDER_SOURCE",
        "web",
        "UNKNWN_SALES_CHANNEL"
)
```

Parameter values:

| Parameter    | Value                |
| ------------ | -------------------- |
| mappedTypeId | SHOPIFY_ORDER_SOURCE |
| mappedKey    | web                  |
| defaultValue | UNKNWN_SALES_CHANNEL |

---

# Step 3: Null Check

```groovy
if (!mappedKey) return defaultValue
```

Equivalent to:

```groovy
if (mappedKey == null || mappedKey == "")
    return defaultValue
```

If Shopify does not send a source name:

```groovy
mappedKey = null
```

Function immediately returns:

```groovy
UNKNWN_SALES_CHANNEL
```

---

# Step 4: Query Mapping Records

```groovy
def mappings = ec.entity.find("co.hotwax.shopify.ShopifyShopTypeMapping")
        .condition("shopId", shopId)
        .condition("mappedTypeId", mappedTypeId)
        .useCache(true)
        .list()
```

This fetches all mappings for:

```groovy
shopId
```

and

```groovy
mappedTypeId = "SHOPIFY_ORDER_SOURCE"
```

Example result:

```groovy
mappings = [
    [mappedKey:"web", mappedValue:"WEB_STORE"],
    [mappedKey:"pos", mappedValue:"POS_STORE"],
    [mappedKey:"mobile", mappedValue:"MOBILE_APP"]
]
```

---

# Step 5: Find Matching Mapping

```groovy
def match = mappings?.find {
    it.mappedKey?.toString()?.equalsIgnoreCase(mappedKey.toString())
}
```

The `find()` method searches the list and returns the first matching record.

### Iteration 1

```groovy
it = [mappedKey:"web", mappedValue:"WEB_STORE"]
```

Comparison:

```groovy
"web".equalsIgnoreCase("web")
```

Result:

```groovy
true
```

Therefore:

```groovy
match = [mappedKey:"web", mappedValue:"WEB_STORE"]
```

The search stops immediately.

---

# Why Use `find()`?

### `find()`

```groovy
list.find { condition }
```

Returns:

* First matching element
* Stops searching once found

### `each()`

```groovy
list.each { }
```

Processes every element.

Since only one mapping is needed, `find()` is more efficient.

---

# Step 6: Return Value

```groovy
return match?.mappedValue ?: defaultValue
```

Uses the Elvis operator (`?:`).

Equivalent to:

```groovy
if (match?.mappedValue)
    return match.mappedValue
else
    return defaultValue
```

Since:

```groovy
match.mappedValue = "WEB_STORE"
```

Function returns:

```groovy
WEB_STORE
```

---

# Final Result

```groovy
channelId = "WEB_STORE"
```

---

# What Happens if No Mapping Exists?

Suppose Shopify sends:

```json
{
  "sourceName": "instagram"
}
```

No matching record exists in the mapping table.

Then:

```groovy
match = null
```

So:

```groovy
return match?.mappedValue ?: defaultValue
```

returns:

```groovy
UNKNWN_SALES_CHANNEL
```

instead of throwing an error.

---

# Null Safety Used in the Function

## Safe Navigation Operator (`?.`)

```groovy
mappings?.find { ... }
```

Prevents:

```groovy
NullPointerException
```

if mappings is null.

---

```groovy
it.mappedKey?.toString()
```

Prevents:

```groovy
NullPointerException
```

if mappedKey is null.

---

```groovy
match?.mappedValue
```

Returns:

```groovy
null
```

instead of throwing an exception when no matching mapping is found.

---

# Elvis Operator (`?:`)

```groovy
match?.mappedValue ?: defaultValue
```

Meaning:

> Return the mapped value if it exists, otherwise return the default value.

Example:

```groovy
null ?: "DEFAULT"
```

Result:

```groovy
"DEFAULT"
```

---

# Flow Diagram

```text
Shopify Order
     |
     v
sourceName = "web"
     |
     v
getTypeMapping(
    "SHOPIFY_ORDER_SOURCE",
    "web",
    "UNKNWN_SALES_CHANNEL"
)
     |
     v
Query ShopifyShopTypeMapping
     |
     v
Find record where mappedKey = "web"
     |
     v
mappedValue = "WEB_STORE"
     |
     v
channelId = "WEB_STORE"
```

---

# Key Concepts Learned

* Closures in Groovy
* Database lookup using Entity Engine
* Dynamic mapping instead of hardcoding values
* `find()` vs `each()`
* Null-safe navigation operator (`?.`)
* Elvis operator (`?:`)
* Default/fallback values
* Converting external Shopify values into internal OMS values




---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------




# Understanding `getTypeMapping()` in Shopify OMS Integration

## Code

```groovy
def getTypeMapping = { String mappedTypeId, String mappedKey, String defaultValue = null ->
    if (!mappedKey) return defaultValue

    def mappings = ec.entity.find("co.hotwax.shopify.ShopifyShopTypeMapping")
            .condition("shopId", shopId)
            .condition("mappedTypeId", mappedTypeId)
            .useCache(true)
            .list()

    def match = mappings?.find {
        it.mappedKey?.toString()?.equalsIgnoreCase(mappedKey.toString())
    }

    return match?.mappedValue ?: defaultValue
}

def channelSource = order.sourceName

def channelId = getTypeMapping(
        "SHOPIFY_ORDER_SOURCE",
        channelSource?.toString(),
        "UNKNWN_SALES_CHANNEL"
)
```

---

# Purpose

This function converts an external Shopify value into an internal OMS/OFBiz value using a database mapping table.

Instead of hardcoding values, mappings are stored in the database and looked up dynamically.

---

# Example Mapping Table

Entity:

```text
ShopifyShopTypeMapping
```

Sample records:

| shopId  | mappedTypeId         | mappedKey | mappedValue |
| ------- | -------------------- | --------- | ----------- |
| GORJANA | SHOPIFY_ORDER_SOURCE | web       | WEB_STORE   |
| GORJANA | SHOPIFY_ORDER_SOURCE | pos       | POS_STORE   |
| GORJANA | SHOPIFY_ORDER_SOURCE | mobile    | MOBILE_APP  |

---

# Step 1: Read Shopify Source

```groovy
def channelSource = order.sourceName
```

Shopify Order:

```json
{
  "sourceName": "web"
}
```

So:

```groovy
channelSource = "web"
```

---

# Step 2: Function Call

```groovy
def channelId = getTypeMapping(
        "SHOPIFY_ORDER_SOURCE",
        "web",
        "UNKNWN_SALES_CHANNEL"
)
```

Parameter values:

| Parameter    | Value                |
| ------------ | -------------------- |
| mappedTypeId | SHOPIFY_ORDER_SOURCE |
| mappedKey    | web                  |
| defaultValue | UNKNWN_SALES_CHANNEL |

---

# Step 3: Null Check

```groovy
if (!mappedKey) return defaultValue
```

Equivalent to:

```groovy
if (mappedKey == null || mappedKey == "")
    return defaultValue
```

If Shopify does not send a source name:

```groovy
mappedKey = null
```

Function immediately returns:

```groovy
UNKNWN_SALES_CHANNEL
```

---

# Step 4: Query Mapping Records

```groovy
def mappings = ec.entity.find("co.hotwax.shopify.ShopifyShopTypeMapping")
        .condition("shopId", shopId)
        .condition("mappedTypeId", mappedTypeId)
        .useCache(true)
        .list()
```

This fetches all mappings for:

```groovy
shopId
```

and

```groovy
mappedTypeId = "SHOPIFY_ORDER_SOURCE"
```

Example result:

```groovy
mappings = [
    [mappedKey:"web", mappedValue:"WEB_STORE"],
    [mappedKey:"pos", mappedValue:"POS_STORE"],
    [mappedKey:"mobile", mappedValue:"MOBILE_APP"]
]
```

---

# Step 5: Find Matching Mapping

```groovy
def match = mappings?.find {
    it.mappedKey?.toString()?.equalsIgnoreCase(mappedKey.toString())
}
```

The `find()` method searches the list and returns the first matching record.

### Iteration 1

```groovy
it = [mappedKey:"web", mappedValue:"WEB_STORE"]
```

Comparison:

```groovy
"web".equalsIgnoreCase("web")
```

Result:

```groovy
true
```

Therefore:

```groovy
match = [mappedKey:"web", mappedValue:"WEB_STORE"]
```

The search stops immediately.

---

# Why Use `find()`?

### `find()`

```groovy
list.find { condition }
```

Returns:

* First matching element
* Stops searching once found

### `each()`

```groovy
list.each { }
```

Processes every element.

Since only one mapping is needed, `find()` is more efficient.

---

# Step 6: Return Value

```groovy
return match?.mappedValue ?: defaultValue
```

Uses the Elvis operator (`?:`).

Equivalent to:

```groovy
if (match?.mappedValue)
    return match.mappedValue
else
    return defaultValue
```

Since:

```groovy
match.mappedValue = "WEB_STORE"
```

Function returns:

```groovy
WEB_STORE
```

---

# Final Result

```groovy
channelId = "WEB_STORE"
```

---

# What Happens if No Mapping Exists?

Suppose Shopify sends:

```json
{
  "sourceName": "instagram"
}
```

No matching record exists in the mapping table.

Then:

```groovy
match = null
```

So:

```groovy
return match?.mappedValue ?: defaultValue
```

returns:

```groovy
UNKNWN_SALES_CHANNEL
```

instead of throwing an error.

---

# Null Safety Used in the Function

## Safe Navigation Operator (`?.`)

```groovy
mappings?.find { ... }
```

Prevents:

```groovy
NullPointerException
```

if mappings is null.

---

```groovy
it.mappedKey?.toString()
```

Prevents:

```groovy
NullPointerException
```

if mappedKey is null.

---

```groovy
match?.mappedValue
```

Returns:

```groovy
null
```

instead of throwing an exception when no matching mapping is found.

---

# Elvis Operator (`?:`)

```groovy
match?.mappedValue ?: defaultValue
```

Meaning:

> Return the mapped value if it exists, otherwise return the default value.

Example:

```groovy
null ?: "DEFAULT"
```

Result:

```groovy
"DEFAULT"
```

---

# Flow Diagram

```text
Shopify Order
     |
     v
sourceName = "web"
     |
     v
getTypeMapping(
    "SHOPIFY_ORDER_SOURCE",
    "web",
    "UNKNWN_SALES_CHANNEL"
)
     |
     v
Query ShopifyShopTypeMapping
     |
     v
Find record where mappedKey = "web"
     |
     v
mappedValue = "WEB_STORE"
     |
     v
channelId = "WEB_STORE"
```

---

# Key Concepts Learned

* Closures in Groovy
* Database lookup using Entity Engine
* Dynamic mapping instead of hardcoding values
* `find()` vs `each()`
* Null-safe navigation operator (`?.`)
* Elvis operator (`?:`)
* Default/fallback values
* Converting external Shopify values into internal OMS values







