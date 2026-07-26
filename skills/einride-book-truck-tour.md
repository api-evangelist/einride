---
name: Book and confirm a truck tour
description: Create a provisional truck tour in a space, confirm it so Saga creates the shipments, then look it up or cancel it.
api: openapi/einride-book-openapi-original.yml
operations: [BookingService_CreateTour, BookingService_ConfirmTour, BookingService_GetTour, BookingService_ListTours, BookingService_SearchTours, BookingService_CancelTour, BookingService_UpdateTour]
---

# Book and confirm a truck tour

A Tour is a preplanned set of stops (PICKUP/DELIVER) and preliminary shipments.
When a confirmed tour is accepted by Saga, Shipments are created from it.

## Steps
1. Authenticate (see einride-authenticate.md); use the parent `spaces/{space}`.
2. Create the tour: `POST /v1beta1/{space}/tours` (`BookingService_CreateTour`)
   with `sender`, `serviceType` (e.g. `FULL_TRUCK_LOAD`), `preliminaryShipments[]`
   (each with `externalReferenceId` + `units[]`), and ordered `stops[]`
   (`address`, `stopType`, `requestedStartTime`, `requestedEndTime`,
   `shipmentExternalReferenceIds`). Optionally set `externalReferenceId` and
   `automationRule` (`CREATE_SHIPMENTS` default, or `CREATE_AND_RELEASE_SHIPMENTS`).
3. Confirm it: `GET /v1beta1/{tour}:confirm` (`BookingService_ConfirmTour`). Once
   accepted, `createdShipments[]` is populated. Reconfirming returns InvalidArgument.
4. Fetch status any time: `GET /v1beta1/{tour}` (`BookingService_GetTour`); list with
   `BookingService_ListTours`; find by reference with `BookingService_SearchTours`
   (`query` matches `tour_id` / `external_reference_id`).
5. Update (limited fields once confirmed): `PATCH /v1beta1/{tour.name}`
   (`BookingService_UpdateTour`). Cancel: `GET /v1beta1/{tour}:cancel`
   (`BookingService_CancelTour`).

## Rules
- Resource names are canonical: `spaces/{space}/tours/{tour}`.
- `externalReferenceId` must be unique within the space — use it to de-dupe and to
  look tours back up via `SearchTours` (there is no idempotency-key header).
- Errors are `google.rpc.Status` (code/message/details) — see errors/einride-problem-types.yml.
