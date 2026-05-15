# Product Requirements

Behavioural requirements for the FastAPI Item Manager. Each H2 below is one discrete requirement that should be testable end-to-end.

## User Login

A registered user can obtain an access token by POSTing their email and password to `/api/v1/login/access-token` as form-encoded credentials. A successful response returns a JWT bearer token; an invalid email or password returns HTTP 400 with `Incorrect email or password`. Inactive users are blocked even with correct credentials.

## User Signup

A new visitor can self-register through `POST /api/v1/users/signup` by providing email, password, and an optional full name. The email must be unique across the system; a duplicate email returns HTTP 400. Successful signup creates a non-superuser, active account and returns the public user record.

## Password Recovery and Reset

A user who has forgotten their password can request a recovery email via `POST /api/v1/password-recovery/{email}`. The system emails a single-use token to the registered address. The user can then call `POST /api/v1/reset-password/` with the token and a new password to update their credentials. Tokens are single-use and must not be reusable after a successful reset.

## User Self-Management

An authenticated user can view their own profile via `GET /api/v1/users/me` (handler `read_user_me`), update their email or full name via `PATCH /api/v1/users/me` (handler `update_user_me`), change their password via `PATCH /api/v1/users/me/password` (handler `update_password_me`), and permanently delete their own account via `DELETE /api/v1/users/me` (handler `delete_user_me`). Superusers cannot delete themselves through this endpoint.

## Admin User Management

A `superuser` can list all users via `read_users` (`GET /api/v1/users/`), create new users without going through signup via `create_user` (`POST /api/v1/users/`), fetch a specific user by ID via `read_user_by_id`, update any user's attributes via `update_user`, and delete any non-self user via `delete_user`. Non-superusers receive HTTP 403 on any of these endpoints.

## Item CRUD

An authenticated user can create items (`POST /api/v1/items/`), list their own items with pagination (`GET /api/v1/items/`), fetch one by ID, update its title or description (`PUT /api/v1/items/{id}`), and delete it (`DELETE /api/v1/items/{id}`). Items are owned by the user who created them and are stamped with that owner's ID at creation time.

## Item Search

An authenticated user can search their own items by a case-insensitive title substring via `GET /api/v1/items/search?q=<term>` (route handler `search_items`). Superusers may search across all items. An empty or missing `q` returns an empty list (`data: []`, `count: 0`) rather than the full catalogue. Pagination follows the same `skip` and `limit` parameters as `read_items`.

## Item Ownership Enforcement

A non-superuser may only read, update, or delete items they own. Attempting to access another user's item by ID returns HTTP 400 with `Not enough permissions`. Superusers see and may modify every item regardless of ownership.

## Authentication and Token Validation

`POST /api/v1/login/test-token` accepts a bearer token in the Authorization header and returns the user record the token represents. An invalid, expired, or missing token returns HTTP 401 or 403 as appropriate. This endpoint is used by the frontend to validate session resumption.
