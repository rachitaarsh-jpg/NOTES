# Shopify Order JSON → OMS Entity Mapping Notes

## Overview

This document explains how each field from the Shopify `orderDetails` JSON maps to the corresponding OMS entity and attribute.

---

# 1. OrderHeader

The **OrderHeader** represents the overall order.

### Important Field Mapping

| Shopify Field | OMS Field |
|---------------|-----------|
| id | externalId |
| legacyResourceId | externalId (numeric) |
| name | orderName |
| number | Display order reference |
| createdAt | orderDate |
| updatedAt | lastUpdatedStamp |
| closedAt | completedDate |
| cancelledAt | cancelledDate |
| currencyCode | currencyUom |
| presentmentCurrencyCode | presentmentCurrencyUom |
| displayFulfillmentStatus | statusId |
| sourceName | salesChannelEnumId |
| customerLocale | locale |
| totalPrice | grandTotal |
| currentTotalPrice | remainingSubTotal |

### Notes

- One OrderHeader exists per Shopify order.
- Contains order lifecycle information.
- Holds pricing, currency, channel and status.
- Tags and custom attributes become `OrderAttribute` records.

---

# 2. OrderItem

Each Shopify Line Item becomes one OrderItem.

### Important Mapping

| Shopify Field | OMS Field |
|---------------|-----------|
| id | externalId |
| title | itemDescription |
| name | Full variant description |
| sku | Used to resolve Product |
| quantity | quantity |
| currentQuantity | quantity |
| discountedUnitPrice | unitPrice |
| originalUnitPrice | unitListPrice |
| requiresShipping | shipGroupSeqId |
| taxable | Used for tax calculation |

### Notes

- One OrderItem per purchased product.
- SKU is **not stored directly**.
- SKU is used to find Product through GoodIdentification.

---

# 3. Tax Mapping (OrderAdjustment)

Every tax line becomes an OrderAdjustment.

| Shopify | OMS |
|----------|-----|
| taxLines.title | description |
| taxLines.rate | sourcePercentage |
| taxLines.amount | amount |

Adjustment Type:

```
SALES_TAX
```

---

# 4. Product Identification (GoodIdentification)

Shopify identifies products using Variant IDs and SKU.

| Shopify Field | OMS Entity |
|---------------|------------|
| Variant ID | GoodIdentification |
| Legacy Variant ID | GoodIdentification |
| SKU | GoodIdentification |
| Barcode | GoodIdentification |
| Variant Title | ProductFeature |

### Notes

GoodIdentification is the bridge between Shopify identifiers and OMS Product IDs.

---

# 5. Shipping & Billing Addresses

Addresses are stored using ContactMech.

## Shipping

Stored as:

- PostalAddress
- OrderContactMech
- SHIPPING_LOCATION

Important fields:

- Name
- Address1
- Address2
- City
- State
- Postal Code
- Country
- Latitude
- Longitude

---

## Billing

Same mapping as Shipping.

Purpose Type:

```
BILLING_LOCATION
```

---

## Email

Email becomes:

```
ContactMech
```

Purpose Type:

```
ORDER_EMAIL
```

---

# 6. Customer Mapping (Party)

Customer information becomes Party records.

| Shopify | OMS |
|----------|-----|
| Customer ID | PartyIdentification |
| Legacy Customer ID | PartyIdentification |
| First Name | Person.firstName |
| Last Name | Person.lastName |
| Email | ContactMech |

Role:

```
PLACING_CUSTOMER
```

---

# 7. Shipping Method (OrderItemShipGroup)

Shipping information becomes an OrderItemShipGroup.

| Shopify | OMS |
|----------|-----|
| Shipping Title | shipmentMethodTypeId |
| Shipping Code | carrierPartyId |
| Shipping Cost | OrderAdjustment |

Example:

```
STOREPICKUP
```

---

# 8. Payment Mapping

Transactions become OrderPaymentPreference.

| Shopify | OMS |
|----------|-----|
| Transaction ID | externalId |
| Gateway | paymentMethodTypeId |
| Kind | statusId |
| Status | statusId |
| Amount | maxAmount |
| ProcessedAt | presentedDate |

Gateway response stored in:

```
PaymentGatewayResponse
```

---

# 9. Discount Mapping

Discounts become OrderAdjustment records.

Sources:

- Discount Codes
- Discount Applications
- Line Item Discount Allocations

Adjustment Type:

```
PROMOTION_ADJUSTMENT
```

---

# 10. OrderAttribute

Stores Shopify metadata that doesn't belong in core OMS entities.

Examples:

- Tags
- Custom Attributes
- Status Page URL

Stored as:

```
OrderAttribute
```

Line-item metadata becomes:

```
OrderItemAttribute
```

---

# 11. Future Fulfillment Mapping

Future Shopify objects map as follows:

| Shopify | OMS |
|----------|-----|
| Fulfillments | Shipment |
| Refunds | ReturnHeader / ReturnItem |
| Returns | ReturnHeader |
| Agreements | Agreement |

---

# Complete Entity Relationship

```text
OrderHeader
│
├── OrderRole
│      └── Party
│
├── OrderContactMech
│      ├── Shipping Address
│      ├── Billing Address
│      └── Email
│
├── OrderItem
│      │
│      ├── GoodIdentification
│      ├── Product
│      └── OrderAdjustment (Tax)
│
├── OrderItemShipGroup
│
├── OrderPaymentPreference
│      └── PaymentGatewayResponse
│
└── OrderAttribute
```

---

# Overall Mapping Flow

```text
Shopify Order JSON
        │
        ▼
   OrderHeader
        │
        ├──────────────► OrderRole
        │                    │
        │                    ▼
        │                  Party
        │
        ├──────────────► OrderContactMech
        │                    │
        │                    ├── Shipping Address
        │                    ├── Billing Address
        │                    └── Email
        │
        ├──────────────► OrderItem
        │                    │
        │                    ├── GoodIdentification
        │                    ├── Product
        │                    └── OrderAdjustment (Tax)
        │
        ├──────────────► OrderItemShipGroup
        │
        ├──────────────► OrderPaymentPreference
        │                    │
        │                    └── PaymentGatewayResponse
        │
        └──────────────► OrderAttribute
```

---

# Quick Revision

## Main OMS Entities

- OrderHeader
- OrderItem
- OrderAdjustment
- GoodIdentification
- ProductFeature
- OrderContactMech
- PostalAddress
- ContactMech
- Party
- Person
- OrderRole
- OrderItemShipGroup
- OrderPaymentPreference
- PaymentGatewayResponse
- OrderAttribute
- Shipment
- ReturnHeader
- ReturnItem
- Agreement

---

## Easy Way to Remember

**OrderHeader**
> Overall order information

**OrderItem**
> Purchased products

**GoodIdentification**
> Connects Shopify IDs to Products

**OrderAdjustment**
> Taxes & Discounts

**Party**
> Customer

**OrderContactMech**
> Shipping, Billing & Email

**OrderItemShipGroup**
> Shipping method

**OrderPaymentPreference**
> Payment information

**OrderAttribute**
> Extra Shopify metadata
