## Builder Track Weekly Report — Week 16

**Name:** Nnadozie Clara  
**Week Ending:** 05-16-2026  

**Repository:** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)

---

### How this week built on Week 15

Last week we had the landing page, a mock job board, and a first wallet connect flow that lived at `/connects` with sessions kept only in memory on the server. Week 16 was about turning that into something you can actually use day to day: real dashboards for clients and freelancers, jobs and talent coming from the API instead of hard-coded lists, and login that survives a backend restart.

We also closed out the “fixes and foundations” slice of the V1 plan—cleaner routes (`/connect` with a redirect from the old URL), a single place to guard logged-in pages, Postgres for sessions, and proper signature checks instead of accepting any fake signature in dev. That last change is why some wallets now show “Signature rejected” until we finish tuning every signer format; it is the trade-off for real security.

---

### What we achieved

**Dashboards (the Week 15 goal)**  
Clients and freelancers each have their own dashboard shell: sidebar navigation, wallet address in the header, and real pages instead of empty placeholders. Clients can see an overview, browse contracts, publish new work, and open personal info. Freelancers get the same structure with freelancer-focused nav. You can switch between client and freelancer from the dashboard in a modal without being kicked out to a separate connect page—the wallet stays connected and you only sign again for the new role.

**Marketplace wired to the backend**  
Find Work and Find Talent now load from the API. Publishing a job saves to the database. Job cards and detail pages reflect what the server returns, with a small static fallback in development if the API is not running. The publish form is smarter: the short card text is taken from the description, and the delivery date can follow the timeline you pick.

**Profiles and talent**  
Freelancers can fill in personal info in the dashboard—about, skills, employment, education, certifications, and a new projects section with links that show on the public talent profile. Avatars upload through the API and display correctly in preview and on Find Talent. New profiles start at zero job success and zero completed jobs until they actually finish work on the platform. We fixed small UX issues along the way (hourly rate inputs no longer stick a leading zero when you type “50”, local time fills from the device clock).

**Auth and app health**  
Sign-in is challenge → sign in wallet → verify → token stored in the browser. Sessions live in Postgres, so restarting the backend does not log everyone out. After connect, you can land back on the page you were trying to reach. A network banner on the dashboard warns when the wallet chain or deployment config does not match what the app expects.

**Under the hood**  
The frontend now talks to the backend through a shared Axios client and Redux for jobs, talent, profile, and dashboard data. Shared types live in a small `packages/shared` package so client and server agree on roles, job status, and categories (we settled on five: Development, Design, Writing, Marketing, Other).

---


### Plan for Week 17 — escrow MVP demo only

Week 17 has one focus: **ship and demonstrate the full escrow MVP**—nothing else.

That means a single believable path a reviewer can watch or click through:

1. Client posts work and funds escrow on-chain.  
2. Freelancer is matched (or selected) and both sides confirm terms.  
3. Work happens in the **workspace** for that job—timeline, status, and the right button for each step.  
4. Freelancer submits; client releases **or** a dispute / timeout path is shown where the contract allows it.  
5. Transaction hashes and explorer links appear in the UI so it is obvious the money moved on CKB, not only in the database.

We will not spend Week 17 on extra marketing pages, profile polish, or nice-to-have dashboard widgets unless they block that demo. The success criterion for the Week 17 report is simple: **a recorded or live demo of escrow working end to end**, with notes on which network we used and which txs proved fund and payout.

---

### Stack (unchanged in spirit)

React + Vite frontend, Express + Prisma + PostgreSQL backend, Rust escrow contract already proven via the integration runner, wallet via CCC on testnet (or devnet) aligned with `deployment/scripts.json`.
