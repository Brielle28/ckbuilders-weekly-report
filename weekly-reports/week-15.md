## Builder Track Weekly Report — Week 15

**Name:** Nnadozie Clara  
**Week Ending:** 05-10-2026  

**Repository (all code & evidence lives here):** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)

---

### This week's deliverables

This week we shipped **three concrete product surfaces** plus **end-to-end wallet auth (browser ↔ API)**:

1. **Landing page** — A full marketing landing experience built as composable sections under **`frontend/src/components/LandingPage/`** (for example **Hero**, **TrustBar**, **ExploreTalent**, **Safety**, **HowItWorks**, **ReadyToWork**), with shared layout **`LandingLayout`**, **`Navbar`**, **`Footer`**, and supporting **`utils/LandingPage/`** data/copy. The page is routed at **`/`** and presents the product narrative before wallet actions.

2. **Market area (job listing)** — A dedicated **job market** at **`/jobs`**: photo hero with search + **Find work** (scrolls to results), a **Categories** strip **below** the hero (category chips + **“N open roles”** on one row), client-side **search + category filtering**, and **`JobCard`** grid backed by **`utils/MarketJobs/marketJobsData.ts`** (mock listings until an API exists).

3. **Job detail** — Each listing opens **`/jobs/:jobId`** with **`JobDetailView`** (brief, skills, sidebar with budget/timeline, CTA toward **`/connects`**), **`JobNotFound`** for unknown ids.

4. **Connect wallet flow — frontend + backend (together)**  
   - **Frontend:** **`ConnectWalletPage`** (`/connects`) connects via **`@ckb-ccc/connector-react`**, lets the user choose **client vs freelancer**, signs the server-issued message, and stores the returned session (**`frontend/src/utils/auth/session.ts`** → **`establishWalletSession`**, **`loadWalletSession`**, **`clearWalletSession`**). UI is split into **`components/wallet/ConnectFlow/`** (**`ConnectWalletIntro`**, **`WalletConnectionStep`**, **`RoleSelectionStep`**, **`RoleOption`**) with helpers **`utils/wallet/connectFlow.ts`** and **`signMessage.ts`**. **`RequireWalletSession`** gates **`/dashboard/client`** and **`/dashboard/freelancer`**.  
   - **Backend:** **`backend/src/server.ts`** exposes **`POST /api/auth/challenge`**, **`POST /api/auth/verify`**, **`GET /api/auth/session`**, **`DELETE /api/auth/session`** with JSON body/signature normalization (string or byte arrays → hex). **`backend/src/auth/store.ts`** holds **challenges** and **Bearer sessions** in memory (TTLs, **`buildAuthMessage`**). **`backend/src/auth/types.ts`** aligns roles with the frontend. **CORS** targets the Vite dev origin (**`FRONTEND_ORIGIN`**, default **`http://localhost:5173`**); the frontend uses **`/api`** via **`VITE_API_BASE_URL`** or the Vite proxy to reach **`PORT`** (default **8787**).

5. **Navigation & polish** — **`Find Work`** in the navbar is a **direct link** to **`/jobs`** (no dropdown). **`FloatingAppButton`** uses **`startTransition`** when clearing address on signer disconnect to satisfy strict React hooks / compiler expectations.

---

### Where this work lives (mapped to files)

| Theme | Where it lives |
|-------|----------------|
| **Landing marketing UI** | **`frontend/src/pages/LandingPage.tsx`**, **`components/LandingPage/**`**, **`utils/LandingPage/**`**, **`layouts/LandingLayout.tsx`** |
| **Job discovery UI** | **`pages/MarketJobsPage.tsx`**, **`pages/JobDetailPage.tsx`**, **`components/MarketJobs/**`**, **`utils/MarketJobs/**`**, **`AppRouter.tsx`** (`/jobs`, `/jobs/:jobId`) |
| **Wallet session contract** | **`frontend/src/utils/auth/session.ts`** ↔ **`backend/src/server.ts`** + **`backend/src/auth/store.ts`** |
| **CCC / signing** | **`utils/wallet/signMessage.ts`**, **`ConnectWalletPage`**, **`ConnectWallet`** (navbar chip) |

---

### Evidence / how to verify

https://github.com/user-attachments/assets/a44149ca-6b32-4b8e-abf0-7812f6c5e80e

https://github.com/user-attachments/assets/9202e741-7341-4d2c-8a3e-6e75cdf129cf

---

### Plan for Week 16 — **client & freelancer dashboards** (primary)

**Goal:** Move from placeholder shells to **real dashboard UX** for both roles:

- **`/dashboard/client`** — client-specific layout, navigation, and starter widgets (e.g. posted work / escrow overview placeholders wired to future API).
- **`/dashboard/freelancer`** — freelancer-specific layout, navigation, and starter widgets (e.g. applications / earnings placeholders).

**Out of scope unless blocking:** new Rust contract changes; full production auth (signature crypto verification on server is still marked **TODO** in **`server.ts`** — acceptable for internal demos but noted for hardening later).

**Goal for the Week 16 report:** screenshots or short demo notes for **both** dashboards plus a concrete list of **routes/components** shipped; optional stretch — **first wallet-driven escrow action** in the UI if the dashboards expose it.

---

### Environment snapshot

- **Frontend:** Vite (**`localhost:5173`**), React 19, **`react-router-dom`**, **`@ckb-ccc/connector-react`**, Tailwind v4.  
- **Backend auth:** Express (**`PORT`** default **8787**), **`FRONTEND_ORIGIN`** for CORS.  
- **Chain:** Same as prior weeks — integration runner / devnet for contract proof; dashboards this week are **UI-first**.
