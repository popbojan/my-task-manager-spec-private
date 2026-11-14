# my-task-manager-spec

Shared OpenAPI contract for the Task Manager backend and frontend.

## v1.16.0

- Add global daily Today-board reminder preferences on the user: `notificationEnabled` (default `true`) and `notificationTime` (default `20:00`, `HH:mm` in the user's IANA timezone)
- Returned on `User` (`GET /users/me`) and `UserPreferencesResponse`; writable via `PATCH /users/me/preferences`
- Independent of per-recurring-task reminders; time is always set and editable while the switch is off so the user can prepare a time and enable later

## v1.15.0

- Raise daily history note limit from 2000 to 10000 Unicode code points (`PutHistoryNoteRequest`, `HistoryNoteResponse`, `HistoryDay.note`, `PUT /history/{date}/note`)

## v1.14.0

- When a new local calendar day has begun but the recurring-task daily reset has not yet run (cron delay or failure), every recurring-task mutating operation and every history write operation rejects with `409` / `DAILY_RESET_REQUIRED`; clients must reload and retry
- Mutating: `POST|PATCH|DELETE /recurring-tasks`, `POST /history/yesterday/repair`, `PUT|DELETE /history/{date}/note`
- Related GETs (`GET /recurring-tasks`, `GET /recurring-tasks/{id}`, `GET /recurring-task-progress`, `GET /history`) always perform a pending manual reset when due, then return the current day state (no error)

## v1.13.0

- Change recurring-task daily reset from 00:05 to 00:00 local time in the user's configured IANA timezone (`IanaTimezone`, `lastResetAt`, `nextResetAt`, history day finalization)

## v1.12.3

- Add nullable `mobileAppVersion` on `User` (`GET /users/me`) — last version reported by the mobile app; still writable via `PATCH /users/me/preferences` (not returned on `UserPreferencesResponse`)

## v1.12.2

- No API changes; semver bump for web client release (history repair UI)

## v1.12.1

- Add `POST /history/yesterday/repair` to retroactively fix **yesterday's** finalized **missed** day (e.g. 6/7); response is the updated `HistoryDay` only (progress via `GET /recurring-task-progress`)
- No `{date}` path parameter — the backend selects the previous calendar day in the user's IANA timezone
- Add `HistoryDay.repairedAt`

## v1.11.1

- Clarify recurring-task `notificationTime`: may stay set when `notificationEnabled` is false so the user's last chosen reminder time is preserved (Create, Update, and `RecurringTask` response)

## v1.11.0

- Add `notificationEnabled` and `notificationTime` on `RecurringTask` (daily reminder preference per task)
- Support both fields on `CreateRecurringTaskRequest` and `UpdateRecurringTaskRequest`
- `notificationTime` is local `HH:mm` in the user's IANA timezone; non-null when reminders are enabled

## v1.10.4

- Change default `Language` in the OpenAPI schema from `de` to `en` (example and default)

## v1.10.3

- Add write-only `webLastLoginAt` and `webLastSeenAt` on `PATCH /users/me/preferences` for web-app tracing (UTC instants; not returned in API responses)

## v1.10.2

- Add `personalFocusTaskName` and `personalFocusMotivation` on `User` and `UserPreferencesResponse` (nullable until set via personal intro)
- Allow updating both fields via `PATCH /users/me/preferences`

## v1.10.1

- Add optional `mobileAppVersion` on `PATCH /users/me/preferences` (write-only; mobile reports installed version, not returned in API responses)

## v1.10.0

- Remove `User.isNewUser` (deprecated onboarding flag)
- Add nullable onboarding timestamps on `User`: `personalIntroSeenAt`, `mobileOnboardingSeenAt`, `webOnboardingSeenAt` (`GET /users/me`)
- Allow setting onboarding timestamps via `PATCH /users/me/preferences`; include them in `UserPreferencesResponse`

## v1.9.3

- Add `currentLevel` to `HistoryDay` (snapshot of global mastery level after each evaluated calendar day; nullable for `unknown` and `server_skipped`; range 1–24)

## v1.9.2

- Add `currentStreak` to `HistoryDay` (snapshot of global recurring-task streak after each evaluated calendar day; nullable for `unknown` and `server_skipped`)

## v1.9.1

- Add `server_skipped` to `HistoryDayStatus` for history days the backend did not finalize or evaluate under normal daily rules

## v1.9.0

- Add `GET /history?month=YYYY-MM` for monthly recurring-task daily history and notes (board tasks excluded; days ordered by date ascending)
- Add `PUT /history/{date}/note` to create or replace a daily note (1–2000 Unicode code points, non-blank after trim)
- Add `DELETE /history/{date}/note` to remove a note without changing daily task results (`204` even when absent)
- Add schemas `HistoryDayStatus`, `HistoryDay`, `HistoryResponse`, `PutHistoryNoteRequest`, and `HistoryNoteResponse`
- Document day statuses (`in_progress`, `success`, `missed`, `no_tasks`, `unknown`), provisional current-day results, and alignment with existing recurring-task progress and local reset rules

## v1.7.1

- Clarify `DELETE /users/me` subscription blocking: only running subscriptions block deletion; subscriptions already set to cancel at period end do not block

## v1.7.0

- Add `DELETE /users/me` to permanently delete the authenticated user's account and associated application data
- Account deletion requires recent OTP re-authentication (`403`, code `REAUTHENTICATION_REQUIRED`)
- Pre-deletion subscription checks against fresh provider data; running subscriptions must be canceled first (`409`, `AccountDeletionBlockedError` with `blockingSubscriptions` and per-provider `cancellationAction`)
- `503` with code `SUBSCRIPTION_VERIFICATION_UNAVAILABLE` when subscription status cannot be verified (no data deleted)
- Successful deletion returns `204`, revokes sessions, clears the refresh-token cookie, and queues durable provider cleanup
- Add schemas `AccountDeletionBlockingSubscription` and `AccountDeletionBlockedError`
- Extend `ErrorResponse` with optional machine-readable `code` field

## v1.6.0

- Centralize timezone as a user preference: only `PATCH /users/me/preferences` may update the persisted timezone
- `GET /users/me` returns the stored IANA timezone on the `User` response
- `UpdateUserPreferencesRequest.timezone` remains optional (at least one of `language` or `timezone` required)
- Add `minLength: 1` on `IanaTimezone`; semantic IANA validation deferred to backend implementation
