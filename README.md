# CheckIn-App Bruno Collection

API test collection for the CheckIn-App Laravel API (`rewards-api/checkin-api`), using [Bruno](https://www.usebruno.com/). This repository is the standalone mirror of the collection shipped in the API repo (`checkin-api/bruno-collection/`).

## Setup

1. Open Bruno → `Open Collection` → select this folder.
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
| `User/` | `/user`, `/users`, `/delete`, `/hosts/*` | `{{auth_token}}` |
| `Admin/` | `/admin/*`, `/events/*` (management) | `{{admin_token}}` |
| `Events/` | `/events`, `/events/nearby`, `/scan` | mixed |
| `Event Missions/` | `/events/{id}/missions`, `/missions/scan` | mixed |
| `Event Winners/` | `/events/{id}/winners` | mixed |
| `Public/` | `/home`, `/events`, `/hosts` | none |

## API contract

For response envelopes, error codes, rate limits, and client (Android/Kotlin) integration guidance, see the [`AUTHENTICATION.md`](https://github.com/HeroAppsDev/rewards-api/blob/develop/checkin-api/docs/AUTHENTICATION.md) and [`API_CONTRACT.md`](https://github.com/HeroAppsDev/rewards-api/blob/develop/checkin-api/docs/API_CONTRACT.md) in the API repository.

## Notes

- `Auth/Register` and `Auth/Reset Password` require a valid Firebase ID token (`firebase_token`) for the phone number — obtain it via the mobile app OTP flow. `BYPASS_TOKEN` only works on local/dev setups.
- `Auth/Change Password` requires an authenticated session (`auth_token` from `Auth/Login`) and the current password.
- `User/Personal Details/Interests.yml` is a public endpoint (`GET /interests`) included here for the personal-details workflow.
- Requests use path variables like `{event_id}`, `{admin_id}`, `{user_id}` — replace them with real UUIDs from the database or from earlier responses before sending.
- The `Single Session` scratch folder from the legacy collection was intentionally dropped (it contained hardcoded/leaked tokens).
