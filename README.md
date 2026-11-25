# Receipt OCR: Frontend

React web client for a receipt-scanning and household expense-tracking app. Upload a photo of a receipt, review the extracted items, and track spending with the people in your household.

The API and OCR pipeline live in a separate repository: [OCR_Backend](https://github.com/sziki12/OCR_Backend). It must be running for this client to work.

## Features

- **Scan receipts:** upload a photo and pick the OCR engine, parsing model and image orientation. The extracted store, date and items appear right away.
- **Review and edit:** browse receipts, edit items in a data grid, and compare the original image with the raw OCR output.
- **Places:** attach a place to a receipt and see places on a Google Map.
- **Households:** switch between households and invite other people by email. Receipts are shared inside a household.
- **Spending charts:** per-household statistics with MUI X Charts.
- **Light and dark themes**
- **Auth:** registration, login, and silent token refresh. If the session can't be refreshed, the user is redirected to the login page.

## Tech stack

| Area | Technology |
| --- | --- |
| UI | React 18 (Create React App), MUI 5, MUI X Data Grid and Charts, Tailwind CSS |
| Routing | React Router 6 |
| API layer | TypeScript, `fetch`, React context |
| Maps | Google Maps (`@vis.gl/react-google-maps`) |
| Auth | JWT bearer tokens, client-side `bcryptjs` password hashing |

## Architecture notes

- **No global store.** Shared state and API access are React contexts stacked in `src/components/handlers/MainHandler.jsx`: endpoints → theme → login → household → receipts.
- **One endpoint module per backend resource** in `src/endpoints/` (receipts, items, places, households, statistics and so on). Each exposes a hook, and components never call `fetch` directly.
- **Central auth handling.** Every request goes through `callAndEnsureLogin` in `src/services/AuthService.jsx`. On a failure it refreshes the token once (guarded against concurrent refreshes), retries, and logs the user out if that fails.
- **Route table by auth state.** `src/routes/RouteProvider.js` drops protected routes for signed-out users, and login or logout rebuilds the router.
- **Household-scoped API.** Every call targets `/api/household/{householdId}/...`.

## Getting started

### Prerequisites

- Node.js 18+
- The [backend](https://github.com/sziki12/OCR_Backend) running on `http://localhost:8080`. The URL is set in `ocr_frontend_react/src/endpoints/BackendAccess.ts`.
- A Google Maps API key for the map views (optional for everything else)

### Run

```bash
cd ocr_frontend_react
npm install
echo "REACT_APP_Google_Maps_API_Key=<your key>" > .env   # optional, enables maps
npx tsc -p .     # compiles the TypeScript API layer into src/dist
npm start        # http://localhost:3000
```

The `tsc` step is required. The components import the compiled output from `src/dist/`, which is git-ignored, so run it again after changing anything in `src/endpoints`, `src/services` or `src/types`.

### Other commands

```bash
npm run build
npx tailwindcss -i ./src/css/index.css -o ./src/css/output.css --watch   # regenerate Tailwind CSS
```

## Status

This was my BSc thesis project. There are no automated tests yet. The included `Dockerfile` and `docker-compose.yaml` are an unfinished attempt and don't compile the TypeScript step, so run the app locally as described above.
