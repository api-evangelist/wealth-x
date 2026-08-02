---
name: Screen a contact against Wealth-X and pull the dossier
description: >-
  Given a person's or company's name, search the Wealth-X database, then
  retrieve the matching wealth-intelligence dossier so an agent can qualify a
  UHNW/VHNW prospect and read their net worth and known associates.
api: openapi/wealth-x-connect-openapi.yml
operations: [searchDossiersAdvanced, getDossier]
---

# Screen a contact against Wealth-X and pull the dossier

Use this to qualify a prospect: find their Wealth-X dossier by name (or company),
then read the full record.

## Authentication
Every request sends three headers together: `username`, `password`, and
`apikey` (see `authentication/wealth-x-authentication.yml`). There is no OAuth
and no idempotency key.

## Steps
1. **Search by name.** `POST /dossiers/search/advanced` (`searchDossiersAdvanced`)
   with a JSON body such as `{ "firstName": "...", "lastName": "...",
   "exactNameOnly": true, "maxRecords": 25 }`. For a company, use
   `{ "companyName": "...", "maxRecords": 25 }`. Page large result sets with
   `page` / `pageSize` and sort with `orderBy` / `sortDirection`.
2. **Pick the match.** Read `dossierId` from the results. Disambiguate with
   geography (`countryID`, `stateID`, `locationType`) or net-worth floors
   (`networthMin`, `tafMin`) in the search body if several match.
3. **Pull the dossier.** `GET /dossiers/{dossierId}` (`getDossier`). Use
   `view=selective` with `fields=netWorth,knownAssociates` to fetch just the
   fields you need, or `view=stub` for a lightweight record.
4. **Walk the network (optional).** Follow `knownAssociates` from the dossier to
   related dossierIds and repeat step 3 to map the prospect's relationship graph.

## Conventions and errors
- Reference IDs come from `GET /countries`, `GET /countries/{countryId}/states`,
  `GET /industrytypes`, and `GET /positions`.
- `401` means the three auth headers are missing or invalid; `404` means no
  dossier for that ID; `400` means malformed search criteria
  (see `errors/wealth-x-problem-types.yml`).
