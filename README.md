# my-task-manager-spec

Shared OpenAPI contract for the Task Manager backend and frontend.

## v1.6.0

- Centralize timezone as a user preference: only `PATCH /users/me/preferences` may update the persisted timezone
- `GET /users/me` returns the stored IANA timezone on the `User` response
- `UpdateUserPreferencesRequest.timezone` remains optional (at least one of `language` or `timezone` required)
- Add `minLength: 1` on `IanaTimezone`; semantic IANA validation deferred to backend implementation