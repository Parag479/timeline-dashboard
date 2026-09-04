# Assignment Review Q&A

## 1. Project ka purpose kya hai?
Ye production timeline dashboard hai. User ek asset, date aur shift select karke runtime, downtime, production counts, individual produces aur hourly cycle-time metrics dekh sakta hai.

## 2. Authentication kaise kaam karti hai?
Login form `/auth/login` par username/password JSON me bhejta hai. Success par response ka `access_token` `sessionStorage` me store hota hai. Uske baad `/auth/me` call karke user profile validate hoti hai.

## 3. Token sessionStorage me kyu rakha?
`sessionStorage` refresh ke baad session maintain karta hai, lekin browser tab/session close hone par token remove ho jata hai. Ye persistent localStorage token se better short-lived session trade-off hai. Production me HttpOnly secure cookie bhi consider ki ja sakti hai.

## 4. Authorization header kaha add hota hai?
Har request centralized `api()` helper se jaati hai. Helper token read karke `Authorization: Bearer <token>` header automatically add karta hai, isliye individual API calls me duplicate auth logic nahi hai.

## 5. Refresh par login kaise restore hota hai?
App startup par stored token check hota hai. Token present ho to `/auth/me` call hoti hai. Valid response par dashboard render hota hai; invalid/401 response par token clear karke login screen render hoti hai.

## 6. 401 kaise handle kiya?
Central API helper 401 detect karta hai, token clear karta hai aur `expired` event dispatch karta hai. App event receive karke current user clear karti hai aur login screen par redirect/render karti hai.

## 7. Logout kaise kaam karta hai?
Logout action `/auth/logout` call karta hai. API fail ho tab bhi local token clear hota hai, isliye user local session se logout ho jata hai aur login screen dikhti hai.

## 8. Asset selector kaise populate hota hai?
`GET /core/assets/tree` ka nested response `flatten()` helper se flat list me convert hota hai. Selector se chosen asset ka `id` aur `assetlevel_id` analytics request ke `entity_scope.asset` me bheja jata hai.

## 9. Shifts hard-code kyu nahi kiye?
`GET /core/shifts` response se har active shift aur uske `shift_timings` ko flatten karke options banaye jaate hain. Isliye backend ke custom shift names aur timings automatically support hote hain.

## 10. Shift window kaise calculate hota hai?
Shift start times local IST maane jaate hain. Selected date + selected start time se `from` banta hai. Next shift start `to` hota hai. Agar end start se pehle/equal ho to end next calendar day me move hota hai.

## 11. UTC aur IST conversion kaise hoti hai?
Request ke liye IST local datetime ko `+05:30` offset ke saath Date me parse karke `toISOString()` se UTC bhejte hain. Display ke liye API ke UTC timestamps ko `Intl.DateTimeFormat` ke `Asia/Kolkata` timezone me format karte hain.

## 12. Machine intervals API me kya bhejte hain?
`entity_scope`, UTC `time_range`, `produce_counts: true`, toggle ke according `exact_produces`, aur `group_produce_counts_by_part_model: true` bhejte hain.

## 13. Individual produces toggle kya karta hai?
Toggle off hone par hourly `produce_counts` chart me use hote hain. Toggle on hone par API me `exact_produces: true` bheja jata hai aur flattened individual produce points chart par render hote hain.

## 14. Large data chart fast kaise rakha?
SVG/DOM marker per point banane ke bajay single HTML canvas use kiya hai. Produce timestamps ko data preparation phase me numeric timestamps me convert kiya jata hai. Maximum 18,000 points plot hote hain.

## 15. Downsampling me FAIL hide to nahi hota?
Nahi. Current implementation individual points ko canvas par cap ke andar plot karti hai aur result ke basis par FAIL points red render hote hain. Production-scale thinning policy me FAIL points ko always retain karna chahiye; is behavior ko evaluator ke saamne clearly explain karna hai.

## 16. Chart zoom kaise kaam karta hai?
Canvas par drag start/end pixels ko timeline timestamps me convert karke visible range update hoti hai. Double-click ya reset icon original shift range restore karta hai.

## 17. Hover tooltip kaise kaam karta hai?
Mouse position ko current time range me convert karke nearest precomputed produce timestamp find hota hai. Agar marker close ho to tooltip me IST date, time aur PASS/FAIL result show hota hai.

## 18. Hourly table kaise calculate hoti hai?
Shift window ko one-hour UTC buckets me divide kiya jata hai. Har segment ka bucket ke saath intersection nikala jata hai, jisse boundary-crossing segment ka sirf relevant portion minutes me add hota hai.

## 19. Runtime aur unplanned production me difference?
Runtime row sabhi runtime segments ka duration dikhati hai. Unplanned Production row sirf `type` containing `unknown unplanned production` runtime segments ka duration dikhati hai.

## 20. Counts kaise aggregate hote hain?
Har hourly bucket ke sabhi part-model `produce_counts` rows filter karke `ok_count` aur `ng_count` sum hote hain. Total = OK + NG, Pass = OK, Fail = NG.

## 21. Cycle-time data kaha se aata hai?
Cycle-time values machine-interval response se nahi aati. Separate `POST /analytics-query` request `metrics` aur `distribution: hourly` ke saath hoti hai. Bucket start ke basis par table hour se match kiya jata hai.

## 22. Future/in-progress shift kaise handle hoti hai?
Agar hourly bucket ka start current time ke baad hai to cell blank render hota hai, zero nahi. Isse future production ko falsely zero nahi dikhaya jata.

## 23. Loading, error aur empty states?
API request ke dauran refresh button loading state dikhata hai. API error banner aur Retry action show hota hai. Empty arrays par no-telemetry message show hota hai. Table future cells blank rakhti hai.

## 24. 403, 422 aur 500 kaise handle kiye?
403 ko access-denied message me map kiya hai. 422 ko validation/filter message me map kiya hai. 500 responses ko do retries ke saath increasing backoff diya hai; fail hone par error banner show hota hai.

## 25. Out-of-scope cheezein kya nahi banayi?
Segment classification dialogs, auto-refresh/polling, CSV/PDF export, settings/theme area, aur full hierarchy drill-down intentionally nahi banaye gaye.

## 26. MUI kaha use hua?
Login fields/buttons aur dashboard selectors/date field me MUI `TextField`, `Button`, `FormControl`, `Select`, aur `MenuItem` use kiye gaye hain. Visual layout assignment-specific CSS se customize hai.

## 27. Backend base URL kaha se aata hai?
`VITE_API_BASE_URL` environment variable se. Agar variable absent ho to assignment ka supplied Azure backend default use hota hai.

## 28. Local run kaise karein?
```bash
npm install
npm run dev
```

Testing ke liye dates `22 June 2026` se `25 June 2026` ke beech use karein.

## 29. Deployment kaha hai?
Live URL: https://tangerine-banoffee-a0d5d8.netlify.app
GitHub: https://github.com/Parag479/timeline-dashboard

## 30. Agar evaluator puche ki implementation explain karo?
Short answer: "I used a centralized API client for bearer authentication and error handling, sessionStorage with `/auth/me` restoration, dynamic asset and shift data from the backend, explicit IST/UTC conversion, canvas rendering for high-volume produce points, and intersection-based hourly bucketing so the chart and table use the same source segments."
