# my-task-manager-spec

Shared OpenAPI contract for the Task Manager backend and frontend.

## v1.7.0

- Add `DELETE /users/me` to permanently delete the authenticated user's account and associated application data
- Account deletion requires recent OTP re-authentication (`403`, code `REAUTHENTICATION_REQUIRED`)
- Pre-deletion subscription checks against fresh provider data; renewable subscriptions must be canceled first (`409`, `AccountDeletionBlockedError` with `blockingSubscriptions` and per-provider `cancellationAction`)
- `503` with code `SUBSCRIPTION_VERIFICATION_UNAVAILABLE` when subscription status cannot be verified (no data deleted)
- Successful deletion returns `204`, revokes sessions, clears the refresh-token cookie, and queues durable provider cleanup
- Add schemas `AccountDeletionBlockingSubscription` and `AccountDeletionBlockedError`
- Extend `ErrorResponse` with optional machine-readable `code` field

## v1.6.0

- Centralize timezone as a user preference: only `PATCH /users/me/preferences` may update the persisted timezone
- `GET /users/me` returns the stored IANA timezone on the `User` response
- `UpdateUserPreferencesRequest.timezone` remains optional (at least one of `language` or `timezone` required)
- Add `minLength: 1` on `IanaTimezone`; semantic IANA validation deferred to backend implementation
