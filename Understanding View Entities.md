# Understanding View Entities and ProductStoreShipmentMethView

## What is a View Entity?

A **View Entity** in Moqui/OFBiz is similar to a **SQL View**.

Unlike normal entities, a view entity:

* Does **not store data physically** in the database.
* Combines data from multiple entities using joins.
* Appears like a normal entity to developers.
* Allows querying multiple related tables as a single entity.

Think of it as:

```sql
SELECT *
FROM ProductStoreShipmentMeth PSSM
JOIN ShipmentMethodType SMT
ON PSSM.shipment_method_type_id = SMT.shipment_method_type_id
```

but defined once in XML and reused throughout the application.

---

# Why Do We Need View Entities?

## 1. Simplify Complex Queries

Without a view entity:

```groovy
def shipmentMeth = ec.entity.find("ProductStoreShipmentMeth")
        .condition(...)
        .list()

// Then another query
def shipmentType = ec.entity.find("ShipmentMethodType")
        .condition(...)
        .one()
```

Multiple queries are needed.

With a view entity:

```groovy
ec.entity.find("ProductStoreShipmentMethView")
        .condition(...)
        .one()
```

Everything is retrieved in a single query.

---

## 2. Better Performance

Instead of:

```text
Query 1 → Get shipment methods
Query 2 → Get shipment method description
Query 3 → Get carrier information
```

the database performs a single JOIN query.

Benefits:

* Fewer database round trips
* Faster execution
* Avoids N+1 query problems

---

## 3. Data Aggregation

View entities can:

* Join data from multiple entities
* Calculate counts
* Calculate sums
* Provide flattened records for UI screens and business logic

Example:

```text
Order
 + Customer
 + Shipment
 + Payment
```

can be exposed as a single view entity.

---

# ProductStoreShipmentMethView

## XML Definition

```xml
<view-entity entity-name="ProductStoreShipmentMethView">

    <member-entity entity-alias="PSSM"
                   entity-name="ProductStoreShipmentMeth"/>

    <member-entity entity-alias="SM"
                   entity-name="ShipmentMethodType"
                   join-from-alias="PSSM">

        <key-map field-name="shipmentMethodTypeId"/>
    </member-entity>

    <alias-all entity-alias="PSSM"/>

    <alias-all entity-alias="SM">
        <exclude field="shipmentMethodTypeId"/>
        <exclude field="sequenceNum"/>
    </alias-all>

</view-entity>
```

---

# Entities Involved

## 1. ProductStoreShipmentMeth (PSSM)

Stores store-specific shipment configurations.

Example:

| Product Store | Shipment Method | Carrier |
| ------------- | --------------- | ------- |
| STORE100      | GROUND          | UPS     |
| STORE100      | NEXT_DAY        | FEDEX   |

This entity answers:

> Which shipment methods are available for this store?

---

## 2. ShipmentMethodType (SM)

Stores master definitions of shipment methods.

Example:

| Shipment Method Type ID | Description     |
| ----------------------- | --------------- |
| GROUND                  | Ground Shipping |
| NEXT_DAY                | Next Day Air    |
| PICKUP                  | Store Pickup    |

This entity answers:

> What does this shipment method mean?

---

# Join Logic

The join happens using:

```xml
<key-map field-name="shipmentMethodTypeId"/>
```

Equivalent SQL:

```sql
SELECT *
FROM ProductStoreShipmentMeth PSSM
JOIN ShipmentMethodType SMT
ON PSSM.shipment_method_type_id =
   SMT.shipment_method_type_id
```

---

# Result of the View

Instead of getting:

```text
ProductStoreShipmentMeth
------------------------
STORE100
NEXT_DAY
```

you also get:

```text
ShipmentMethodType
------------------------
NEXT_DAY
Next Day Air
```

Combined result:

| Product Store | Shipment Method Type | Description  |
| ------------- | -------------------- | ------------ |
| STORE100      | NEXT_DAY             | Next Day Air |

---

# Why It Is Used in Shipment Mapping Code

In the shipment mapping logic:

```groovy
def productStoreShipmentMeth =
    ec.entity.find("ProductStoreShipmentMethView")
```

the code searches by:

```groovy
.condition(
    "description",
    "like",
    "%${shipmentMethodDesc}%"
)
```

Example:

Shopify sends:

```text
Standard Shipping
```

The view allows the system to search against:

```text
ShipmentMethodType.description
```

while simultaneously having access to:

```text
ProductStoreShipmentMeth.shipmentMethodTypeId
```

because both entities are already joined together.

Without the view:

1. Query ShipmentMethodType using description.
2. Extract shipmentMethodTypeId.
3. Query ProductStoreShipmentMeth using that ID.

With the view:

```groovy
ProductStoreShipmentMethView
```

does everything in a single query.

---

# Real Example

### ProductStoreShipmentMeth

| productStoreId | shipmentMethodTypeId |
| -------------- | -------------------- |
| STORE100       | NEXT_DAY             |

### ShipmentMethodType

| shipmentMethodTypeId | description  |
| -------------------- | ------------ |
| NEXT_DAY             | Next Day Air |

### View Result

| productStoreId | shipmentMethodTypeId | description  |
| -------------- | -------------------- | ------------ |
| STORE100       | NEXT_DAY             | Next Day Air |

Now the application can search using:

```groovy
description LIKE "%Next Day%"
```

and immediately retrieve:

```groovy
shipmentMethodTypeId = "NEXT_DAY"
```

without any additional query.

---

# Key Takeaway

`ProductStoreShipmentMethView` is a view entity that joins `ProductStoreShipmentMeth` and `ShipmentMethodType`, allowing the system to retrieve store-specific shipment method configurations along with human-readable shipment method descriptions in a single efficient query. It is commonly used when mapping external shipping methods (e.g., Shopify shipping methods) to internal OMS shipment method types.

