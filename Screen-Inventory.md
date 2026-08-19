# Page / Screen Inventory

## Nevo &ndash; Cross-Platform Note Management Application

**Course:** 503107 &ndash; Cross-Platform Mobile App Development, Semester I/2026-2027, Ton Duc Thang University
**Document type:** Page/Screen Inventory (derived from the User Flow Diagrams and FRS)
**Prepared by:** Long (Dranov)
**Status:** Draft v2.1 &ndash; Date: 2026-08-19

## Purpose

This document enumerates every page/screen and key overlay (dialog, banner, panel) required to implement the flows in the User Flow Diagrams (Authentication & Onboarding; Note Lifecycle & Collaboration). Each row is a build checklist item: name, route, build status, where it nests in the navigation, who owns it, what it does, who can access it, and which rubric item it satisfies.

## Legend

**Status:** New = not started &middot; In Progress &middot; Needs Redesign &middot; Existing (reused) &middot; Remove.
**Access & Who Uses It:** Public = reachable without login &middot; Auth = requires login &middot; Auth+Owner/Editor = requires login and note-level authorization.
**Owner/Assignee:** left as *Unassigned* below &ndash; the team should fill in Kaka / Chou / Long (Dranov) per screen during sprint planning; nothing here was assumed on anyone's behalf.

## A. Authentication & Onboarding Screens

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 1 | **Login Page** | `/login` | New | Root of the unauthenticated flow; default redirect target for any protected route. | Unassigned | Authenticate with email + password; links to Register and Forgot Password; on success routes to Home. | Public &middot; Visitor | Must exist on both Web and native. | 3 |
| 2 | **Register Page** | `/register` | New | Child of Login (linked from "Create account"). | Unassigned | Create an account with email, display name, and password entered twice. | Public &middot; Visitor | On success: auto-login to Home + activation email sent automatically. | 1 |
| 3 | **Forgot Password Page** | `/forgot-password` | New | Child of Login (linked from "Forgot password?"). | Unassigned | User enters their account email to request a password reset. | Public &middot; Visitor | Triggers an emailed reset link or an emailed OTP. | 4 |
| 4 | **Reset Password Page** | `/reset-password/:token` | New | Child of Forgot Password (reached via emailed link/OTP entry). | Unassigned | User follows the emailed link or enters the emailed OTP, then sets a new password. | Public (token/OTP-gated) &middot; Visitor | After success, routes back to Login; no auto-login. | 4 |
| 5 | **Account Activation Banner** | n/a &ndash; overlay, not routed | New | Persistent overlay on Home and its children until activation succeeds. | Unassigned | Reminds an unverified user to activate their account via the emailed link. | Auth &middot; Any user (unverified) | Disappears automatically on activation; full functionality stays usable while shown. | 2 |

## B. Home & Note Browsing

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 6 | **Home Page** (Notes Grid / List) | `/home` (post-login root) | New | Root of the authenticated app; parent of Note Editor, Label Management, Shared-with-me, AI Q&A, Profile. | Unassigned | Lists the user's notes (grid by default, togglable list view); live search; label filter; sort with pinned notes first; status icons; entry point to create a note. | Auth &middot; Any user | View mode is a persisted preference; search debounces ~300 ms. | 9, 10, 17, 18, 19, 22 |
| 7 | **Label Management Screen/Dialog** | `/labels` (or modal over `/home`) | New | Child of Home (opened from a "Manage labels" action). | Unassigned | List all labels; add, rename, and delete labels. | Auth &middot; Any user | Renaming updates the label everywhere; deleting a label never damages its notes. | 20 |

## C. Note Creation, Editing & Organization

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 8 | **Note Editor Page** (single reusable create/edit screen) | `/notes/new` and `/notes/:noteId` | New | Child of Home; parent of Attachment Picker, Delete Confirmation, Note Password dialog, Share dialog, AI Summary panel. | Unassigned | Title + content (only mandatory inputs); auto-saves without a Save button; entry points to labels, attachments, pin, note password, share, AI summary. | Auth+Owner/Editor | Same screen for create and edit &ndash; no separate implementation. Preserves latest valid change through pause/close/reopen. | 11, 12, 14 |
| 9 | **Attachment Picker** (gallery / camera / file picker overlay) | n/a &ndash; native picker over `/notes/:noteId` | New | Child of Note Editor. | Unassigned | Add one or more image, video, or general-file attachments to a note. | Auth+Owner/Editor | Validates file type/size; handles cancel/denied-permission gracefully. | 15, 16 |
| 10 | **Delete Confirmation Dialog** | n/a &ndash; modal over `/home` or `/notes/:noteId` | New | Child of Home / Note Editor. | Unassigned | Explicit confirm prompt shown before a note is permanently deleted. | Auth+Owner/Editor | Always required before deletion proceeds. | 13 |
| 11 | **Note Password Screen/Dialog** (set / change / unlock) | n/a &ndash; modal over `/notes/:noteId` | New | Child of Note Editor. | Unassigned | Enable/disable note password protection; change an existing note password; unlock before viewing/editing/deleting/sharing/summarizing. | Auth+Owner/Editor (owner sets/changes; owner or authorized collaborator unlocks) | Confirms current password before change/disable; enforced server-side. | 23, 24 |

## D. Sharing & Real-Time Collaboration

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 12 | **Share & Permission Management Dialog** | n/a &ndash; modal over `/notes/:noteId` | New | Child of Note Editor. | Unassigned | Add recipients by email; assign read-only or edit permission; change or revoke access anytime. | Auth+Owner | Recipient must be a registered account; enforcement via backend authorization. | 25 |
| 13 | **Shared-with-me Page** | `/shared` | New | Child of Home (top-level nav item, sibling of Home's note list). | Unassigned | Lists notes shared with the current user: permission level, sharer, timestamp. | Auth &middot; Collaborator | Edit-permission notes open into real-time collaborative editing; read-only notes cannot be modified. | 25, 26 |

## E. AI-Powered Features

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 14 | **AI Summary Panel** (inside Note Editor) | n/a &ndash; panel within `/notes/:noteId` | New | Child of Note Editor. | Unassigned | Generates a concise LLM summary of the open note; regenerable on demand. | Auth+Owner/Editor | Shown separately from note body; never silently overwrites note content. | 27 |
| 15 | **AI Question & Answer Page** | `/ask` | New | Child of Home (top-level nav item). | Unassigned | User asks a natural-language question; system returns a grounded answer with links to source notes. | Auth &middot; Any user | Locked notes usable only after unlock; explicit "not enough information" when applicable. | 28 |

## F. Account & Preferences

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 16 | **User Profile Page** | `/profile` | New | Child of Home (via avatar/menu); parent of Edit Profile, Change Password, Preferences. | Unassigned | View current display name and avatar (default avatar when none set). | Auth &middot; Any user | Entry point to the three screens below. | 5 |
| 17 | **Edit Profile Page/Dialog** | `/profile/edit` | New | Child of User Profile. | Unassigned | Update display name and/or avatar image. | Auth &middot; Any user | Validates uploaded avatar file type/size. | 6 |
| 18 | **Change Password Page** | `/profile/change-password` | New | Child of User Profile. | Unassigned | Change account password: current password + new password twice. | Auth &middot; Any user | Session handled securely after a successful change. | 7 |
| 19 | **User Preferences Page** | `/profile/preferences` | New | Child of User Profile. | Unassigned | Adjust note font size, note color, light/dark theme; stores default note view. | Auth &middot; Any user | Preferences persist per user across submitted platforms. | 8 |

## G. System / Cross-Cutting UI Elements

| # | Page / Screen | Route | Status | Parent / Child Hierarchy | Owner / Assignee | Functionality | Access & Who Uses It | Notes | Rubric ID |
|---|---|---|---|---|---|---|---|---|---|
| 20 | **Offline / Sync Status Indicator** | n/a &ndash; overlay on any authenticated route | New | Overlay attached to Home and its children. | Unassigned | Communicates offline mode, pending-sync, and sync-failure states, with a retry action. | Auth &middot; Any user | Previously loaded notes stay readable offline; queued edits sync on reconnect. | 31 |
| 21 | **Loading / Empty / Error States** | n/a &ndash; shared pattern, no dedicated route | New | Cross-cutting &ndash; applies inside every screen above. | Unassigned | Every screen presents a meaningful loading, empty, success, and error state. | &ndash; &middot; Any user | Required for touch, mouse, and keyboard input across all screens. | 29 |

## Summary

**21 screens/UI elements** total: 5 authentication/onboarding, 2 home/browsing, 4 note creation & organization, 2 sharing/collaboration, 2 AI-powered, 4 account/preferences, 2 cross-cutting system elements. All currently **Status: New** (project has not started implementation) and all **Owner/Assignee: Unassigned** &ndash; the team should claim rows during sprint planning. Every row traces to at least one rubric ID and covers 100% of the flows in the User Flow Diagrams.
