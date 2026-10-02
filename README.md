# CheckIn-App Bruno Collection

API test collection for the CheckIn-App Laravel API (`checkin-api/routes/api.php`), using [Bruno](https://www.usebruno.com/).

## Setup

1. Open Bruno → `Open Collection` → select this folder (`checkin-api/bruno-collection/`).
2. Open the **Environments** tab (`Ctrl+Shift+E` / `Cmd+Shift+E`) and create/edit the `local`, `lightsail`, or `production` environment, or point `baseUrl` at your own instance.
3. Select the environment you want to run against.

| Environment | baseUrl |
|---|---|
| `local` | `http://localhost/api/v1` |
| `lightsail` | `http://54.169.6.190/api/v1` |
| `production` | `http://13.213.228.82/api/v1` |

## Authentication flow

Tokens are stored in the selected environment automatically after login — **run login first**:

1. `Auth/Login` → sets `auth_token` (user bearer token) via an after-response script.
2. `Admin/Admin Login` → sets `admin_token` (admin/host/facilitator bearer token).

`User/*`, `Admin/*` folders inherit their bearer token at the folder level, so every request in them is authenticated automatically. `Public/*` and `Events/*` are unauthenticated or carry their own token where needed.

## Collection layout

| Folder | Base path | Auth |
|---|---|---|
| `Auth/` | `/auth/*`, `/logout` | none |
| `User/` | `/user`, `/delete`, `/hosts/*` | `{{auth_token}}` |
| `Admin/` | `/admin/*`, `/events/*` (management) | `{{admin_token}}` |
| `Events/` | `/events`, `/events/nearby`, `/scan` | mixed |
| `Event Missions/` | `/events/{id}/missions`, `/missions/scan` | mixed |
| `Event Winners/` | `/events/{id}/winners` | mixed |
| `Public/` | `/home`, `/events`, `/hosts` | none |

## API contract

For response envelopes, error codes, rate limits, and client (Android/Kotlin) integration guidance, see [`docs/API_CONTRACT.md`](../docs/API_CONTRACT.md).

## Notes

- `Auth/Register` and `Auth/Reset Password` require a valid Firebase ID token (`firebase_token`) for the phone number — obtain it via the mobile app OTP flow. There is no bypass token. Verification must be a phone sign-in within the last five minutes, for the submitted number, with a non-revoked token and an enabled Firebase account.
- `Auth/Change Password` requires an authenticated session (`auth_token` from `Auth/Login`) and the current password.
- `User/Personal Details/Interests.yml` is a public endpoint (`GET /interests`) included here for the personal-details workflow.
- Requests use path variables like `{event_id}`, `{admin_id}`, `{user_id}` — replace them with real UUIDs from the database or from earlier responses before sending.
- The `Single Session` scratch folder from the legacy collection was intentionally dropped (it contained hardcoded/leaked tokens).

## Account access checks

`Auth Security/` verifies mobile/admin separation and protects the legacy `/users` listing. Run mobile and super-admin login first; `Legacy User Listing` requires a super-admin `admin_token`. Blocked mobile users and inactive admins receive 403 even with an existing bearer token. Shared event reads and `/logout` remain available to both active account types.

Login limits are 5 requests per normalized account per minute and 20 per IP. Forgot-password limits are 6/account and 24/IP; reset-password limits are 10/account and 40/IP. Recovery requests return the same successful response for registered and unknown contacts/emails. Set `firebase_token` from a real recent phone OTP flow before registration/reset requests.

## Token lifetime and lifecycle checks

Login responses include `token_expires_at`. Mobile tokens expire in 30 days and new admin tokens in 12 hours by default. Logins preserve other devices; mobile password change revokes other devices; either password-reset flow revokes all devices; logout revokes the current token.

`Auth Security/` also needs `blocked_token`, `inactive_admin_token`, and `expired_token` from dedicated test fixtures. These tokens must identify a blocked user, an inactive admin, and an expired mobile token respectively.

`Token Lifecycle/` is a destructive test workflow: **run it only against a dedicated test database**. Prepare a mobile fixture at `+639956421817` with password `password`, its existing second-device token as `auth_other_token`, a super-admin fixture at `admin@checkin.app`, its token as `admin_token`, and a valid broker reset token as `admin_reset_token`. Execute the folder in sequence. It changes both passwords and revokes fixture tokens. Recreate/reset fixtures before each run. The main `Auth/Change Password` request also changes credentials and revokes other devices.

## Browser sessions and deletion handoff

The `Browser Session` folder runs in sequence: CSRF bootstrap, missing-CSRF rejection, cookie login/profile, logout protection, and post-logout denial. Set `apiOrigin` to the API origin (without `/api/v1`), `frontendOrigin` to a trusted SPA origin, and `browser_admin_email` / secret `browser_admin_password` to an active test admin. The scripts capture cookies and the decoded XSRF token. Reset `browser_cookies`, `browser_cookie`, and `xsrf_token` before a new workflow if needed. Keep runtime cookie/link values out of version control.

**`Account Deletion` deletes the account identified by `deletion_phone`. Run it only against disposable fixtures in a dedicated test database.** Set secret `deletion_password` and a separate surviving `manual_deletion_phone` using that password. Run the whole folder in order: mobile login, issue link, CSRF, single-use exchange, scoped access, replay/wrong-password/CSRF denial, confirmed deletion, revoked bearer token, and manual session creation. It creates no bearer credential for the browser. Do not run the entire collection against a development or production account without reviewing destructive requests.

Both folders can run with Bruno CLI using an environment JSON file containing these variables:

```sh
bru run 'Browser Session' 'Account Deletion' --env-file /tmp/auth-test-environment.json --sandbox=developer
```

Mirror request and environment changes into the sibling `Checkin-bruno-collection` repository. The HTTP workflows complement Laravel regressions covering expiration, account status, password-reset invalidation, and preservation of an existing admin session during deletion.
