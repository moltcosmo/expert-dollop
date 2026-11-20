# Print Job Manager 1.0 — UI foundation

## Implemented in this checkpoint

- Local customer creation and customer list.
- Sales Order creation and order detail.
- Project creation under a Sales Order, with its own quantity, unit, selling price, and deadline.
- Sales Order summary derived from its Projects and valid payment transactions.
- Multiple payments per Sales Order and payment voiding (no physical deletion).
- Workflow templates and template stages.
- Copy template stages into a new Project.
- Add Project-specific stages.
- Sequential stage start/completion enforcement in SQLite.
- Project status changes to active when production starts and completed when all stages are complete.
- Stage estimated completion time and PIC fields can be recorded.
- Project and Sales Order list search with 20-record batches and Load More.

## Added in HPP checkpoint 1.1

- HPP component settings UI for Media, Print Process, Finishing, and Other Costs.
- Create and edit reusable cost components.
- Add selected components to an individual Project's HPP.
- HPP breakdown by category, total HPP, and HPP per Project unit.
- Cost formula: `max(quantity * rate, minimum_charge) + setup_charge`.
- Rate and specification snapshots are stored on each Project cost line; editing a master component does not change existing Project calculations.
- Remove a cost line from a Project calculation.

## Added in Paper Layout Optimizer checkpoint 1.2

- Standalone calculations and calculations linked to an individual Project.
- Inputs: source sheet dimensions, cut dimensions, required quantity, edge margin, knife kerf, and a rotation/grain constraint.
- Generates guillotine strip patterns with normal, rotated, and mixed row orientations.
- Pattern result includes yield per sheet, required sheets, waste, efficiency, row sequence, and a schematic diagram.
- Selected layout can feed required plano consumption into Project HPP using a Media component and rate snapshot.

## Added in Purchasing checkpoint 1.3

- Supplier list and supplier creation.
- Purchase entry with optional supplier and multiple line items.
- Direct purchase starts as draft for item entry, then can be marked received.
- Supplier order starts as ordered and can be marked received.
- Purchase history list with supplier, date, status, item count, and total.
- Purchase detail shows line quantities, units, unit prices, and subtotals.
- Voiding is available before receipt; received and void purchases cannot be changed through the UI.
- Purchasing remains independent: no Project link, inventory posting, supplier payment, or accounting.

## Added in Export & Backup checkpoint 1.4

- CSV export for Customers, Sales Orders, Projects, Payments, Purchasing, Suppliers, Workflow templates, Project production stages, HPP components, Project HPP detail, and Paper Layout calculations/patterns.
- CSV is UTF-8 with BOM and escaped fields; file is saved using Android's system document picker, so the user chooses the destination.
- Full SQLite database backup to a `.pjmbackup` file using Android's system document picker.
- Restore from a selected backup file; the incoming database is checked with SQLite integrity_check and required PJM tables before replacing the current database.
- Restore replaces all current local data. It is not a merge operation.
- No third-party libraries or database schema changes.

## Not implemented yet

- Native Android scheduled notifications and notification permission handling.
- Attachments/files UI.
- Full editing/reordering/archive management for master records and workflow stages.

## Export & Backup behavior

- CSV export is one dataset per file, with related records flattened where useful (for example, purchase lines and Project HPP cost lines).
- Timestamp fields are formatted as local date/time text; date-only fields remain as stored.
- CSV export includes all rows in the selected dataset and does not apply the current screen's search/filter.
- Database backup is the complete local SQLite database. Restore validates the SQLite file and core PJM tables, then replaces the app database and reloads the UI.
- The user should keep backup files outside the device. Uninstalling the app removes its private database but does not remove a backup saved to shared storage/cloud storage.

## HPP calculation behavior

- `rate_basis` is a descriptive unit for the quantity entered on a Project cost line (for example, lembar, rim, kg, or jam).
- Variable cost is rounded to the nearest Rupiah after multiplying quantity by rate.
- Minimum charge is applied to the variable cost; setup charge is added afterward.
- HPP is a cost estimate. Selling price remains separate on the Project. The UI displays the difference between selling price and HPP as a simple reference, not a full margin/tax analysis.

## Local data

The app uses a SQLite database named `print_job_manager.db` in private app storage. This prototype has no production data and no migration path from earlier prototypes. Uninstalling the app removes its local database.

## Validation performed

- Executed all 54 SQL statements extracted from `MainActivity.java` against SQLite in-memory.
- Confirmed 16 tables, 31 indexes, 5 triggers, and 2 views are created.
- Smoke-tested duplicate Project selling prices, payment balance, and sequential workflow blocking/advance.
- Android APK compilation and on-device behavior still require GitHub Actions build and phone testing; this environment did not have Gradle/Android SDK installed.

## Added in Paper Layout Optimizer checkpoint 1.2

- Standalone calculations and calculations linked to an individual Project.
- Inputs: source sheet dimensions, cut dimensions, required quantity, edge margin, knife kerf, and a rotation/grain constraint.
- Generates up to three guillotine strip patterns: all normal, all rotated, and mixed row orientations when a genuine mixed solution is found.
- Uses a row-based guillotine model: first divide the plano into full-width strips, then divide each strip into pieces. Kerf is included between adjacent pieces and rows; outer margin is applied on all four edges.
- Pattern result includes yield per sheet, number of sheets required, waste area, waste percentage, efficiency, row sequence, and a schematic row diagram.
- User can select one pattern per calculation.
- For a Project-linked calculation, the required plano sheets can be added to HPP using an existing Media cost component. The HPP line stores the selected layout pattern reference and snapshots the media rate/specification.
- No additional database tables or libraries were added; the existing `layout_calculations`, `layout_patterns`, and `project_cost_lines` tables are used.

## Paper Layout Optimizer limitations

- This is a guillotine strip-layout estimator, not a general 2D nesting engine.
- It considers rows spanning the usable sheet width and allows normal/rotated orientation by row. It does not yet model arbitrary multi-stage cut trees, clamps/gripper zones, bleed, registration marks, machine-specific constraints, or irregular shapes.
- Grain selection currently acts as an orientation restriction: free, normal-only, or rotated-only. The operator must confirm actual grain direction and machine constraints before production.
- The diagram is schematic and is not a dimensioned cutting drawing.
- APK build and physical on-device testing remain pending GitHub Actions and phone validation.

## Purchasing checkpoint 1.3 behavior

- Supplier is optional; purchase line items are free-text descriptions, not inventory products.
- Direct purchase is created as draft so the user can enter all items before marking it received.
- Supplier order is created as ordered; line items can be entered while ordered, then marked received.
- Item entry is blocked once a purchase is received or void.
- Purchase totals are derived from purchase lines, not stored separately.
- No database schema changes or additional libraries were required.
- JavaScript syntax check passed with Node.js. Android APK compilation and device testing remain pending because Gradle/Android SDK are not available in the authoring environment.


## Export & Backup checkpoint 1.4 validation

- JavaScript syntax check passed with Node.js.
- CSV SQL queries were executed against an in-memory SQLite database created from the app schema.
- ZIP archive integrity check passed.
- Android APK compilation and actual Storage Access Framework behavior still require GitHub Actions build and phone testing; Gradle/Android SDK are not available in the authoring environment.


## PDF reports & production instructions checkpoint 1.5

- Added Android native print integration through `PrintManager` and `WebView.createPrintDocumentAdapter`; no PDF-generation library was added.
- Added a Project production instruction sheet with customer/order identifiers, job name, quantity, deadline, written specifications, workflow sequence, PIC/estimate fields, selected Paper Layout pattern details, operator/QC notes, and sign-off areas. It deliberately excludes selling price and HPP.
- Added an internal Project report with selling price, HPP total/per-unit, cost-line details, workflow progress, and job description.
- Added a Sales Order report with customer, project list, payments, total paid, and outstanding balance.
- Print actions are available from Project detail, Sales Order detail, and Export & Backup. Android's print dialog can send the document to a printer or save it as PDF.
- Output is formatted for A4 paper. The app prepares printable HTML and delegates PDF rendering to Android's system print framework.
- No database schema changes, permissions, or third-party libraries were added.

## PDF checkpoint 1.5 validation

- JavaScript syntax check passed with Node.js.
- Confirmed print actions are wired from Project detail, Sales Order detail, and Export & Backup.
- Android compilation, Android print preview, Save as PDF, and physical printer output remain to be verified by GitHub Actions and on a device; Gradle/Android SDK are not available in the authoring environment.


## Production time & notifications checkpoint 1.6

- Added native Android local notifications for Project production-stage estimated completion times.
- Saving or changing a future estimate schedules one reminder using AlarmManager; clearing an estimate or completing the stage cancels its pending reminder.
- Uses Android notification channel `Estimasi produksi`; Android 13+ notification permission is requested when the user first saves an estimate, rather than unexpectedly on app launch.
- Pending future reminders are rebuilt when the app starts and after device reboot. No Firebase, network service, or third-party library is used.
- Project workflow shows a `Terlambat` marker when an unfinished stage's estimate has passed. The marker remains until the stage is completed.
- Reminder is one-time per scheduled estimate. It is not an exact-alarm guarantee: Android battery optimization/Doze and notification settings may delay or suppress delivery.
- No database schema changes; existing `estimated_at`, `status`, and `project_stages` fields are reused.

## Checkpoint 1.6 validation

- JavaScript syntax check passed with Node.js.
- AndroidManifest.xml parsed successfully.
- ZIP archive integrity check passed.
- Android APK compilation and actual notification permission/alarm/reboot behavior still require GitHub Actions and on-device testing; this authoring environment has no Android SDK/Gradle.


## Operational dashboard checkpoint 1.7

- Reworked Home into an operational dashboard focused on production priorities.
- Shows active Project count, overdue unfinished production stages, stages estimated within the next 24 hours, and active Project deadlines within the next 3 days (including already-overdue deadlines).
- Priority cards open the related Project detail so the operator can continue the workflow.
- Shows recent Projects and quick actions for adding a Customer or Sales Order.
- Dashboard reads Project/stage data from the existing bridge and SQLite schema; no database migration or third-party library was added.
- Project deadline and stage estimate values are compared as Unix-millisecond timestamps, consistent with the existing native bridge.

## Checkpoint 1.7 validation

- JavaScript syntax check passed with Node.js.
- ZIP archive integrity check passed.
- Dashboard uses existing native bridge methods (`listProjects`, `listProjectStages`, and `listSalesOrders`).
- Android APK compilation and on-device UI/performance testing remain pending GitHub Actions and phone validation. The current dashboard loads up to 500 recent Projects; larger installations may need paginated aggregation in a future iteration.


## Operator workflow UX checkpoint 1.8

- Reworked Project workflow into an operator-oriented sequence with a prominent current task / next action panel.
- The focus panel shows the active stage or the next eligible stage, its PIC, estimated completion time, overdue indication, and operator instruction.
- Only the active stage can be completed; only the first not-started stage whose predecessors are all completed is offered as the next stage. Later stages are visibly muted as waiting. Database sequential-start triggers remain the final enforcement layer.
- Added per-stage operator instructions/notes: can be entered when creating a stage and edited later. Reuses the existing `project_stages.notes` column through a small native bridge method; no schema migration.
- Stage cards retain PIC/estimate editing and provide clearer current/next/waiting visual states. Cancelled Projects show a stopped-production message.
- No third-party libraries or database schema changes. Version bumped to 1.8.0 (versionCode 10).

## Checkpoint 1.8 validation

- JavaScript syntax check passed with Node.js.
- ZIP archive integrity check passed.
- Verified stage list already exposes `notes`; the new bridge method updates that existing field and timestamp.
- Android APK compilation and on-device workflow interaction testing remain pending GitHub Actions and phone validation; this authoring environment has no Android SDK/Gradle.
