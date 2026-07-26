---
name: Find available Dott vehicles in a city
description: Discover Dott cities and read real-time e-scooter/e-bike availability, types and pricing from the public GBFS 2.3 feeds.
api: openapi/dott-gbfs-openapi.yml
operations: [getGbfsDiscovery, getSystemInformation, getVehicleTypes, getSystemPricingPlans, getFreeBikeStatus, getGeofencingZones]
---

# Find available Dott vehicles in a city

Dott exposes its fleet through the open **GBFS 2.3** feeds at
`https://gbfs.api.ridedott.com/public/v2`. All calls are unauthenticated HTTP `GET`s that
return the GBFS envelope `{ last_updated, ttl, version, data }`. Respect `ttl` (seconds) before
refetching. Public vehicle IDs rotate; use the authenticated partner feeds if you need stable IDs.

## Steps

1. **Discover cities** — call `getGbfsDiscovery` (`GET /gbfs.json`). Read `data.<lang>.feeds[]`
   and extract the city slug from each feed URL (`/public/v2/{city}/...`).
2. **Confirm the city** — call `getSystemInformation` (`GET /{city}/system_information.json`) to
   verify the slug and read `system_id`, `name`, `timezone`. A `404`
   `{"error":"Not Found: ERR_REGION_NOT_FOUND"}` means the city is not served.
3. **Learn the fleet** — call `getVehicleTypes` (`GET /{city}/vehicle_types.json`) for
   `form_factor` (scooter/bike), `propulsion_type`, `max_range_meters` and the linked
   `pricing_plan_ids`.
4. **Read pricing** — call `getSystemPricingPlans` (`GET /{city}/system_pricing_plans.json`);
   join `plan_id` from step 3 to get `currency`, unlock `price` and `per_min_pricing`.
5. **Find vehicles now** — call `getFreeBikeStatus` (`GET /{city}/free_bike_status.json`).
   Filter `is_disabled == false` and `is_reserved == false`; use `lat`/`lon`,
   `current_range_meters` and `rental_uris` (deep links) to surface the nearest usable vehicle.
6. **Respect zones (optional)** — call `getGeofencingZones` (`GET /{city}/geofencing_zones.json`)
   and honor each feature's `rules` (`ride_allowed`, `ride_through_allowed`, `station_parking`).

## Rules
- Read-only: there are no write/booking operations in the public API (see `conventions/dott-conventions.yml`).
- Cache to `ttl`; do not poll faster than the feed refreshes.
- Errors are a flat `{ "error": string }` envelope (see `errors/dott-problem-types.yml`).
