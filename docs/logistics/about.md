# Logistics <!-- {docsify-ignore} -->

A micro service architecture relies on services to do stuff.
But most of the time there is an orchestration layer.
You can use RBS Process Manager to orchestrate other micro services.

## API

For more information about Process Manager API please visit documentation section on developer console.

### Working With Options

**rbs.logistics.request.GET_OPTION :** returns current options

**Example Request :** { "optionId": "SHIPMENT" }

**Example Request :** { "optionId": "SHIPMENT_TYPES" }
***

**rbs.logistics.request.UPSERT_OPTION :** updates existing options or inserts a new one

**Example Request :** { "optionId": "SHIPMENT", "data": { "days": 3, "linearStatuses": true } }

**Example Request :** { "optionId": "SHIPMENT_TYPES", "data": [ "CARGO", "RESERVED", "ON_DEMAND", "SELF" ] }
***

### Working With Zones

**rbs.logistics.request.UPSERT_ZONE :** inserts a new zone or updates an existing zone

**Example Response :** { "zoneId": "ZONE_ID", "created": true }
***

**rbs.logistics.request.LIST_ZONES :** returns list of zones

**Example Response :** [{"zoneId":"01ERVD8Z3A827Z1MGMSEQH0R6Z","name":"Test","timezone":3}]
***

**rbs.logistics.request.GET_ZONE :** returns a zone

zoneId: ZONE_ID

**Example Response :** {"zoneId":"01ERVD8Z3A827Z1MGMSEQH0R6Z","name":"Test","timezone":3}
***

**rbs.logistics.request.DELETE_ZONE :** deletes a zone

zoneId: ZONE_ID

**Example Response :** {"zoneId":"01ERVD8Z3A827Z1MGMSEQH0R6Z","name":"Test","timezone":3, "deleted": true }
***

**rbs.logistics.request.LOCATE_ZONE :** returns a zoneId by address or location

{ address: ADDRESS, location: LOCATION }

**Example Request :** { "address": { "city": "İstanbul", "district": "Ataşehir" } }

**Example Request :** { "location": { "lat": 41, "lng": 29 } }

**Example Response :** {"zoneId":"01ERVD8Z3A827Z1MGMSEQH0R6Z"}
***

### Working With Stores

**rbs.logistics.request.UPSERT_STORE :** inserts a new store or updates an existing store

**Example Response :** { "merchantId": "MERCHANT_ID", "storeId": "STORE_ID", "created": true }
***

**rbs.logistics.request.LIST_STORES :** returns list of stores

merchantId: MERCHANT_ID

**Example Response :** [{"merchantId": "MERCHANT_ID","storeId":"01ERVD8Z3A827Z1MGMSEQH0R6Z","name":"Test"}]
***

**rbs.logistics.request.GET_STORE :** returns a store

merchantId: MERCHANT_ID

storeId: STORE_ID

**Example Response :** {"merchantId": "MERCHANT_ID", "storeId":"01ERVD8Z3A827Z1MGMSEQH0R6Z", "name":"Test"}
***

**rbs.logistics.request.DELETE_STORE :** deletes a store

merchantId: MERCHANT_ID

storeId: STORE_ID

**Example Response :** {"merchantId": "MERCHANT_ID","storeId":"01ERVD8Z...H0R6Z", "name":"Test", "deleted": true}
***
