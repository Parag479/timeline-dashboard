# Timeline Dashboard Notes

## Run

```bash
npm install
# copy .env.example to .env and adjust VITE_API_BASE_URL if needed
npm run dev
```

## Session and API

The access token returned in the JSON login response is stored in `sessionStorage`. This keeps a refresh within the browser session convenient while avoiding a long-lived token in persistent storage; the trade-off is that closing the tab ends the session. All requests go through the `api` helper, which adds `Authorization: Bearer <token>`, unwraps the MES `{ data }` envelope, retries HTTP 500 responses twice with backoff, and emits a session-expired event for any authenticated 401. Startup calls `/auth/me` before rendering the dashboard. Logout calls `/auth/logout` and clears the token.

## Chart performance

The timeline is rendered on a single canvas rather than one DOM/SVG node per produce. Timestamps are converted to pixel geometry only during the canvas draw pass, and the canvas is capped at 18,000 plotted points. When the backend returns more points than that, the stride is applied only to PASS points; FAIL points are never dropped. Dragging selects a range for zoom and double-click resets it. The chart has no polling, so the only expensive operation is an explicit data refresh or filter change.

## Time and table math

The selected date and shift start times are interpreted as local IST (`+05:30`). Cross-midnight shifts advance the end date, and the request sends `Date.toISOString()` UTC values. API timestamps are retained as UTC instants and formatted with `Intl.DateTimeFormat` using `Asia/Kolkata` for axis and table labels. Segment minutes are calculated by intersecting every segment with each hourly bucket, so a segment spanning boundaries contributes only its clipped portion to each hour. Counts are grouped by bucket start and summed across part models. Cycle metrics come from the separate hourly `/analytics-query` call and nullable values render blank. Future buckets are blank for an in-progress shift.

## Scope and assumptions

The asset tree is flattened into a single selector so any returned node can be queried without implementing an out-of-scope hierarchy browser. Shift choices are flattened from every active backend shift definition and its dynamic `shift_timings`; no A/B/C names are hard-coded. The UI intentionally omits segment classification, export, polling, and settings as requested. Filter data is fetched from the live backend after session validation; analytics remains empty until a real authenticated request succeeds.
