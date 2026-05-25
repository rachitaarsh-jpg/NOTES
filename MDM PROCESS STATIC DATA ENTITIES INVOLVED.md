**MDM PROCESS STATIC DATA ENTITIES INVOLVED**

**Problem Statement: Find out Entities and fields from the MDM Processing phase which have static data like ENUMS and Types**

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

| Entity | Field | Example Value |  Found In |
| :---: | :---: | :---: | ----- |
| OrderHeader | statusId | ORDER\_CREATED | PrepareTransformedShopifyOrderPayload.groovy |
| Order Identification | orderIdentificationTypeId | SHOPIFY\_ORD\_NO | PrepareTransformedShopifyOrderPayload.groovy |
| OrderItemShipGroup | ShipmentMethodTypedId | STANDARD/POS\_COMPLETED | PrepareTransformedShopifyOrderPayload.groovy |
| OrderAdjustment | orderAdjustmentTypeId | DONATION\_ADJUSTMENT | PrepareTransformedShopifyOrderPayload.groovy |
| PartyIdentification | customerIdentification | SHOPIFY\_CUST\_ID | PrepareTransformedShopifyOrderPayload.groovy |
| ProductStoreSetting | settingTypeEnumId | SAVE\_BILL\_TO\_INF | PrepareTransformedShopifyOrderPayload.groovy |
| ProductStoreShipment (VIEW) | roleTypeId  | CARRIER | PrepareTransformedShopifyOrderPayload.groovy |
| ProductStoreShipment (VIEW) | productStoreShipMethId | NEXT\_DAY\_SHIP | PrepareTransformedShopifyOrderPayload.groovy |
| ShopifyShopTypeMapping | mappedTypeId | SHOP\_FULL\_SRVC\_ALLOC | PrepareTransformedShopifyOrderPayload.groovy |
| OrderItem | statusId | ITEM\_COMPLETED | PrepareTransformedShopifyOrderPayload.groovy |
| ContactMechPurposeType / FacilityContactDetailByPurpose | contactMechPurposeTypeId | PRIMARY\_LOCATION | PrepareTransformedShopifyOrderPayload.groovy |
| ProductStore | productIdentifierEnumId | SHOPIFY\_PRODUCT\_SKU | PrepareTransformedShopifyOrderPayload |
| OrderAjdustment | itemTaxAdjustmentType/orderAjdustmentTypeId | SALES\_TAX | PrepareTransformedShopifyOrderPayload.groovy |
| OrderItemAssocType | orderItemAssocTypeId | EXCHANGE | PrepareTransformedShopifyOrderPayload.groovy |
| OrderRole | roleTypeId | PLACING\_CUSTOMER | ShopifyOrderServices.xml |
| OrderPaymentPreference | paymentStatusId | PAYMENT\_REFUNDED | ShopifyOrderServices.xml |
| OrderPaymentPreference | paymentMethodTypeId | EXT\_SHOP\_OTHR\_GATEWAY | ShopifyOrderServices.xml |
| Party | statusId | PARTY\_ENABLED | ShopifyOrderServices.xml |
| OrderItemShipGroup | facilityId | \_NA\_ | ShopifyOrderServices.xml |
| OrderItem | orderItemTypeId | PRODUCT\_ORDER\_ITEM | ShopifyOrderServices.xml |
| CommunicationEventAndOrder | CommunicationEventTypeId | ORDER\_NOTE | ShopifyOrderServices.xml |
| SaleOrder (VIEW) | statusId | ITEM\_APPROVED | ShopifyOrderServices.xml |
| ShipmentItem (VIEW) | ShipmentStatusId | SHIPMENT\_SHIPPED | ShopifyOrderServices.xml |
| ProductAssocAndForm | ProductTypeIdProductAssocTypeId | MARKETING\_PKG\_PICKPRODUCT\_COMPONENT | ShopifyOrderServices.xml |
| SystemMessageRemote | internalTypeId | HOTWAX\_SHOPID | ShopifyOrderServices.xml |

### 

       .

