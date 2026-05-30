## Builder Track Weekly Report — Week 18

**Name:** Nnadozie Clara  
**Week Ending:** 05-30-2026  

**Repository:** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)

---

### What we achieved

**Chain & testnet**  
Redeployed escrow contracts on CKB testnet and updated deployment metadata so the app targets the current testnet deployment. Fixed the **release spend** bug in `buildSpendTx` and escrow cell handling so payout transactions build and broadcast correctly after work is approved or a dispute is resolved in favor of the freelancer. Improved **chain transaction modal** UX (`ChainTransactionModal`) with clearer prepare → sign → broadcast → report steps and consistent workspace modal styling for on-chain actions.

**Realtime**  
Added Socket.io on the backend and wired the frontend so the product updates live without full page reloads. **Live chat** — messages appear instantly in workspace chat and the messages hub. **Live timeline** — new timeline events show as job state changes. **Live milestones** — milestone status updates in the workspace as the contract progresses.

**Workspace & messages**  
Redesigned the escrow workspace: action panel, header, milestones, timeline, and agreement snapshot. **Separated chat from the workspace** — chat is reached via dedicated message threads instead of being embedded in the workspace shell. **Chat UI redesign** — cleaner thread layout and improved message composer. **Shared workspace modals** — consistent dialogs for amend terms, dispute, confirm, and chain transactions.

**Dashboard polish**  
**Layout and sidebar restructure** — client, freelancer, and admin dashboards share the same shell pattern. **Navbar** — global search, notifications menu, network badge, and profile chip in the header. **Shared overview components** — KPI strip, recent activity feed, and recent messages preview on client and freelancer home screens. **Contracts table** — single table with horizontal scroll on mobile and tablet. **Bucket tabs** — evenly spread full-width tabs on contract lists.

**Escrow timeout**  
**Configurable `escrowTimeoutHours` on publish** — clients choose the grace period after delivery before timeout refund is allowed; stored in the database and shown in terms. **Red countdown in workspace** — after the delivery date, the workspace header shows a visible timeout countdown so both parties know when timeout becomes eligible.

**Admin dashboard (Week 17 carry-over — done this week)**  
Built the full admin panel to match client/freelancer dashboard design:

- **Email/password auth + seed script** — `AdminUser` table, password hashing, `npm run seed:admins` from env; optional legacy wallet login retained.
- **Overview** — dispute KPIs, platform listing counts, total contracts, recent moderator activity.
- **Users** — paginated list and user detail pages.
- **Dispute console** — dedicated route (`/admin/disputes/:disputeId`) before jumping into job workspace.
- **All contracts** — platform-wide jobs table with inspect and archive for open listings.
- **Platform jobs** — publish demo/marketplace listings as the platform client.
- **Activity log** — admin audit trail of moderator actions.
- **Settings / Help** — password change and moderator playbook.
- **User moderation** — suspend party, hide from talent search, archive a user’s open job listings.
- **Audit logging + backend enforcement** — admin actions written to `AdminAuditEvent`; suspended parties blocked on authenticated writes; hidden addresses filtered from Find talent.

**Achieved the full MVP Demo & testnet UX (Week 17 carry-over — done this week)**  
**Two-wallet E2E demo with explorer hashes** — screen recording of the full happy path on testnet: client and freelancer wallets, fund escrow, work submission, and release (or dispute path) with transaction hashes visible in the timeline and linked to the CKB explorer. **Browser fund reliability** — improved MetaMask/CCC fund flow and error messaging so Fund escrow works consistently in the browser. **Faucet popup at 0 balance** — when a user tries to fund escrow or seal a deal with zero CKB, the app shows guidance and directs them to [https://faucet.nervos.org/](https://faucet.nervos.org/) to obtain testnet CKB.

---

### Priority for Week 19

1. **Test dispute release** — open a dispute from the workspace, resolve it in the admin dispute console (release to freelancer, refund to client, reject), and confirm on-chain payout or refund with tx hashes in the timeline and explorer links.
2. **Test timeout release** — run a contract past delivery + escrow timeout grace period and verify timeout refund/settlement works end to end on testnet with correct status changes and chain evidence.
3. **Full testing of the admin dashboard** — walk through every admin flow: login, overview KPIs, users list and user detail moderation, dispute inbox and console, all contracts, platform job publish and archive, activity log, settings (password change), and help; confirm audit events and enforcement (suspend, hide from talent) behave as expected.

---

### Evidence

Screen recording: two-wallet E2E demo with fund and payout hashes in the UI and CKB explorer links.

**Repository:** [https://github.com/Brielle28/escrow](https://github.com/Brielle28/escrow)
