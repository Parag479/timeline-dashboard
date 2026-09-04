# Pulseboard Timeline Dashboard

React 18 + TypeScript + Vite implementation of the Timeline Dashboard assignment.

## Run locally

```bash
npm install
Copy-Item .env.example .env
npm run dev
```

Open `http://localhost:5173/` and sign in with the supplied assignment credentials:

- Username: `analytics_user`
- Password: `dashboard123`

The API base URL defaults to the assignment backend. Override it with `VITE_API_BASE_URL` in `.env`.

## Included

Authentication and refresh-time session validation, centralized bearer-token API calls, dynamic asset and shift filters, IST-aware UTC requests/display, canvas timeline rendering, fail-preserving large-point handling, drag zoom/reset, manual refresh, retry/error states, and the hourly production/downtime/cycle-time table.

See [NOTES.md](NOTES.md) for implementation decisions and assignment assumptions.
