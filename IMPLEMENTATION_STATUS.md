# Implementation status

Last verified: 2026-10-26, PostgreSQL 17.2 (local, port 5433), Node 24.

## Done
- **Database:** migrations 001–003, views, indexes, idempotent non-destructive seed (10 departments / doctors / patients / medicines / rooms / resources, 19 appointments, prescriptions, allocations, audit rows, receptionist + admin accounts).
- **Backend:** four roles (patient, doctor, receptionist, admin) with server-side authorization; logout/role-change/deactivation invalidate tokens; auth rate limiting; paginated, searchable list endpoints; full CRUD for departments, doctors, patients, staff, rooms, equipment, medicines with deactivation rules; slot generation (schedule − leave − appointments − requested resources, booking window); schedule/leave conflict detection; appointment transitions (confirm, complete, no-show, cancel, reschedule); atomic booking/cancel/reschedule/prescription/account creation; reports (dashboard, daily schedule, department counts, doctor workload, resource utilization, patient history); audit log.
- **Frontend:** redesigned (Tailwind design system, lucide icons, responsive sidebar layout, toasts, modals, confirmation dialogs, paginated tables). Patient, doctor, receptionist and administrator portals, including **Manage Hospital Data** with eight tabs.
- **Tests:** 56 integration tests on a real PostgreSQL database, including concurrency tests with separate connections.

## Commands run and results
| Command | Result |
|---|---|
| `npm run db:reset` | migrations 001–003 + seed applied |
| `npm run db:seed` run twice more | row counts unchanged in all 15 tables (also asserted by a test) |
| `npm run typecheck` | server, client, tests: no errors |
| `npm test` | 4 files, **56 passed**, 0 failed |
| `npm run build` | server `tsc` and client `vite build` succeeded |
| compiled `node server/dist/index.js` | started; unauthenticated request 401; reception login 200 |
| every query in `database/reports.sql` | executed successfully against the seeded dev DB |
| Browser (built-in pane, Chrome-like): admin, patient, doctor, receptionist logins; admin Add department; full patient booking flow; booking-conflict path (second patient takes the slot before confirmation); receptionist blocked from `/staff/manage`; 375px mobile layout (no horizontal overflow on dashboard / appointments) | worked as described |

## Not verified / limitations
- Frontend interactions not exercised in a browser: reschedule dialog, doctor prescribe/complete/no-show buttons, patient edit/history dialogs, resource allocation dialog, staff-account and medicine dialogs, leave editor, admin doctor edit. Their APIs are covered by the integration tests; the screens type-check and build. There are no automated frontend tests.
- `database/explain_output.txt` was generated on the earlier small seed and not regenerated.
- No least-privilege database role is configured; the app uses the role in `DATABASE_URL`.
- Times are shown and entered in the clinic time zone (`CLINIC_TZ`, default Asia/Kolkata); the `daily_doctor_schedule` view and the seed use that zone literally.
- Sample data is relative to the day it is first seeded (business days around "today"); re-seeding later does not move it (by design, to avoid duplicates) — use `npm run db:reset` for a fresh dataset.
- `npm audit` reports vulnerabilities in dev dependencies; not reviewed.
- Local PostgreSQL provided by `embedded-postgres` (`npm run pg:start`, data in `C:/pgemb`) keeps running until stopped.

## Next steps (optional)
1. Click through the remaining dialogs listed above.
2. Add Playwright (or similar) tests for the main UI flows.
3. Create a restricted database role for the API and document its grants.
