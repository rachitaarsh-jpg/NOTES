**MDM PROCESS STATIC DATA ENTITIES INVOLVED**

***PROBLEM STATEMENT:-*** **Find and list the entities and their fields where we create the type or enum data during the order import process in OMS. The purpose of the field, the list of the types/enums we have for these fields, and scenarios when each type/enum is used.**

***How did I Trace The Code ?***

### **Steps Followed for Tracing the Flow**

1. In `SQSOrderImport.xml`, towards the end of the process flow, a service named `upload#DataManagerFile` is invoked from `UtilityServices`.  
2. Navigate to `UtilityServices` and search for the `upload#DataManagerFile` service definition.  
3. Continue tracing the service implementation. Inside the service, locate the `<actions>` section, which contains the core execution logic.  
4. Within the action block, observe that a configuration value (`config`) is being fetched from the `DataManagerConfig` entity.  
5. To verify the configuration details, search for the `DataManagerConfig` entity in the local application through:  
   `localhost:8080 → Entity List`  
6. Pass the same `configId` that was used in `SQSOrderImport.xml`.  
7. After searching, the corresponding configuration record returned is:  
   `SYNC_SHOPIFY_ORDER`  
8. This configuration maps to the service:  
   `ShopifyOrderServices.sync#ShopifyOrder`  
9. Therefore, this is the service ultimately responsible for handling Shopify order synchronization and further processing.

    SQSOrderImport.xml  
        │  
        ▼  
    Calls Service:  
upload\#DataManagerFile (UtilityServices)  
        │  
        ▼  
Go to UtilityServices  
        │  
        ▼  
Trace service implementation  
        │  
        ▼  
Check \<actions\> logic  
        │  
        ▼  
Fetch config from:  
DataManagerConfig Entity  
        │  
        ▼  
Open localhost:8080 → Entity List  
        │  
        ▼  
Pass configId from SQSOrderImport.xml  
        │  
        ▼  
Retrieved Config:  
SYNC\_SHOPIFY\_ORDER  
        │  
        ▼  
Mapped Service:  
ShopifyOrderServices.sync\#ShopifyOrder

| Entity | Field | Static Values |  Found In |
| ----- | ----- | ----- | ----- |
| OrderHeader | statusId | ORDER\_CREATEDORDER\_COMPLETED (in case of POS) | PrepareTransformedShopifyOrderPayload.groovy |
| Order Identification | orderIdentificationTypeId | SHOPIFY\_ORD\_NOSHOPIFY\_ORD\_NAMESHOPIFY\_ORD\_ID | PrepareTransformedShopifyOrderPayload.groovy |
|  OrderItemShipGroup |  ShipmentMethodTypedId | STANDARD/POS\_COMPLETED STOREPICKUP SHIP\_TO\_STORENEXT\_DAYSECOND\_DAYTHIRD\_DAYNO\_SHIPPING | PrepareTransformedShopifyOrderPayload.groovy |
|  OrderAdjustment |  orderAdjustmentTypeId | DONATION\_ADJUSTMENTEXT\_PROMO\_ADJUSTMENT(external item level Promotions)EXT\_SHIP\_ADJUSTMENT(external shipping discounts)SHIPPING\_CHARGES(standard shipping charges)SALES\_TAX SHIPPING\_SALES\_TAX | PrepareTransformedShopifyOrderPayload.groovy |
|  PartyIdentification |  customerIdentification |  SHOPIFY\_CUST\_ID | PrepareTransformedShopifyOrderPayload.groovy |
|                  OrderItem |                 statusId |            ITEM\_COMPLETED ITEM\_CREATED | PrepareTransformedShopifyOrderPayload.groovy |
|  ContactMechPurposeType / FacilityContactDetailByPurpose |  contactMechPurposeTypeId | PRIMARY\_LOCATION PRIMARY\_PHONE PRIMARY\_EMAIL  |  PrepareTransformedShopifyOrderPayload.groovy |
|  ProductStore |  productIdentifierEnumId | SHOPIFY\_PRODUCT\_SKU SHOPIFY\_BARCODE SHOPIFY\_PRODUCT\_ID |  PrepareTransformedShopifyOrderPayload |
|  OrderItemAssocType |  orderItemAssocTypeId |  EXCHANGE | PrepareTransformedShopifyOrderPayload.groovy |
| OrderRole | roleTypeId | PLACING\_CUSTOMER | ShopifyOrderServices.xml |
|  OrderPaymentPreference |  paymentStatusId | PAYMENT\_REFUNDED PAYMENT\_AUTHORISEDPAYMENT\_SETTLEDPAYMENT\_CANCELLED |  ShopifyOrderServices.xml |
|  OrderPaymentPreference |  paymentMethodTypeId | EXT\_SHOP\_OTHR\_GATEWAY EXT\_SHOP\_VISA EXT\_SHOP\_AMEX EXT\_SHOP\_MASTERCARD EXT\_SHOP\_DISCOVER EXT\_SHOP\_PAYPAL EXT\_SHOP\_AFTRPAY EXT\_SHOP\_KLARNAEXT\_SHOP\_PAY\_INSTALL EXT\_GIFT\_CARD SHOP\_STORE\_CREDIT EXT\_SHOP\_CASH\_ON\_DEL |  ShopifyOrderServices.xml |
| Party | statusId | PARTY\_ENABLED | ShopifyOrderServices.xml |
| CommunicationEventAndOrder | CommunicationEventTypeId | ORDER\_NOTE | ShopifyOrderServices.xml |

### 

       .

