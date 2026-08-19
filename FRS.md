# Functional Requirements Specification (FRS)

## Nevo &ndash; Cross-Platform Note Management Application

**Course:** 503107 &ndash; Cross-Platform Mobile App Development, Semester I/2026-2027, Ton Duc Thang University
**Document type:** Functional Requirements Specification (FRS)
**Prepared by:** Long (Dranov)
**Status:** Draft v1.1
**Date:** 2026-08-19

## Revision History

| Version | Date | Author | Description |
|---|---|---|---|
| 1.0 | 2026-08-19 | Long (Dranov) | Initial draft derived from the Final Project brief and rubric |
| 1.1 | 2026-08-19 | Long (Dranov) | Renamed project from NoteSphere to Nevo |

# 1. Introduction

## 1.1 Purpose

This Functional Requirements Specification (FRS) defines the functional and non-functional behavior required of **Nevo**, the cross-platform note management application assigned as the Final Project for course 503107. It translates the official project brief and 32-item grading rubric into unambiguous, testable requirements that the development team can implement, the instructor can verify, and QA can validate before submission. Every requirement below is traceable to a rubric item (see Section 6, Traceability Matrix) so the team can self-assess progress against Rubric.xlsx at any time.

## 1.2 Scope

Nevo lets an authenticated user create, organize, protect, share, and intelligently interact with personal notes. The product consists of a single Flutter/Dart codebase producing two mandatory release artifacts: a publicly deployed Flutter Web build and a submitted native build (Android APK, Windows package, or macOS .app). Both releases share identical core functionality, differing only in genuinely platform-specific integration code. A backend (managed BaaS such as Firebase/Supabase/Appwrite, or a custom service) provides authentication, data persistence, authorization enforcement, real-time collaboration, and a secure proxy for LLM-powered AI features.

Out of scope: native replacement clients written in a different framework per platform, offline-only operation with no eventual synchronization, and any feature that contradicts the mandatory (red-highlighted) requirements of the official brief.

## 1.3 Definitions, Acronyms, and Abbreviations

| Term | Definition |
|---|---|
| FRS | Functional Requirements Specification |
| BaaS | Backend-as-a-Service (e.g. Firebase, Supabase, Appwrite) |
| LLM | Large Language Model, used for AI Summary and AI Q&A features |
| OTP | One-Time Password, used for password-reset verification |
| RBAC | Role/permission-based access control (owner, editor, viewer of a note) |
| CRUD | Create, Read, Update, Delete |
| RTO | Read-only permission on a shared note |
| Rubric ID | The numeric ID (1&ndash;32) assigned to each gradable criterion in the official rubric |

## 1.4 References

503107 &ndash; Cross-Platform Mobile App Development, Final Project Brief & Rubric, Semester I/2026-2027, Ton Duc Thang University (source PDF, 19 pages, sections I&ndash;VII).

# 2. Overall Description

## 2.1 Product Perspective

Nevo is a new, self-contained product (not an extension of an existing system). It is a client&ndash;server system: a Flutter client (Web + one native target) communicates with a backend that owns authentication, data storage, authorization rules, real-time sync, and any operation requiring a secret credential (including all LLM calls).

## 2.2 User Classes and Characteristics

| User class | Description |
|---|---|
| Unauthenticated visitor | Can reach only public auth screens: login, registration, account activation, forgotten-password, password-reset. |
| Registered user (unverified) | Logged in, full functional access, but a persistent banner nags for email activation. |
| Registered user (verified) | Full owner-level access to their own notes, labels, sharing, AI features, and preferences. |
| Recipient / collaborator | A registered user who has been granted read-only or edit access to another user's note. |
| Instructor / grader | Uses grading credentials supplied in Rubric.xlsx to verify every criterion on the deployed Web build and the submitted native artifact. |

## 2.3 Operating Environment

- Client: Flutter (single project, single Dart codebase) targeting Web (Chrome/Edge/Firefox, responsive from compact mobile widths through desktop windows) and one native platform (Android APK, or Windows package/ZIP, or macOS .app bundle).
- Backend: Firebase, Supabase, Appwrite, another BaaS, or a custom backend (any stack), deployed and reachable over HTTPS for the entire grading period.
- LLM provider: any provider capable of summarization and retrieval-augmented question answering, invoked only from the backend/serverless layer, never from the client with an embedded key.

## 2.4 Design and Implementation Constraints

- Single Flutter project/codebase for both released platforms; no per-platform replacement app.
- No secrets (API keys, DB passwords, signing keys) in Dart source, the client bundle, or Git history; sensitive/LLM calls go through a secured backend.
- Authorization (note ownership, share permissions, password-protected notes) must be enforced server-side / by data-service security rules &ndash; never solely by hiding UI controls.
- Passwords (account and per-note) must never be stored in plaintext.
- Repository must show at least 4 consecutive active calendar weeks with &ge;2 meaningful commits per member per week.
- Low-code/no-code app generators are prohibited; the team must author the implementation.

## 2.5 Assumptions and Dependencies

- The chosen BaaS or custom backend can be provisioned within course infrastructure limits and remain online for the full grading period.
- An LLM API (or equivalent) is available and affordable for the team for Summary and Q&A features.
- Instructor grading accounts, if required, will be preloaded with representative data as described in Readme.txt.

# 3. External Interface Requirements

## 3.1 User Interfaces

Screens required: Login, Registration, Account Activation notice, Forgotten Password, Password Reset (link or OTP), Home (notes grid/list), Note Editor (create/edit, single reusable screen), Note Detail/Viewer, Label management, Search results, Shared-with-me section, Sharing/permission management dialog, Note password (set/change/unlock) dialog, AI Summary panel, AI Q&A panel, User Profile, Change Password, User Preferences. All screens must supply loading, empty, success, and error states, and must adapt to compact mobile, tablet, desktop, and web breakpoints, including portrait/landscape where applicable.

## 3.2 Software Interfaces

- Authentication interface (managed BaaS auth or custom): registration, login, logout, session/token refresh, password reset, email verification link/OTP delivery.
- Data/storage interface: CRUD for notes, labels, attachments, shares, user profile, preferences; real-time subscription channel for collaborative edits.
- File storage interface: upload/download of avatars and note attachments (image, video, general file), with type/size validation.
- LLM interface (server-side only): summarization endpoint; retrieval-augmented question-answering endpoint scoped to notes the requesting user is authorized to read.
- Email interface: activation email, password-reset email/OTP, share-notification email (recommended, see Section 5.9).

## 3.3 Communications Interfaces

All client&ndash;backend traffic over HTTPS/TLS. Real-time updates for shared, editable notes over the backend's real-time channel (e.g. WebSocket-based listeners, Firestore/Realtime DB streams, or Supabase Realtime).

# 4. Functional Requirements

Requirements are grouped by rubric section. Each requirement carries an ID of the form FR-&lt;RubricID&gt; for traceability. Priority: M = Mandatory (red-highlighted in the brief), S = Standard.

## 4.1 Account Management (Rubric IDs 1&ndash;8, weight 2.0 pt)

| ID | Requirement | Pri. |
|---|---|---|
| FR-1 | Users register with email, display name, and password entered twice. Managed-auth password handling is delegated to the provider; a custom backend must hash passwords with a strong algorithm and never store plaintext. On success the user is auto-logged-in and an activation email is sent. | M |
| FR-2 | Before activation, all functionality remains available, but a persistent, prominent banner indicates the account is unverified; activation link/mechanism removes the banner on success, consistently on Web and native. | M |
| FR-3 | Unauthenticated users attempting a protected screen are redirected to Login; auth-related screens (register, activation, forgot/reset password) remain reachable without login; successful login routes to the personalized Home showing the user's notes; Logout ends the session. | M |
| FR-4 | Password reset via email: user follows a link or enters an emailed OTP, is routed to a new-password screen, and must then log in manually. Works on both Web and native. | M |
| FR-5 | Profile screen displays display name and avatar (with a sensible default when none is set). | M |
| FR-6 | Profile edit allows updating display name and avatar; uploaded avatar files are validated (type/size); the profile stays consistent across submitted platforms. | M |
| FR-7 | Change Password requires the current password plus the new password twice; on success the app manages the authentication session securely (e.g. re-auth or safe session refresh). | M |
| FR-8 | User Preferences (font size, note color, light/dark theme, and the last-selected note view) are persisted per user and re-applied whenever the app is reopened on any submitted platform. | M |

## 4.2 Simple Note Management (Rubric IDs 9&ndash;22, weight 3.5 pt)

| ID | Requirement | Pri. |
|---|---|---|
| FR-9 / FR-10 | Notes display in a grid view by default; the user can switch to list view; the chosen layout is a persisted preference and renders correctly from compact mobile to desktop/web. | M |
| FR-11 / FR-12 | A single, reusable editor screen handles both note creation and editing. Only title and content are mandatory. Content auto-saves without a Save button, with a short debounce, and communicates saving / saved / failed states; the latest valid changes survive pause, background, close, and reopen within platform capability. | M |
| FR-13 | Deleting a note always requires an explicit confirmation dialog. | M |
| FR-14 | Auto-save and app-lifecycle handling (see FR-11/12) is verified across pause/resume/kill scenarios on each submitted platform. | M |
| FR-15 / FR-16 | A note may include one or more image/video/general-file attachments, added via gallery, camera, browser file picker, or native file picker as appropriate; the app validates file type/size, handles cancellation and denied permissions gracefully, and restricts attachment access to users authorized on the parent note. | M |
| FR-17 | Notes sort by creation time or last-modified time (newest first) by default; users may pin notes; pinned notes always precede unpinned notes, and multi-pin ordering is deterministic (by pin time). | M |
| FR-18 | Shared, pinned, and password-protected notes show recognizable status indicators in both grid and list views, on Web and native; multiple indicators can co-exist on one note. | M |
| FR-19 | Live search filters notes by keyword in title or content as the user types, using a short debounce (~300 ms), no Search button required. | M |
| FR-20 / FR-21 / FR-22 | Users can list, add, rename, and delete labels; a note may carry zero or more labels; deleting a label never deletes or damages associated notes; renaming a label updates its displayed name everywhere; users can filter notes by selected labels. | M |

## 4.3 Advanced Note Management (Rubric IDs 23&ndash;28, weight 2.5 pt)

| ID | Requirement | Pri. |
|---|---|---|
| FR-23 / FR-24 | Any note can be password-protected independently, with its own password. Once protected, viewing, editing, deleting, sharing, summarizing, or otherwise accessing the note requires the correct password. Owners can change the note password (current password verified first) or disable protection (current password confirmed first). Note passwords are never stored in plaintext; enforcement happens server-side, not only by hiding UI controls. | M |
| FR-25 | Owners share notes with one or more registered accounts by email, assigning read-only or edit permission per recipient, and can change or revoke access at any time; all enforcement is via backend/data-service authorization, so a read-only user cannot mutate the note by calling the API/data-service directly. | M |
| FR-26 | Recipients see a dedicated "Shared with me" section showing, per note, permission level, sharer identity, and share timestamp. Edit-permission notes support real-time collaboration: authorized users see updates live, without manual refresh; unauthorized or read-only users cannot submit changes. | M |
| FR-27 | AI Summary: generate a concise LLM summary of a single note capturing main ideas without altering meaning; regenerable on demand; never silently overwrites the original note content. | M |
| FR-28 | AI Question Answering: natural-language questions answered from only the notes the authenticated user may access, grounded in that content (not keyword search alone), with visible references back to source notes openable on Web and native; unlocked protected notes only; if insufficient relevant content exists, the system says so rather than fabricating an answer. | M |

## 4.4 Other / Cross-Cutting Requirements (Rubric IDs 29&ndash;32, weight 2.0 pt)

| ID | Requirement | Pri. |
|---|---|---|
| FR-29 | UI/UX is visually consistent, intuitive, accessible, and adaptive across compact mobile, tablet, desktop, and web (including orientation where relevant); every screen provides loading, empty, success, and error states; touch, mouse, and keyboard input are all supported appropriately. | M |
| FR-30 | Web and native builds originate from one Flutter project with a documented, consistently-applied architecture separating presentation, state, business logic, and data access; minimum automated tests: 3 unit, 3 widget, 1 integration test covering a critical user flow, all exercising real behavior. | M |
| FR-31 | Previously loaded notes remain viewable offline; notes created/edited offline are stored locally and synced when connectivity returns; UI communicates offline / pending-sync / sync-failure states; cached data never leaks across accounts; a documented conflict-handling strategy governs concurrent edits. | M |
| FR-32 | The Flutter Web build and all required backend services are deployed publicly and remain operational through grading; one working native release artifact (Android APK, Windows package/ZIP, or macOS .app) is submitted; both releases share the same Flutter project and provide the same mandatory core functionality. | M |

# 5. Non-Functional Requirements

## 5.1 Security

Account and note passwords hashed, never logged or transmitted in plaintext beyond TLS; authorization enforced server-side for every protected resource (notes, attachments, shares, AI answers); secrets kept out of client code and Git history; sensitive/LLM operations proxied through a secured backend function.

## 5.2 Performance

Live search responds within ~300 ms of the last keystroke; auto-save debounce short enough to feel responsive while avoiding excessive writes; real-time collaboration updates propagate without manual refresh.

## 5.3 Reliability & Availability

Web deployment and backend services remain reachable for the entire grading window; offline mode preserves previously loaded data and queues pending writes safely; sync retries safely on reconnect.

## 5.4 Usability & Accessibility

Consistent navigation and interaction patterns; meaningful empty/loading/error feedback; adaptive layouts for mobile, tablet, desktop, web; accessible touch, mouse, and keyboard interaction.

## 5.5 Maintainability

Single codebase with documented architecture and state-management approach (Readme.txt); automated tests (3 unit / 3 widget / 1 integration minimum) exercising real behavior; clean, analyzer-warning-free, consistently formatted code.

## 5.6 Portability

Core Dart code shared across Web and the chosen native target; platform-specific code isolated behind clear abstractions; packages compatible with both submitted platforms or given a documented fallback.

## 5.7 Data Integrity & Multi-Tenancy

Local/offline cache isolated per account; no cross-account data leakage; conflict-handling strategy documented and applied consistently.

## 5.8 Compliance with Course Rules

No low-code/no-code generation of the core app; no plaintext secrets in source or Git history; teamwork evidenced by &ge;4 consecutive active weeks with &ge;2 meaningful commits per member per week, verifiable via GitHub Insights.

## 5.9 Recommended (non-mandatory) Enhancements

Email notification when a note is shared; note version history; export note to PDF/Markdown; keyboard shortcuts on desktop/web. These may be added only if they never replace, weaken, or contradict a mandatory (red) requirement.

# 6. Requirements Traceability Matrix (Rubric Cross-Reference)

| Rubric Section | Rubric IDs | Weight | FRS Section |
|---|---|---|---|
| Account management | 1&ndash;8 | 2.0 pt | 4.1 |
| Simple note management | 9&ndash;22 | 3.5 pt | 4.2 |
| Advanced note management | 23&ndash;28 | 2.5 pt | 4.3 |
| Other requirements (UI/UX) | 29 | 0.5 pt | 4.4, 5.4 |
| Flutter architecture, state mgmt, testing | 30 | 0.5 pt | 4.4, 5.5 |
| Offline persistence & sync | 31 | 0.5 pt | 4.4, 5.3, 5.7 |
| Cross-platform build & deployment | 32 | 0.5 pt | 4.4 |
| Teamwork & GitHub contribution | n/a (deduction rule) | -0.5 pt if unmet | 5.8 |

**Total rubric points:** 10.0 (before deductions).

# 7. Acceptance Criteria Summary

A requirement is considered fully satisfied only when it works as specified on every submitted platform, handles expected error conditions, enforces authorization securely (never by hiding UI alone), avoids critical vulnerabilities, and is demonstrated live in demo.mp4. A criterion not shown in the demo video is treated as not implemented regardless of the team's self-assessment in Rubric.xlsx.
