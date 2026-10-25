# TrustVault --- Zero-Trust Document API

TrustVault is a secure document workspace built around a zero-trust
access model. It lets authenticated users upload private business
documents, view them through short-lived signed URLs, and explicitly
share files with selected workspace members. The application combines a
React/Vite frontend with Supabase Authentication, PostgreSQL Row-Level
Security (RLS), private Supabase Storage, realtime subscriptions, and an
administrator-focused audit trail.

## What the project does

-   **Authenticated workspace** --- Users sign in or create an account
    before accessing the vault.
-   **Private document storage** --- Uploaded files are stored in a
    private `trustvault-documents` bucket.
-   **Supported files** --- PDF, DOCX, and XLSX files up to 50 MB.
-   **Owner-based access** --- Document ownership is derived from the
    authenticated Supabase user.
-   **Explicit sharing** --- Owners can share a document with a
    workspace member as a `reader` or `editor`.
-   **Workspace-wide sharing model** --- The database supports private,
    specific-person, and all-signed-in-user sharing modes.
-   **Secure file access** --- The UI requests a one-hour signed URL
    only after the user is allowed to access the document.
-   **Realtime updates** --- Document, share, audit, and
    workspace-setting changes can update the UI through Supabase
    Realtime.
-   **Audit trail** --- Security and document activity is stored in
    `audit_events` and is visible to administrators.
-   **Role model** --- Profiles support `administrator`, `owner`, and
    `viewer` roles, with RLS protecting sensitive operations.
-   **Workspace controls** --- Administrators can configure the
    workspace name, default sharing mode, maximum file size, audit
    logging, and member-approval setting.

## Security model

TrustVault follows a **deny-by-default, identity-first** approach:

1.  A user authenticates through Supabase Auth.
2.  The application uses the authenticated user's ID (`auth.uid()`) as
    the identity boundary.
3.  PostgreSQL RLS policies decide which rows the user can read or
    modify.
4.  Storage policies restrict files to the owning user or an authorized
    document recipient.
5.  File bytes are not exposed through a public bucket.
6.  A permitted file is opened through a temporary signed URL rather
    than a permanent public link.
7.  Important actions are written to the audit table for administrator
    visibility.

The database contains separate entities for profiles, documents,
document shares, document versions, audit events, and workspace
settings.

## Architecture

``` text
React + TypeScript + Vite
          |
          v
   Supabase Client SDK
      /           \
     v             v
Supabase Auth   PostgreSQL + RLS
                     |
                     +--> profiles
                     +--> documents
                     +--> document_shares
                     +--> document_versions
                     +--> audit_events
                     +--> workspace_settings
                     |
                     v
             Supabase Realtime

Supabase Storage
  private bucket
      |
      v
Short-lived signed URLs
```

### Frontend

-   React 18
-   TypeScript
-   Vite
-   Lucide React icons
-   Custom responsive CSS

### Backend / platform services

-   Supabase Authentication
-   Supabase PostgreSQL
-   PostgreSQL Row-Level Security
-   Supabase Storage
-   Supabase Realtime

## Main application areas

### 1. Authentication

The landing/authentication screen supports:

-   Sign in
-   Account creation
-   Full-name profile metadata
-   Password validation
-   Session persistence through Supabase Auth

### 2. Workspace overview

The overview displays:

-   Total documents
-   Documents shared with the current user
-   Storage usage
-   Realtime connection status
-   Recent documents
-   Latest audit activity

### 3. Documents

The document area supports:

-   Searching documents by title
-   Uploading PDF/DOCX/XLSX files
-   50 MB upload validation
-   Private-by-default uploads
-   Secure document viewing
-   Secure downloading through a signed URL
-   Sharing documents with workspace members

### 4. Sharing

A document owner can choose a workspace member and assign:

-   **Reader** --- view access
-   **Editor** --- intended editing permission level stored in the data
    model

The share is represented in `document_shares` and can be revoked through
the underlying data model.

### 5. Audit activity

Administrators can view the live audit feed. The audit schema supports
events such as:

-   Upload
-   View
-   Download
-   Edit
-   Delete
-   Share
-   Permission change
-   Revoke
-   Access denied
-   Role change

The current frontend explicitly records upload, view, and share activity
in the implemented flows, while the database model is prepared for the
broader event vocabulary.

### 6. Users & roles

The administration area displays workspace profiles and their roles:

-   Administrator
-   Owner
-   Viewer

The current UI includes the member list and an invite-user presentation
control; full invitation and role-management workflows are not
implemented as backend actions in this version.

### 7. Workspace settings

Administrators can configure:

-   Workspace name
-   Default sharing mode
-   Maximum file size
-   Audit logging preference
-   New-member approval preference

Settings are stored in a singleton `workspace_settings` record and are
published through Supabase Realtime.

## Database model

  -----------------------------------------------------------------------
  Table                               Purpose
  ----------------------------------- -----------------------------------
  `profiles`                          Workspace identity and role
                                      information

  `documents`                         Document metadata, owner, storage
                                      path, MIME type and sharing mode

  `document_shares`                   Explicit recipient permissions and
                                      revocation state

  `document_versions`                 Version metadata for future/current
                                      document history

  `audit_events`                      Security and document activity
                                      records

  `workspace_settings`                Singleton workspace configuration
  -----------------------------------------------------------------------

The migration files under `supabase/migrations/` create the schema, RLS
policies, storage policies, realtime configuration, profile trigger
behavior, and security fixes.

## Zero-trust design principles

TrustVault is designed around these principles:

-   **Never trust the browser alone.** Authorization is enforced in
    PostgreSQL and Storage policies.
-   **Least privilege.** Access is granted to the owner, an explicit
    recipient, or a defined signed-in-user sharing mode.
-   **Private by default.** Newly uploaded documents start with
    `private` sharing.
-   **Short-lived access.** Permitted files are exposed through signed
    URLs rather than public file URLs.
-   **Identity-bound ownership.** The database derives ownership from
    `auth.uid()`.
-   **Auditable activity.** Security-relevant actions can be recorded
    and reviewed.
-   **Realtime visibility.** Relevant changes can propagate to the UI
    without a manual refresh.

## Project structure

``` text
project/
├── src/
│   ├── App.tsx
│   ├── main.tsx
│   ├── styles.css
│   └── lib/
│       ├── data.ts
│       ├── hooks.ts
│       ├── supabase.ts
│       └── types.ts
├── supabase/
│   └── migrations/
├── public/
├── dist/
├── .env
├── index.html
├── package.json
├── package-lock.json
├── tsconfig.json
└── vite.config.ts
```

## Environment variables

Create a `.env` file containing the Supabase project values expected by
`src/lib/supabase.ts`.

Example:

``` env
VITE_SUPABASE_URL=your_supabase_project_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

Do not commit real credentials or private keys to source control.

## Installation

``` bash
npm install
```

## Development

``` bash
npm run dev
```

Vite will start the development server and print the local URL in the
terminal.

## Production build

``` bash
npm run build
```

To preview the production build locally:

``` bash
npm run preview
```

## Supabase setup

1.  Create a Supabase project.
2.  Configure the project URL and anon key in `.env`.
3.  Apply the SQL migrations in `supabase/migrations/` in chronological
    order.
4.  Confirm the private `trustvault-documents` storage bucket exists.
5.  Enable the required Supabase Realtime publication entries created by
    the migrations.
6.  Create a test user through the application.
7.  Verify that the user receives a corresponding profile.
8.  Test upload, view, sharing, signed URL access, and audit events with
    multiple accounts.

## Important implementation notes

-   The frontend currently limits uploads to PDF, DOCX, and XLSX.
-   The client enforces a 50 MB upload limit, and the database/storage
    configuration also limits files to 50 MB.
-   The current upload flow creates a document with `private` sharing.
-   The current sharing lookup is based on the workspace profile display
    name, even though the UI placeholder mentions email.
-   The `document_versions` table exists as the foundation for version
    history, but a complete version-upload UI is not present in this
    build.
-   The `editor` permission is stored in the share model, but a complete
    document-editing workflow is not implemented in the current
    frontend.
-   The Users & Roles page currently presents workspace members; invite
    and role-change actions are UI placeholders rather than complete
    backend workflows.
-   The Settings UI restricts editing to administrator profiles, while
    the current database migration for `workspace_settings` permits
    authenticated updates. For production, the database policy should
    also enforce administrator-only updates.
-   The current audit UI provides an export control visually, but an
    actual log-export implementation is not included.
-   The code contains a one-time migration that promotes a specific
    existing account to administrator. Before reusing the migrations for
    another deployment, review and replace that account-specific
    migration.

## Security testing checklist

Before production use, verify:

-   [ ] Anonymous users cannot read document metadata.
-   [ ] Anonymous users cannot access Storage objects.
-   [ ] A user cannot read another user's private document.
-   [ ] A user cannot modify another user's document.
-   [ ] A non-owner cannot create a share for a document.
-   [ ] A revoked share no longer grants access.
-   [ ] Signed URLs expire as expected.
-   [ ] Roles cannot be changed through normal browser writes.
-   [ ] Audit records cannot be edited by ordinary users.
-   [ ] Workspace settings are protected by database-level administrator
    authorization.
-   [ ] Real credentials are excluded from Git.
-   [ ] Production CORS, redirect URLs, password policies, and Supabase
    Auth settings are configured appropriately.

## Why TrustVault?

Traditional file sharing often treats possession of a link as sufficient
access. TrustVault instead treats **identity, authorization, and policy
as part of every document request**. This makes the project suitable as
a foundation for secure internal document workflows where organizations
need private storage, controlled sharing, and visibility into access
activity.

## Status

**Prototype / hiring-challenge implementation**

The project demonstrates the core zero-trust document-storage and
access-control architecture, while several enterprise features---such as
full invitations, role management, document version editing, audit
export, and complete policy enforcement for every UI setting---remain
suitable areas for further development.
