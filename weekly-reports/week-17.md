## Builder Track Weekly Report — Week 17

**Name:** Nnadozie Clara  
**Week Ending:** 05-23-2026  

**Repository:** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)

### What we achieved

**Escrow lifecycle in the database**  
Freelancers can apply from the job detail page (`JobDetailApplyPanel` on Find Work). Clients select a freelancer from the applicant list. Both parties confirm terms from the workspace; the freelancer can submit work when funded. Either party can open a dispute from the workspace (`WorkspaceDisputeDialog`), backed by lifecycle and dispute services—not placeholder toasts. Workspace buttons for confirm terms, submit work, and open dispute call real APIs. We added an automated check: `backend/scripts/phase1-lifecycle-selftest.mjs` (run from `backend/` per `commands.md`).

**On-chain fund and release **  
The backend exposes chain transaction routes under `/api/jobs/:jobId/tx/*` via `chainTx.service`: prepare and report fund, server-broadcast release, and prepare timeout. Chain modules `buildFundTx` and `buildSpendTx` build unsigned fund txs and signed spend txs; jobs store `fundTxHash`, escrow outpoint, and related fields after confirmation. In the workspace, **Fund escrow** and **Release** (and timeout where enabled) open `ChainTransactionModal`, which walks prepare → sign → broadcast → report. Timeline events can carry `txHash` values, and the UI links to the explorer when a hash is present. 

**Workspace beyond the Week 16 shell**  
`EscrowWorkspaceView` is now the operational hub: hero action panel, agreement snapshot, milestones, timeline, cancel fund while `funding_pending`, and amend-terms dialog. The app enforces the CKB network minimum (61 CKB) before funding and explains when a budget is too low. Admins can open the same job workspace in read-only mode with a resolve panel (`AdminJobWorkspacePage`, `AdminResolvePanel`).

**Chat and disputes**  
Workspace chat is real: `WorkspaceChat` talks to the messages API instead of the old placeholder. Client and freelancer dashboards have a messages hub (`MessagesListPage`, `MessageThreadPage`) for threads per job. Admins get a disputes inbox (`AdminDisputesPage`) with resolve outcomes—release to freelancer, refund to client, or reject—with optional on-chain release when resolving in favor of the freelancer. The **Disputed** tab on contract lists is back. Automated check: `phase3-dispute-selftest.mjs`.

---

### Main blocker

### Built but not fully tested yet (because of the fund bug)

All of the above is implemented and can be exercised in parts (API, scripts, some UI steps). We have **not** signed off a full two-wallet walkthrough on testnet because **Fund escrow** in the browser often fails at the CCC/MetaMask stage while:

- `prepareFund` from the API succeeds, and  
- `pnpm debug:fund` with a native private key can broadcast successfully.

---

### Other open gaps (Week 18)

- **Two-wallet E2E demo** with fund + payout hashes recorded (blocked on fund fix first).
- **On-chain dispute refund** and a complete **timeout** product path—partial or integration-only in places.
- **Admin dashboard** — dispute inbox and job workspace exist; activity, help, and settings pages are still stubs.

---

### Priority for Week 18

1. **Fix MetaMask fund reliability** — hard gate for any demo.  
2. **Run and record two-wallet E2E** — fund and release (or dispute) with explorer links in the UI.  
3. **Close chain gaps** — dispute refund and timeout with consistent timeline + hashes.  
4. **Admin dashboard essentials** — yes, finish dispute operations and resolution visibility; expand non-critical admin pages after the happy path is proven on testnet.

---

### Evidence 

chek out the repository:
**Repository:** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)