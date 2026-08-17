# my-task-manager-spec

Shared OpenAPI contract for the Task Manager backend and frontend.

## v1.5.0

- Add `User.timezone` (IANA time zone, required)
- Add required `timezone` on `OTPRequest` and `LoginRequest` (browser IANA timezone captured on registration)
- Add optional `timezone` on `UpdateUserPreferencesRequest` (language is now optional too; at least one field required)
- Add `timezone` to `UserPreferencesResponse`
- Add `IanaTimezone` schema
- Clarify `RecurringTask.lastResetAt` / `nextResetAt` as UTC instants aligned to 00:05 in the user's IANA timezone (not UTC midnight)
- Registration contract: clients must always send browser timezone on auth requests; backend must persist the supplied IANA timezone and must not substitute UTC
