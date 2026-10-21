# my-task-manager-spec

Shared OpenAPI contract for the Task Manager backend and frontend.

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

- Add `isNewUser` to the `User` schema returned by `GET /users/me` (server-evaluated onboarding hint; clients must not derive it locally)
- Add `GET /history?month=YYYY-MM` for monthly recurring-task daily history and notes (board tasks excluded; days ordered by date ascending)
- Add `PUT /history/{date}/note` to create or replace a daily note (1–2000 Unicode code points, non-blank after trim)
- Add `DELETE /history/{date}/note` to remove a note without changing daily task results (`204` even when absent)
- Add schemas `HistoryDayStatus`, `HistoryDay`, `HistoryResponse`, `PutHistoryNoteRequest`, and `HistoryNoteResponse`
- Document day statuses (`in_progress`, `success`, `missed`, `no_tasks`, `unknown`), provisional current-day results, and alignment with existing recurring-task progress and 00:05 local reset rules

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
