# Issue map

Every planning ID from `ISSUE_PLAN.md` and the GitHub issue that now carries it.
The **GitHub issues are the live record** — this file maps IDs to them and is not updated as issues change.

Created 2026-08-14 in `jmanico/afterimpact`. 193 issues, 192 parent links.

| Planning ID | Issue | Title |
| --- | --- | --- |
| `REQ-EPIC-001` | [#12](https://github.com/jmanico/afterimpact/issues/12) | Deliver AfterImpact v1 |
| `REQ-PLATFORM-000` | [#13](https://github.com/jmanico/afterimpact/issues/13) | Platform, deployment, and supply chain |
| `REQ-PLATFORM-010` | [#14](https://github.com/jmanico/afterimpact/issues/14) | Establish the Terraform baseline and private network boundary |
| `REQ-PLATFORM-020` | [#15](https://github.com/jmanico/afterimpact/issues/15) | Provision managed PostgreSQL with KMS encryption and PITR |
| `REQ-PLATFORM-030` | [#16](https://github.com/jmanico/afterimpact/issues/16) | Provision the private S3 file store with SSE-KMS and lifecycle expiry |
| `REQ-PLATFORM-040` | [#17](https://github.com/jmanico/afterimpact/issues/17) | Separate KMS key custody from the data plane |
| `REQ-PLATFORM-050` | [#18](https://github.com/jmanico/afterimpact/issues/18) | Scope least-privilege IAM roles per component |
| `REQ-PLATFORM-060` | [#19](https://github.com/jmanico/afterimpact/issues/19) | Inject runtime secrets and keep them out of code and logs |
| `REQ-PLATFORM-070` | [#20](https://github.com/jmanico/afterimpact/issues/20) | Deploy the CloudFront and WAF edge with a pinned TLS policy |
| `REQ-PLATFORM-080` | [#21](https://github.com/jmanico/afterimpact/issues/21) | Build the CI pipeline with OIDC federation and scanners |
| `REQ-PLATFORM-090` | [#22](https://github.com/jmanico/afterimpact/issues/22) | Enforce the dependency admission and patching policy |
| `REQ-PLATFORM-100` | [#23](https://github.com/jmanico/afterimpact/issues/23) | Gate releases on the ASVS 5.0.0 Level 2 self-assessment |
| `REQ-PLATFORM-110` | [#24](https://github.com/jmanico/afterimpact/issues/24) | Configure the SES sending domain with SPF, DKIM, and DMARC |
| `REQ-API-000` | [#25](https://github.com/jmanico/afterimpact/issues/25) | REST API foundation and boundary controls |
| `REQ-API-010` | [#26](https://github.com/jmanico/afterimpact/issues/26) | Stand up the Quarkus API with /v1 URL versioning |
| `REQ-API-020` | [#27](https://github.com/jmanico/afterimpact/issues/27) | Enforce owner-only authorization in a single centralized layer |
| `REQ-API-030` | [#28](https://github.com/jmanico/afterimpact/issues/28) | Serve all functionality over TLS 1.2 or higher only |
| `REQ-API-040` | [#29](https://github.com/jmanico/afterimpact/issues/29) | Validate HTTP method and declared content type |
| `REQ-API-050` | [#30](https://github.com/jmanico/afterimpact/issues/30) | Emit the browser security header set including a strict CSP |
| `REQ-API-060` | [#31](https://github.com/jmanico/afterimpact/issues/31) | Keep CORS disabled and mark every response no-store |
| `REQ-API-070` | [#32](https://github.com/jmanico/afterimpact/issues/32) | Enforce request-rate and resource limits at the recorded values |
| `REQ-API-080` | [#33](https://github.com/jmanico/afterimpact/issues/33) | Bind request bodies to explicit models, rejecting unknown fields |
| `REQ-API-090` | [#34](https://github.com/jmanico/afterimpact/issues/34) | Provide the server-side validation framework |
| `REQ-API-100` | [#35](https://github.com/jmanico/afterimpact/issues/35) | Use parameterized statements and allowlisted sort identifiers |
| `REQ-API-110` | [#36](https://github.com/jmanico/afterimpact/issues/36) | Centralize exception handling and sanitize client errors |
| `REQ-API-120` | [#37](https://github.com/jmanico/afterimpact/issues/37) | Meet the interactive latency budget under expected load |
| `REQ-DATA-000` | [#38](https://github.com/jmanico/afterimpact/issues/38) | Data persistence and protection |
| `REQ-DATA-010` | [#39](https://github.com/jmanico/afterimpact/issues/39) | Define the third-normal-form schema for every entity |
| `REQ-DATA-020` | [#40](https://github.com/jmanico/afterimpact/issues/40) | Model soft-delete state for the 30-day undo windows |
| `REQ-DATA-030` | [#41](https://github.com/jmanico/afterimpact/issues/41) | Persist document version lineage and journal entry versions |
| `REQ-DATA-040` | [#42](https://github.com/jmanico/afterimpact/issues/42) | Snapshot contact names on referencing records |
| `REQ-DATA-050` | [#43](https://github.com/jmanico/afterimpact/issues/43) | Encrypt all stored personal data at rest |
| `REQ-DATA-060` | [#44](https://github.com/jmanico/afterimpact/issues/44) | Meet the backup RPO and RTO with tested restores |
| `REQ-DATA-070` | [#45](https://github.com/jmanico/afterimpact/issues/45) | Reconcile security and privacy state after a restore |
| `REQ-DATA-080` | [#46](https://github.com/jmanico/afterimpact/issues/46) | Limit collection to data the functional requirements need |
| `REQ-DATA-090` | [#47](https://github.com/jmanico/afterimpact/issues/47) | Keep personal data unshared and screens free of trackers |
| `REQ-DATA-100` | [#48](https://github.com/jmanico/afterimpact/issues/48) | Operate the breach-notification runbook |
| `REQ-AUTH-000` | [#49](https://github.com/jmanico/afterimpact/issues/49) | Accounts, authentication, and sessions |
| `REQ-AUTH-010` | [#50](https://github.com/jmanico/afterimpact/issues/50) | Terminate identity-provider ceremonies server-side |
| `REQ-AUTH-020` | [#51](https://github.com/jmanico/afterimpact/issues/51) | Register an account and verify its email before sign-in |
| `REQ-AUTH-030` | [#52](https://github.com/jmanico/afterimpact/issues/52) | Show the privacy policy before registration and record acceptance |
| `REQ-AUTH-040` | [#53](https://github.com/jmanico/afterimpact/issues/53) | Bootstrap the first passkey from a restricted email-code session |
| `REQ-AUTH-050` | [#54](https://github.com/jmanico/afterimpact/issues/54) | Register a passkey and constrain stored credential material |
| `REQ-AUTH-060` | [#55](https://github.com/jmanico/afterimpact/issues/55) | Authenticate with a passkey and issue a normal session |
| `REQ-AUTH-070` | [#56](https://github.com/jmanico/afterimpact/issues/56) | Issue and transport opaque server-side session identifiers |
| `REQ-AUTH-080` | [#57](https://github.com/jmanico/afterimpact/issues/57) | Invalidate a session immediately on sign-out |
| `REQ-AUTH-090` | [#58](https://github.com/jmanico/afterimpact/issues/58) | Enforce absolute and idle session lifetimes server-side |
| `REQ-AUTH-100` | [#59](https://github.com/jmanico/afterimpact/issues/59) | Throttle failed authentication behind source-based limits |
| `REQ-AUTH-110` | [#60](https://github.com/jmanico/afterimpact/issues/60) | Respond identically whether or not an account exists |
| `REQ-AUTH-120` | [#61](https://github.com/jmanico/afterimpact/issues/61) | Recover from loss of every passkey through the verified address |
| `REQ-AUTH-130` | [#62](https://github.com/jmanico/afterimpact/issues/62) | List and revoke credentials with a no-lockout guard |
| `REQ-AUTH-140` | [#63](https://github.com/jmanico/afterimpact/issues/63) | Prompt for a second passkey while only one is registered |
| `REQ-AUTH-150` | [#64](https://github.com/jmanico/afterimpact/issues/64) | Change the account email address with re-verification |
| `REQ-AUTH-160` | [#65](https://github.com/jmanico/afterimpact/issues/65) | Require a fresh passkey ceremony before high-impact operations |
| `REQ-AUTH-170` | [#66](https://github.com/jmanico/afterimpact/issues/66) | Send out-of-band notices for security-significant events |
| `REQ-AUTH-180` | [#67](https://github.com/jmanico/afterimpact/issues/67) | Delete an account behind re-authentication and a cancellation window |
| `REQ-AUTH-190` | [#68](https://github.com/jmanico/afterimpact/issues/68) | Complete the account purge within the stated bounds |
| `REQ-AUTH-200` | [#69](https://github.com/jmanico/afterimpact/issues/69) | Purge inactive accounts after advance warnings |
| `REQ-AUTH-210` | [#70](https://github.com/jmanico/afterimpact/issues/70) | Store and accept no user-chosen shared secret |
| `REQ-LOG-000` | [#71](https://github.com/jmanico/afterimpact/issues/71) | Security event logging and activity visibility |
| `REQ-LOG-010` | [#72](https://github.com/jmanico/afterimpact/issues/72) | Record the security event log across every listed event type |
| `REQ-LOG-020` | [#73](https://github.com/jmanico/afterimpact/issues/73) | Keep credentials, tokens, and health data out of logs |
| `REQ-LOG-030` | [#74](https://github.com/jmanico/afterimpact/issues/74) | Make the security event log append-only and tamper-evident |
| `REQ-LOG-040` | [#75](https://github.com/jmanico/afterimpact/issues/75) | Pseudonymize surviving log entries on account deletion |
| `REQ-LOG-050` | [#76](https://github.com/jmanico/afterimpact/issues/76) | Show the user their own recent security activity in-app |
| `REQ-NOTIFY-000` | [#77](https://github.com/jmanico/afterimpact/issues/77) | Notification feed and outbound email |
| `REQ-NOTIFY-010` | [#78](https://github.com/jmanico/afterimpact/issues/78) | Serve the in-app notification feed |
| `REQ-NOTIFY-020` | [#79](https://github.com/jmanico/afterimpact/issues/79) | Tie notification lifetime to its source record, case, and account |
| `REQ-NOTIFY-030` | [#80](https://github.com/jmanico/afterimpact/issues/80) | Send outbound email carrying only the minimum personal data |
| `REQ-NOTIFY-040` | [#81](https://github.com/jmanico/afterimpact/issues/81) | Surface persistent email-delivery failure as an in-app notice |
| `REQ-NOTIFY-050` | [#82](https://github.com/jmanico/afterimpact/issues/82) | Issue and validate one-time email codes under the provider rules |
| `REQ-CASE-000` | [#83](https://github.com/jmanico/afterimpact/issues/83) | Accident cases |
| `REQ-CASE-010` | [#84](https://github.com/jmanico/afterimpact/issues/84) | Create a case with a validated accident date and derived title |
| `REQ-CASE-020` | [#85](https://github.com/jmanico/afterimpact/issues/85) | List cases open-first, newest accident date first |
| `REQ-CASE-030` | [#86](https://github.com/jmanico/afterimpact/issues/86) | Edit any case field under the creation validation rules |
| `REQ-CASE-040` | [#87](https://github.com/jmanico/afterimpact/issues/87) | Compute the per-case overview |
| `REQ-CASE-050` | [#88](https://github.com/jmanico/afterimpact/issues/88) | Archive a case and reverse the archive without data loss |
| `REQ-CASE-060` | [#89](https://github.com/jmanico/afterimpact/issues/89) | Delete a case behind step-up and explicit confirmation |
| `REQ-CASE-070` | [#90](https://github.com/jmanico/afterimpact/issues/90) | Purge a deleted case permanently after the undo window |
| `REQ-DOC-000` | [#91](https://github.com/jmanico/afterimpact/issues/91) | Documents and paperwork |
| `REQ-DOC-010` | [#92](https://github.com/jmanico/afterimpact/issues/92) | Accept uploads only after server-side content inspection |
| `REQ-DOC-020` | [#93](https://github.com/jmanico/afterimpact/issues/93) | Enforce per-file size limits and the per-user storage quota |
| `REQ-DOC-030` | [#94](https://github.com/jmanico/afterimpact/issues/94) | Store originals under server-generated names with a SHA-256 hash |
| `REQ-DOC-040` | [#95](https://github.com/jmanico/afterimpact/issues/95) | Record document title, category, and optional metadata |
| `REQ-DOC-050` | [#96](https://github.com/jmanico/afterimpact/issues/96) | List a case's documents with filtering and sorting |
| `REQ-DOC-060` | [#97](https://github.com/jmanico/afterimpact/issues/97) | Search document titles, notes, and tags within a case |
| `REQ-DOC-070` | [#98](https://github.com/jmanico/afterimpact/issues/98) | Stream byte-identical downloads through authorized endpoints |
| `REQ-DOC-080` | [#99](https://github.com/jmanico/afterimpact/issues/99) | View safe, non-active types inline in a sandboxed viewer |
| `REQ-DOC-090` | [#100](https://github.com/jmanico/afterimpact/issues/100) | Edit document metadata without altering the stored file |
| `REQ-DOC-100` | [#101](https://github.com/jmanico/afterimpact/issues/101) | Replace a document file while retaining prior versions |
| `REQ-DOC-110` | [#102](https://github.com/jmanico/afterimpact/issues/102) | Trash, restore, and purge documents across the 30-day window |
| `REQ-DOC-120` | [#103](https://github.com/jmanico/afterimpact/issues/103) | Scan uploads for malware in the isolated worker and quarantine |
| `REQ-DOC-130` | [#104](https://github.com/jmanico/afterimpact/issues/104) | Meet the upload throughput floor at every accepted file size |
| `REQ-HEALTH-000` | [#105](https://github.com/jmanico/afterimpact/issues/105) | Health issues and appointments |
| `REQ-HEALTH-010` | [#106](https://github.com/jmanico/afterimpact/issues/106) | Record a health issue |
| `REQ-HEALTH-020` | [#107](https://github.com/jmanico/afterimpact/issues/107) | Track health issue status with a timestamped history |
| `REQ-HEALTH-030` | [#108](https://github.com/jmanico/afterimpact/issues/108) | Add timestamped progress updates in chronological order |
| `REQ-HEALTH-040` | [#109](https://github.com/jmanico/afterimpact/issues/109) | Record treatments and medications as recordkeeping only |
| `REQ-HEALTH-050` | [#110](https://github.com/jmanico/afterimpact/issues/110) | Link a health issue to providers, documents, expenses, appointments |
| `REQ-HEALTH-060` | [#111](https://github.com/jmanico/afterimpact/issues/111) | Create appointments and list upcoming ones per case |
| `REQ-HEALTH-070` | [#112](https://github.com/jmanico/afterimpact/issues/112) | Create the default 24-hour reminder on a future appointment |
| `REQ-HEALTH-080` | [#113](https://github.com/jmanico/afterimpact/issues/113) | Record appointment outcome notes after it occurs |
| `REQ-HEALTH-090` | [#114](https://github.com/jmanico/afterimpact/issues/114) | Keep health features to recordkeeping with no medical advice |
| `REQ-HEALTH-100` | [#115](https://github.com/jmanico/afterimpact/issues/115) | Edit and delete health records |
| `REQ-TASK-000` | [#116](https://github.com/jmanico/afterimpact/issues/116) | Recovery tasks, deadlines, and reminders |
| `REQ-TASK-010` | [#117](https://github.com/jmanico/afterimpact/issues/117) | Create a task |
| `REQ-TASK-020` | [#118](https://github.com/jmanico/afterimpact/issues/118) | Offer the starter recovery checklist at case creation |
| `REQ-TASK-030` | [#119](https://github.com/jmanico/afterimpact/issues/119) | Mark a task complete or not complete with a timestamp |
| `REQ-TASK-040` | [#120](https://github.com/jmanico/afterimpact/issues/120) | Edit and delete tasks |
| `REQ-TASK-050` | [#121](https://github.com/jmanico/afterimpact/issues/121) | Flag overdue tasks visibly and in the overview count |
| `REQ-TASK-060` | [#122](https://github.com/jmanico/afterimpact/issues/122) | Set one or more reminders on a task or appointment |
| `REQ-TASK-070` | [#123](https://github.com/jmanico/afterimpact/issues/123) | Deliver reminders within five minutes of fire time |
| `REQ-TASK-080` | [#124](https://github.com/jmanico/afterimpact/issues/124) | Toggle email reminders account-wide |
| `REQ-TASK-090` | [#125](https://github.com/jmanico/afterimpact/issues/125) | Escalate hard-deadline reminders with a visible countdown |
| `REQ-TASK-100` | [#126](https://github.com/jmanico/afterimpact/issues/126) | Attach the guidance-not-advice notice to guidance content |
| `REQ-CONTACT-000` | [#127](https://github.com/jmanico/afterimpact/issues/127) | Contacts |
| `REQ-CONTACT-010` | [#128](https://github.com/jmanico/afterimpact/issues/128) | Add a case-scoped contact |
| `REQ-CONTACT-020` | [#129](https://github.com/jmanico/afterimpact/issues/129) | List a case's contacts filterable by role |
| `REQ-CONTACT-030` | [#130](https://github.com/jmanico/afterimpact/issues/130) | Edit a contact |
| `REQ-CONTACT-040` | [#131](https://github.com/jmanico/afterimpact/issues/131) | Delete a contact while retaining its name on referencing records |
| `REQ-EXPENSE-000` | [#132](https://github.com/jmanico/afterimpact/issues/132) | Expenses |
| `REQ-EXPENSE-010` | [#133](https://github.com/jmanico/afterimpact/issues/133) | Record an expense with a validated amount and date |
| `REQ-EXPENSE-020` | [#134](https://github.com/jmanico/afterimpact/issues/134) | Edit expenses and delete them after confirmation |
| `REQ-EXPENSE-030` | [#135](https://github.com/jmanico/afterimpact/issues/135) | Compute per-case expense totals |
| `REQ-EXPENSE-040` | [#136](https://github.com/jmanico/afterimpact/issues/136) | Record partial reimbursements and show the remaining balance |
| `REQ-EXPENSE-050` | [#137](https://github.com/jmanico/afterimpact/issues/137) | Fix one currency per case and perform no conversion |
| `REQ-EXPENSE-060` | [#138](https://github.com/jmanico/afterimpact/issues/138) | Export a case's expense list as CSV with inert output |
| `REQ-TIMELINE-000` | [#139](https://github.com/jmanico/afterimpact/issues/139) | Timeline and journal |
| `REQ-TIMELINE-010` | [#140](https://github.com/jmanico/afterimpact/issues/140) | Write timeline events in the triggering action's transaction |
| `REQ-TIMELINE-020` | [#141](https://github.com/jmanico/afterimpact/issues/141) | Author journal entries with a backdatable entry date |
| `REQ-TIMELINE-030` | [#142](https://github.com/jmanico/afterimpact/issues/142) | Edit journal entries and retain prior versions |
| `REQ-TIMELINE-040` | [#143](https://github.com/jmanico/afterimpact/issues/143) | Delete a journal entry leaving a permanent timeline trace |
| `REQ-TIMELINE-050` | [#144](https://github.com/jmanico/afterimpact/issues/144) | Show timeline events and journal entries together with filters |
| `REQ-TIMELINE-060` | [#145](https://github.com/jmanico/afterimpact/issues/145) | Keep system-generated timeline events uneditable |
| `REQ-EXPORT-000` | [#146](https://github.com/jmanico/afterimpact/issues/146) | Export and data management |
| `REQ-EXPORT-010` | [#147](https://github.com/jmanico/afterimpact/issues/147) | Generate the case summary PDF within its time budget |
| `REQ-EXPORT-020` | [#148](https://github.com/jmanico/afterimpact/issues/148) | Generate the full-account archive asynchronously |
| `REQ-EXPORT-030` | [#149](https://github.com/jmanico/afterimpact/issues/149) | Authorize export artifact retrieval |
| `REQ-EXPORT-040` | [#150](https://github.com/jmanico/afterimpact/issues/150) | Purge export artifacts seven days after generation |
| `REQ-EXPORT-050` | [#151](https://github.com/jmanico/afterimpact/issues/151) | State export scope to the user on every export |
| `REQ-EXPORT-060` | [#152](https://github.com/jmanico/afterimpact/issues/152) | Count export artifacts against the storage quota |
| `REQ-DESIGN-000` | [#153](https://github.com/jmanico/afterimpact/issues/153) | Design language and style guide |
| `REQ-DESIGN-010` | [#154](https://github.com/jmanico/afterimpact/issues/154) | Define the semantic color tokens for both themes |
| `REQ-DESIGN-020` | [#155](https://github.com/jmanico/afterimpact/issues/155) | Define the type scale, spacing scale, grid, and breakpoints |
| `REQ-DESIGN-030` | [#156](https://github.com/jmanico/afterimpact/issues/156) | Specify the core component conventions |
| `REQ-DESIGN-040` | [#157](https://github.com/jmanico/afterimpact/issues/157) | Specify focus, form feedback, and destructive-action conventions |
| `REQ-DESIGN-050` | [#158](https://github.com/jmanico/afterimpact/issues/158) | Encode the accessibility rules that meet WCAG 2.2 Level AA |
| `REQ-DESIGN-060` | [#159](https://github.com/jmanico/afterimpact/issues/159) | Specify the on-screen privacy conventions |
| `REQ-DESIGN-070` | [#160](https://github.com/jmanico/afterimpact/issues/160) | Make the brand mark render with the active theme's tokens |
| `REQ-DESIGN-080` | [#161](https://github.com/jmanico/afterimpact/issues/161) | Bring style-guide.html into agreement with DESIGN.md |
| `REQ-WEB-000` | [#162](https://github.com/jmanico/afterimpact/issues/162) | Web client |
| `REQ-WEB-010` | [#163](https://github.com/jmanico/afterimpact/issues/163) | Build the web app shell, routing, and session handling |
| `REQ-WEB-020` | [#164](https://github.com/jmanico/afterimpact/issues/164) | Implement the design system in React |
| `REQ-WEB-030` | [#165](https://github.com/jmanico/afterimpact/issues/165) | Build i18n infrastructure with English and Spanish |
| `REQ-WEB-040` | [#166](https://github.com/jmanico/afterimpact/issues/166) | Build the authentication and recovery screens |
| `REQ-WEB-050` | [#167](https://github.com/jmanico/afterimpact/issues/167) | Build the case screens |
| `REQ-WEB-060` | [#168](https://github.com/jmanico/afterimpact/issues/168) | Build the document library and upload screens |
| `REQ-WEB-070` | [#169](https://github.com/jmanico/afterimpact/issues/169) | Build the in-app PDF and image viewer |
| `REQ-WEB-080` | [#170](https://github.com/jmanico/afterimpact/issues/170) | Build the health screens with the severity chart |
| `REQ-WEB-090` | [#171](https://github.com/jmanico/afterimpact/issues/171) | Build the task, deadline, and reminder screens |
| `REQ-WEB-100` | [#172](https://github.com/jmanico/afterimpact/issues/172) | Build the contact screens |
| `REQ-WEB-110` | [#173](https://github.com/jmanico/afterimpact/issues/173) | Build the expense screens and totals |
| `REQ-WEB-120` | [#174](https://github.com/jmanico/afterimpact/issues/174) | Build the timeline and journal screens |
| `REQ-WEB-130` | [#175](https://github.com/jmanico/afterimpact/issues/175) | Build the export screens with the scope notice |
| `REQ-WEB-140` | [#176](https://github.com/jmanico/afterimpact/issues/176) | Build the in-app notification feed |
| `REQ-WEB-150` | [#177](https://github.com/jmanico/afterimpact/issues/177) | Build the security activity view |
| `REQ-WEB-160` | [#178](https://github.com/jmanico/afterimpact/issues/178) | Implement destructive-action confirmations |
| `REQ-WEB-170` | [#179](https://github.com/jmanico/afterimpact/issues/179) | Present errors in plain language |
| `REQ-WEB-180` | [#180](https://github.com/jmanico/afterimpact/issues/180) | Render user content as text and linkify only safe schemes |
| `REQ-WEB-190` | [#181](https://github.com/jmanico/afterimpact/issues/181) | Meet responsive and WCAG 2.2 AA conformance on web |
| `REQ-MOBILE-000` | [#182](https://github.com/jmanico/afterimpact/issues/182) | Mobile client |
| `REQ-MOBILE-010` | [#183](https://github.com/jmanico/afterimpact/issues/183) | Build the mobile app shell, navigation, and session handling |
| `REQ-MOBILE-020` | [#184](https://github.com/jmanico/afterimpact/issues/184) | Implement the design system in Compose Multiplatform |
| `REQ-MOBILE-030` | [#185](https://github.com/jmanico/afterimpact/issues/185) | Build i18n with English and Spanish |
| `REQ-MOBILE-040` | [#186](https://github.com/jmanico/afterimpact/issues/186) | Build the authentication screens with platform passkeys |
| `REQ-MOBILE-050` | [#187](https://github.com/jmanico/afterimpact/issues/187) | Build the case screens |
| `REQ-MOBILE-060` | [#188](https://github.com/jmanico/afterimpact/issues/188) | Build the document library and upload screens |
| `REQ-MOBILE-070` | [#189](https://github.com/jmanico/afterimpact/issues/189) | Build the in-app PDF and image viewer |
| `REQ-MOBILE-080` | [#190](https://github.com/jmanico/afterimpact/issues/190) | Build the health screens with the severity chart |
| `REQ-MOBILE-090` | [#191](https://github.com/jmanico/afterimpact/issues/191) | Build the task, deadline, and reminder screens |
| `REQ-MOBILE-100` | [#192](https://github.com/jmanico/afterimpact/issues/192) | Build the contact screens |
| `REQ-MOBILE-110` | [#193](https://github.com/jmanico/afterimpact/issues/193) | Build the expense screens and totals |
| `REQ-MOBILE-120` | [#194](https://github.com/jmanico/afterimpact/issues/194) | Build the timeline and journal screens |
| `REQ-MOBILE-130` | [#195](https://github.com/jmanico/afterimpact/issues/195) | Build the export screens with the scope notice |
| `REQ-MOBILE-140` | [#196](https://github.com/jmanico/afterimpact/issues/196) | Build the in-app notification feed |
| `REQ-MOBILE-150` | [#197](https://github.com/jmanico/afterimpact/issues/197) | Build the security activity view |
| `REQ-MOBILE-160` | [#198](https://github.com/jmanico/afterimpact/issues/198) | Implement destructive-action confirmations |
| `REQ-MOBILE-170` | [#199](https://github.com/jmanico/afterimpact/issues/199) | Present errors in plain language |
| `REQ-MOBILE-180` | [#200](https://github.com/jmanico/afterimpact/issues/200) | Render user content as text and linkify only safe schemes |
| `REQ-MOBILE-190` | [#201](https://github.com/jmanico/afterimpact/issues/201) | Keep session credentials only in platform secure storage |
| `REQ-MOBILE-200` | [#202](https://github.com/jmanico/afterimpact/issues/202) | Enforce the mobile platform security posture |
| `REQ-MOBILE-210` | [#203](https://github.com/jmanico/afterimpact/issues/203) | Meet accessibility and responsive conformance on mobile |
| `REQ-MOBILE-220` | [#204](https://github.com/jmanico/afterimpact/issues/204) | Distribute through the App Store and Google Play with signing |
