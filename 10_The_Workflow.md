# 10 THE WORKFLOW (PROFESSIONAL GIT)

**Status**: ACTIVE
**Scope**: GLOBAL
**Enforcement**: STRICT

## 1. The "Main is Sacred" Law
*   **NEVER** commit directly to `main` (or `master`).
*   **NEVER** push broken code to `main`.
*   `main` must *always* be deployable.

## 2. The Feature Branch Standard
*   **Format**: `type/scope/description`
    *   `feat/auth/google-login`
    *   `fix/api/cors-error`
    *   `chore/deps/update-astro`
    *   `refactor/ui/sidebar-component`
*   **Lifecycle**:
    1.  `git checkout -b feat/my-feature`
    2.  Dev Loop (Code -> Test -> Commit)
    3.  `git push origin feat/my-feature`
    4.  **Pull Request** -> Review -> Merge
    5.  `git branch -d feat/my-feature`

## 3. Commit Discipline (Conventional Commits)
*   **Format**: `type(scope): description`
*   **Good**: `feat(auth): add google oauth provider`
*   **Bad**: `wip`, `fixed stuff`, `update`

## 4. The Staging Protocol (For Critical Infra)
*   When modifying "Heart" systems (DB, Auth, Stripe):
    1.  Deploy to **Staging** environment first.
    2.  Verify URL: `staging.myapp.com`
    3.  Only after verification, merge Staging branch to `main`.

## 5. Agent Behavior
*   If you are asked to "fix" something, **automatically** create a `fix/...` branch. Do not ask permission.
*   If you are asked to "build" something, **automatically** create a `feat/...` branch.
*   **Exception**: Documentation updates can go to `main` if trivial.
