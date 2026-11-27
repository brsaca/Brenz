# Contract: Authentication errors and waits

**Feature**: 001-account-onboarding  
**Date**: 2026-11-27

## Operations and wait policy

| Operation | Client pre-check | Server wait | On timeout / offline |
|-----------|------------------|-------------|----------------------|
| SignUp | email format; password ≥ 8 | **10 s** timeout (injected monotonic clock) | `noNetwork` **by design**; no session created |
| SignIn | email format; password ≥ 8 | **10 s** timeout (injected monotonic clock) | `noNetwork` **by design**; session unchanged |
| ResetPassword | email format | **10 s** timeout (injected monotonic clock) | `noNetwork` **by design**; no session; stay on screen |

A **10 s** server timeout maps to `noNetwork` by design (same kind as offline). Unlike splash/Intro **minimums**, this wait is an exact timeout. Primary actions stay disabled while `isSubmitting == true` (no double submit) on Sign Up, Sign In, and Reset Password. No email verification step after Sign Up.

## Error kinds surfaced in UI

| Domain kind | Screens | User effect |
|-------------|---------|-------------|
| `invalidCredentials` | Sign In | Inline/alert; remain on Sign In. Also used for **any client-side format failure** on Sign In (malformed email or password length < 8). Sign In MUST NOT return `weakPassword`. |
| `emailAlreadyInUse` | Sign Up | Remain on Sign Up |
| `weakPassword` | Sign Up only (and client length < 8 on Sign Up) | Remain; no request if client-blocked. **Not used on Sign In.** |
| `invalidEmail` | Sign Up, Reset Password | No request when client-invalid |
| `noNetwork` | Sign Up, Sign In, Reset Password | After wait / timeout by design; no session change |
| `profileLoadFailed` | Root (post-splash) | Retryable surface; session kept; no Home/Onboarding |

## Reset Password success contract

For every **valid-format** email (registered or not):

1. Show the same generic “link sent” confirmation.
2. Remain on Reset Password with confirmation visible.
3. Provide a clear path back to Sign In.
4. Do not establish a session.
5. Do not reveal whether the email is registered.

## Mapping boundary

- Firebase Auth error codes are translated inside `Data/Firebase/` only.
- No Firebase types cross into Domain or Presentation.
- Logs use `os.Logger` and never include email or nickname.
