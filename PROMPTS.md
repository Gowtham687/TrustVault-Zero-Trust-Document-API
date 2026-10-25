# TrustVault --- Prompts

This file contains reusable prompts for continuing development of the
TrustVault project. They are written to preserve the existing
architecture instead of asking an AI coding agent to rebuild the
application unnecessarily.

------------------------------------------------------------------------

## 1. Project context prompt

``` text
You are working on TrustVault, a zero-trust document management application.

Stack:
- React 18
- TypeScript
- Vite
- Supabase Auth
- Supabase PostgreSQL
- PostgreSQL Row-Level Security (RLS)
- Supabase Storage
- Supabase Realtime
- Lucide React

Core entities:
- profiles
- documents
- document_shares
- document_versions
- audit_events
- workspace_settings

Core security model:
- Documents are private by default.
- Ownership is based on auth.uid().
- PostgreSQL RLS is the source of truth for authorization.
- The Storage bucket is private.
- File access should use short-lived signed URLs.
- Document sharing is explicit and recipient-scoped.
- Administrator audit visibility is protected by RLS.
- Do not move authorization logic into the frontend as a substitute for database policies.

Before changing code:
1. Inspect the existing implementation.
2. Reuse existing types, hooks, data functions and UI patterns.
3. Preserve the current visual design.
4. Preserve existing RLS and Storage protections.
5. Do not expose secrets or make the Storage bucket public.
6. Do not claim a feature is implemented until the backend and frontend both support it.
7. Keep changes focused and production-oriented.
```

------------------------------------------------------------------------

## 2. Production security hardening prompt

``` text
Harden TrustVault's security without changing its visual design.

Audit the complete authorization path:
- Supabase Auth
- PostgreSQL RLS
- document ownership
- document_shares
- document_versions
- audit_events
- workspace_settings
- Storage object policies
- signed URL generation

Tasks:
1. Identify any authorization that is enforced only in React and move the enforcement to PostgreSQL/RLS where appropriate.
2. Ensure only administrators can modify workspace_settings at the database level.
3. Ensure users cannot change their own role.
4. Ensure document shares can only be created, updated or revoked by document owners.
5. Ensure revoked recipients cannot access document rows or Storage objects.
6. Ensure document_versions follow the same access model as documents.
7. Ensure audit records cannot be modified or deleted by ordinary users.
8. Review SECURITY DEFINER functions for fixed search_path and minimum privileges.
9. Verify the private Storage bucket cannot be accessed anonymously.
10. Preserve short-lived signed URL access.

Do not weaken any existing RLS policy just to make the UI work. If a UI feature conflicts with the security model, fix the feature rather than bypassing authorization.
```

------------------------------------------------------------------------

## 3. Complete document versioning prompt

``` text
Implement document versioning in TrustVault using the existing document_versions table.

Requirements:
- Only the document owner can upload a new version.
- Each version must receive the next version_number atomically.
- Store each version as a separate private Storage object.
- Keep the current document version in documents.current_version.
- Preserve access rules from the parent document.
- Allow authorized users to view/download the currently permitted version.
- Show version history in the document viewer.
- Record upload/version events in audit_events.
- Prevent users from accessing versions after their document access is revoked.

Important:
Do not trust a version number or storage path supplied by the browser. Authorization and ownership must be checked server-side through RLS/database logic.
```

------------------------------------------------------------------------

## 4. Complete invitation and role-management prompt

``` text
Implement the Users & Roles area as a real administrator workflow.

Requirements:
- Only administrators can invite users.
- Only administrators can change roles.
- Users cannot assign themselves administrator privileges.
- Role changes must be recorded in audit_events.
- Pending invitations must not grant document access before acceptance.
- Display administrator, owner and viewer roles clearly.
- Add confirmation before destructive or privilege-changing actions.
- Prevent an administrator from accidentally removing the last administrator.
- Enforce all permissions in PostgreSQL/RLS, not only in React.

Keep the existing TrustVault design system and table layout.
```

------------------------------------------------------------------------

## 5. Audit export prompt

``` text
Implement the Audit activity "Export log" feature.

Requirements:
- Only administrators can export audit records.
- Export the currently available audit records with useful fields:
  event type, actor, document, target user, timestamp and metadata.
- Use CSV as the default export format.
- Respect RLS and never export records the current user cannot read.
- Do not expose service-role credentials in the browser.
- Add a clear success/error state.
- Record an audit event for an audit-log export if the security model supports it.
- Keep the existing Audit activity UI.
```

------------------------------------------------------------------------

## 6. Document sharing hardening prompt

``` text
Review and improve TrustVault's sharing flow.

Current intended model:
- Private
- Specific people
- All signed-in users

Implement:
1. Search workspace members by a stable identifier rather than relying only on display_name.
2. Prevent sharing a document with the current owner.
3. Prevent duplicate active shares.
4. Allow the owner to revoke a share.
5. Record share, permission-change and revoke events.
6. Make sure revoked users immediately lose access.
7. Enforce the same rules through PostgreSQL RLS and Storage policies.
8. Preserve the Reader and Editor permission model.
9. Keep the document private unless the owner explicitly changes its sharing policy.
```

------------------------------------------------------------------------

## 7. Realtime reliability prompt

``` text
Review TrustVault's Supabase Realtime implementation.

Ensure realtime updates work correctly for:
- documents
- document_shares
- audit_events
- workspace_settings

Requirements:
- Avoid duplicate subscriptions.
- Clean up channels when React components unmount.
- Refresh only the affected data where practical.
- Handle reconnects gracefully.
- Show a meaningful connection state.
- Do not treat realtime events as an authorization mechanism; RLS remains the security boundary.
- Preserve the current UI and loading/error states.
```

------------------------------------------------------------------------

## 8. Testing prompt

``` text
Create a security-focused test plan for TrustVault.

Test at minimum:

Authentication:
- unauthenticated access
- valid sign-in
- invalid credentials
- account creation

Documents:
- owner upload
- unsupported file type
- file larger than 50 MB
- owner read
- owner download
- unauthorized user read
- unauthorized user update/delete

Sharing:
- owner shares with reader
- owner shares with editor
- recipient reads shared document
- revoked recipient loses access
- non-owner attempts to share

Roles:
- administrator access
- viewer restrictions
- self-role escalation attempt

Audit:
- upload event
- view event
- share event
- administrator audit visibility
- non-administrator audit denial

Storage:
- anonymous object access
- owner object access
- shared object access
- revoked object access

Settings:
- administrator update
- non-administrator update attempt
- invalid file-size configuration

For every security test, verify the result at the database/Storage policy level rather than relying only on UI behavior.
```

------------------------------------------------------------------------

## 9. Deployment prompt

``` text
Prepare TrustVault for a production deployment.

Review:
- environment variables
- Supabase Auth redirect URLs
- Storage bucket privacy
- RLS policies
- Realtime publication
- database migrations
- account-specific migration files
- error handling
- build configuration
- source-map/security considerations
- secret handling
- production logging

Run:
npm install
npm run build

Fix TypeScript or build errors without weakening security.

Before deployment, explicitly identify any prototype-only feature that should not be presented as production-ready.
```

------------------------------------------------------------------------

## 10. UI/UX improvement prompt

``` text
Improve TrustVault's UI/UX while preserving its existing identity.

Keep:
- dark visual language
- TrustVault branding
- security-focused tone
- responsive layout
- current navigation structure
- Lucide icons

Improve:
- accessibility
- keyboard navigation
- loading states
- error messages
- empty states
- mobile usability
- focus states
- confirmation dialogs for destructive actions
- clear permission labels
- document status and sharing indicators

Do not replace the application with a generic dashboard template. Make incremental improvements to the existing design.
```

------------------------------------------------------------------------

## 11. Code-review prompt

``` text
Perform a senior-level code review of TrustVault.

Look for:
- security vulnerabilities
- broken authorization assumptions
- RLS gaps
- Storage policy gaps
- race conditions
- unsafe client-side trust
- missing error handling
- duplicate Supabase requests
- realtime subscription leaks
- TypeScript issues
- accessibility problems
- misleading UI states
- incomplete features presented as complete
- hard-coded deployment-specific values

For each issue provide:
1. Severity
2. File/location
3. Why it matters
4. Recommended fix
5. Whether the fix requires a migration

Do not rewrite working code unnecessarily.
```

------------------------------------------------------------------------

## 12. Feature implementation prompt

``` text
Add the following feature to TrustVault:

[DESCRIBE FEATURE HERE]

Constraints:
- Keep React + TypeScript + Vite.
- Keep Supabase as the backend.
- Preserve the existing database/RLS security model.
- Use existing hooks and data utilities where possible.
- Add migrations when the data model changes.
- Add appropriate audit events for security-sensitive actions.
- Do not make the Storage bucket public.
- Do not put Supabase service-role keys in the frontend.
- Preserve the existing TrustVault visual design.
- Make the feature responsive.
- Handle loading, success and failure states.
- Update README.md if the feature changes the architecture or setup.

Before coding, briefly identify:
- affected frontend files
- affected database tables
- required RLS changes
- required Storage changes
- required audit events

Then implement the feature and run the production build.
```

------------------------------------------------------------------------

## Development principle

When extending TrustVault, prefer this order:

``` text
Identity
   ↓
Database authorization
   ↓
Storage authorization
   ↓
Auditability
   ↓
Frontend experience
```

The UI should make secure actions easy, but the UI should never be the
security boundary.
