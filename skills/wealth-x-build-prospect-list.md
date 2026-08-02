---
name: Build a targeted UHNW prospect list from Wealth-X
description: >-
  Assemble a segmented list of ultra-high-net-worth prospects from the Wealth-X
  database using geography, net-worth floors, industry, position, interests,
  education, and philanthropy criteria, then bulk-export the matching dossiers.
api: openapi/wealth-x-connect-openapi.yml
operations: [listCountries, listStates, listIndustryTypes, listPositions, searchDossiersAdvanced, getAllDossiers]
---

# Build a targeted UHNW prospect list from Wealth-X

Use this to generate a prospecting segment (e.g. "UHNWIs in NY State with net
worth ≥ $100M in Financial Services") and export it.

## Authentication
Send `username`, `password`, and `apikey` headers on every request
(`authentication/wealth-x-authentication.yml`).

## Steps
1. **Resolve reference IDs.** Call `GET /countries` (`listCountries`) and
   `GET /countries/{countryId}/states` (`listStates`) for geography IDs,
   `GET /industrytypes` (`listIndustryTypes`) for industry filters, and
   `GET /positions` (`listPositions`) for position filters.
2. **Run the segmented search.** `POST /dossiers/search/advanced`
   (`searchDossiersAdvanced`) with the criteria you resolved, e.g.
   `{ "countryID": 1, "stateID": <NY>, "locationType": "...",
   "networthMin": 100000000, "tafMin": 100000000, "industryType": "...",
   "position": "...", "page": 1, "pageSize": 100,
   "orderBy": "netWorth", "sortDirection": "desc" }`. You can also target
   `hobbies`, or `keyword` + `section` for education / philanthropy /
   relationship segments. Set `idOnly: true` to get just the dossierIds.
3. **Page through the results.** Increment `page` until fewer than `pageSize`
   records return, collecting dossierIds.
4. **Bulk-export.** `GET /alldossiers` (`getAllDossiers`) with
   `dossierIds=<comma-separated ids>` (or an index range via
   `fromIndex`/`toIndex`, or a net-worth band via `networthMin`/`networthMax`).
   Shape the export with `view` and `fields`.
5. **Keep it fresh.** For incremental refreshes, call `GET /alldossiers` with
   `lastModifiedFrom` / `lastModifiedTo` (optionally `idOnly=true`) to pull only
   changed records, bounded by `GET /lastdossierid`.

## Conventions and errors
- Pagination on search is `page` / `pageSize`; bulk uses `fromIndex` / `toIndex`
  / `size` (see `conventions/wealth-x-conventions.yml`).
- `401` = bad/missing auth headers; `400` = malformed criteria
  (`errors/wealth-x-problem-types.yml`).
