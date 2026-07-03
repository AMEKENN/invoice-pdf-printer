# AuthHub Login User Center Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add AuthHub-backed login/register modals, a lightweight user center modal, invite-code registration flow, and Pro model membership gating.

**Architecture:** Keep AuthHub integration isolated in `src/api/authhub.ts` and shared auth state in `src/composables/useAuth.ts`. UI lives under `src/views/user/`, while `Home.vue` coordinates modal visibility, URL invite handling, and Pro model gating.

**Tech Stack:** Vue 3 `<script setup>`, TypeScript, Tailwind CSS, axios/fetch-compatible browser APIs, existing localStorage cache helpers, existing Node-style `.test.ts` tests.

---

## File Structure

- Create `invoice-server/src/api/authhub.ts`: AuthHub request wrapper, response normalization, typed API functions, app-key/base-url config.
- Create `invoice-server/src/api/authhub.test.ts`: Lightweight tests for headers, token injection, error normalization, and invite URL parsing helpers where practical.
- Create `invoice-server/src/composables/useAuth.ts`: Singleton auth state and actions for login, register, refresh, membership, logout.
- Create `invoice-server/src/composables/useAuth.test.ts`: Storage/login/logout state behavior with injected fake API.
- Create `invoice-server/src/views/user/LoginModal.vue`: Email/password login and register modal, URL invite read-only field support.
- Create `invoice-server/src/views/user/UserCenterModal.vue`: White user center modal shell with top summary and horizontal tabs.
- Create `invoice-server/src/views/user/tabs/ProfileTab.vue`: Lightweight account details.
- Create `invoice-server/src/views/user/tabs/MembershipTab.vue`: Lightweight membership status and expiry display.
- Create `invoice-server/src/views/user/tabs/InviteTab.vue`: Invite summary, copyable code/link, first page invite records, collapsible rules.
- Create `invoice-server/src/views/user/tabs/SettingsTab.vue`: Logout and local cache actions.
- Modify `invoice-server/src/components/ui/AmountRecognitionModeSelect.vue`: Let parent approve Pro selection before switching to `ai`.
- Modify `invoice-server/src/views/Home.vue`: Add account button, login/user-center modals, invite URL auto-register behavior, and Pro gating.
- Modify `invoice-server/src/env.d.ts`: Add `VITE_AUTHHUB_API_URL` and `VITE_AUTHHUB_APP_KEY`.

## Task 1: AuthHub API Layer

**Files:**
- Create: `invoice-server/src/api/authhub.ts`
- Create: `invoice-server/src/api/authhub.test.ts`
- Modify: `invoice-server/src/env.d.ts`

- [ ] **Step 1: Write failing API-layer tests**

Create `invoice-server/src/api/authhub.test.ts` with tests that assert AuthHub requests send `X-App-Key`, send bearer tokens when provided, normalize errors, and expose the configured app key.

- [ ] **Step 2: Run tests and verify they fail**

Run: `cd invoice-server && npx tsx src/api/authhub.test.ts`

Expected: FAIL because `src/api/authhub.ts` does not exist.

- [ ] **Step 3: Implement `authhub.ts`**

Add types for user, membership, invite summary, invite records, login/register payloads, and API functions:

- `authHubConfig`
- `setAuthHubTokenGetter`
- `authHubRequest`
- `loginWithAuthHub`
- `registerWithAuthHub`
- `fetchAuthHubMe`
- `fetchMembershipStatus`
- `fetchInviteSummary`
- `fetchInviteRecords`
- `useInviteCode`
- `getInviteCodeFromLocation`
- `buildInviteShareUrl`

- [ ] **Step 4: Update env typing**

Add `VITE_AUTHHUB_API_URL` and `VITE_AUTHHUB_APP_KEY` to `invoice-server/src/env.d.ts`.

- [ ] **Step 5: Run tests and verify they pass**

Run: `cd invoice-server && npx tsx src/api/authhub.test.ts`

Expected: PASS with `authhub api tests passed`.

## Task 2: Shared Auth State

**Files:**
- Create: `invoice-server/src/composables/useAuth.ts`
- Create: `invoice-server/src/composables/useAuth.test.ts`

- [ ] **Step 1: Write failing auth-state tests**

Create tests for token restoration, login storing token/user/membership, logout clearing token, and `isProActive` from membership status.

- [ ] **Step 2: Run tests and verify they fail**

Run: `cd invoice-server && npx tsx src/composables/useAuth.test.ts`

Expected: FAIL because `useAuth.ts` does not exist.

- [ ] **Step 3: Implement `useAuth.ts`**

Implement singleton state with:

- `token`
- `user`
- `membership`
- `membershipStatus`
- `authReady`
- `isLoggedIn`
- `isProActive`
- `login`
- `register`
- `refreshMe`
- `refreshMembership`
- `logout`
- `initializeAuth`

Use `AUTHHUB_TOKEN` in localStorage and call `setAuthHubTokenGetter`.

- [ ] **Step 4: Run tests and verify they pass**

Run: `cd invoice-server && npx tsx src/composables/useAuth.test.ts`

Expected: PASS with `useAuth tests passed`.

## Task 3: Login and User Center UI

**Files:**
- Create: `invoice-server/src/views/user/LoginModal.vue`
- Create: `invoice-server/src/views/user/UserCenterModal.vue`
- Create: `invoice-server/src/views/user/tabs/ProfileTab.vue`
- Create: `invoice-server/src/views/user/tabs/MembershipTab.vue`
- Create: `invoice-server/src/views/user/tabs/InviteTab.vue`
- Create: `invoice-server/src/views/user/tabs/SettingsTab.vue`

- [ ] **Step 1: Build `LoginModal.vue`**

Implement a 420px white modal with login/register modes, email/password fields, optional invite field, read-only invite field when provided by URL, loading and error states, and `success`/`close` emits.

- [ ] **Step 2: Build `UserCenterModal.vue`**

Implement a 720px white modal with title bar, top identity summary, horizontal tabs, and content scroll area. Support initial tab values `profile`, `membership`, `invite`, `settings`.

- [ ] **Step 3: Build lightweight tabs**

Implement Profile, Membership, Invite, and Settings tabs according to the spec. Invite tab loads summary/records only when mounted and caches in component state for the current modal session.

- [ ] **Step 4: Type-check UI components**

Run: `cd invoice-server && pnpm type-check`

Expected: PASS, or fail only with actionable TypeScript errors from the new files that must be fixed before continuing.

## Task 4: Home Integration and Pro Gating

**Files:**
- Modify: `invoice-server/src/components/ui/AmountRecognitionModeSelect.vue`
- Modify: `invoice-server/src/views/Home.vue`

- [ ] **Step 1: Update mode selector contract**

Change `AmountRecognitionModeSelect.vue` so selecting `ai` emits `request-pro-mode` instead of immediately mutating state. Add an exposed `confirmMode(next)` method for the parent to complete the switch after auth/membership approval.

- [ ] **Step 2: Wire Home auth modals**

In `Home.vue`, import `useAuth`, `LoginModal`, and `UserCenterModal`. Add state for login modal visibility, user center visibility, login initial mode, locked invite code, active user center tab, and pending Pro selection.

- [ ] **Step 3: Implement account entry**

Add a right-side rounded account button. If logged in, show email initial and open user center. If logged out, open login modal.

- [ ] **Step 4: Implement invite URL behavior**

On mounted, call `initializeAuth()`. If URL contains `?invite=<code>` and the user is not logged in, open `LoginModal` in register mode with the invite code locked.

- [ ] **Step 5: Implement Pro model gating**

When `request-pro-mode` fires:

- If not logged in, open login modal and set pending Pro selection.
- If logged in, call `refreshMembership()`.
- If active, call selector `confirmMode("ai")`.
- If inactive, open user center membership tab and keep default mode.
- After login success with pending Pro selection, re-run the same membership check.

- [ ] **Step 6: Remove automatic early-bird popup on mount**

Keep the early-bird modal available from its existing Pro/early-bird button, but do not auto-open it after adding account/login flows.

## Task 5: Verification

**Files:**
- Modify as needed based on verification failures.

- [ ] **Step 1: Run focused tests**

Run:

```bash
cd invoice-server
npx tsx src/api/authhub.test.ts
npx tsx src/composables/useAuth.test.ts
```

Expected: both PASS.

- [ ] **Step 2: Run type-check**

Run: `cd invoice-server && pnpm type-check`

Expected: PASS.

- [ ] **Step 3: Run build**

Run: `cd invoice-server && pnpm build`

Expected: PASS.

- [ ] **Step 4: Manual smoke with dev server**

Run: `cd invoice-server && pnpm dev -- --host 127.0.0.1`

Smoke checks:

- `/?invite=ABC123` opens register mode with locked invite code.
- Right account button opens login when logged out.
- Selecting Pro model while logged out opens login.
- User center opens after successful auth state is present.

## Self-Review

- Spec coverage: The plan covers AuthHub config, login/register, URL invite lock, user center tabs, invite records, Pro gating, token restoration, and verification.
- Scope check: The plan intentionally excludes payment, orders, phone login, nickname/avatar persistence, and router changes.
- Placeholder scan: No `TBD` or unspecified implementation task remains.
