---
name: Create, release and track a shipment
description: Create a shipment in a space, release it for pickup, then get/list/update or cancel it and read its delivery state.
api: openapi/einride-book-openapi-original.yml
operations: [ShipmentService_CreateShipment, ShipmentService_GetShipment, ShipmentService_ListShipments, ShipmentService_UpdateShipment, ShipmentService_ReleaseShipment, ShipmentService_CancelShipment]
---

# Create, release and track a shipment

A Shipment is a demand to move goods from an origin to a destination.

## Steps
1. Authenticate (see einride-authenticate.md); use the parent `spaces/{space}`.
2. Create: `POST /v1beta1/{space}/shipments` (`ShipmentService_CreateShipment`) with
   required `sender`, `deliveryAddress` (recipient/line1/postalCode/city/regionCode),
   and `units[]` (each with `type` + `weight`). Optionally set `pickupAddress`,
   pickup/delivery time windows, `service` (`PALLET`/`FTL`/`DRAYAGE`), and a unique
   `externalReferenceId`.
3. Release for pickup: `POST /v1beta1/{shipment}:release`
   (`ShipmentService_ReleaseShipment`) — state becomes `RELEASED`.
4. Track: `GET /v1beta1/{shipment}` (`ShipmentService_GetShipment`) and read
   `deliveryState` (`AWAITING` / `IN_TRANSIT` / `DELIVERED`) and `trackingId`.
   List with `ShipmentService_ListShipments` (paginate via `pageSize`/`pageToken`,
   sort with `orderBy`).
5. Amend: `PATCH /v1beta1/{shipment.name}` (`ShipmentService_UpdateShipment`).
   Cancel: `POST /v1beta1/{shipment}:cancel` (`ShipmentService_CancelShipment`) —
   state becomes `CANCELLED`.

## Rules
- Resource names are canonical: `spaces/{space}/shipments/{shipment}`.
- Read-only fields (`space`, `createTime`, `trackingId`, `deliveryState`, ...) are
  server-set; do not send them on create.
- Paginate with `pageToken` until `nextPageToken` is empty; `totalSize` gives the count.
- Errors are `google.rpc.Status` — see errors/einride-problem-types.yml.
