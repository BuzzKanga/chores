# Shared Household Chore Manager — Task Backlog

## 1. Project bootstrap with passing test
Goal: Create a runnable Django project skeleton with one passing smoke test.
Description: Initialise a new Django project and app (e.g. `chores`) inside a virtual-environment-friendly layout. Add a single test that asserts the Django test runner executes successfully (e.g. a trivial `assertTrue(True)`). Confirm `python manage.py test` exits green with no warnings.

---

## 2. Data models — Household and Member
Goal: Define and migrate the `Household` and `HouseholdMember` models.
Description: Create a `Household` model (name, created_at, invite_code UUID) and a `HouseholdMember` join model linking Django's built-in `User` to a `Household` with an `is_admin` flag. Write and run the initial migration. Add model-level unit tests covering field defaults and the `__str__` representation.

---

## 3. Data models — Chore and ChoreCycle
Goal: Define and migrate the `Chore` and `ChoreCycle` models.
Description: Create a `Chore` model (household FK, title, description, frequency, cycle reset weekday/time, ordered rotation sequence, is_archived flag) and a `ChoreCycle` model that records each generated cycle instance (chore FK, assignee FK, period start/end, status: active/overdue/completed). Write unit tests covering the rotation-sequence ordering and status field choices.

---

## 4. User registration and login
Goal: Allow a new user to register an account and log in via standard Django auth views.
Description: Wire up Django's built-in `UserCreationForm` and `AuthenticationForm` to `/register/` and `/login/` URL routes backed by simple Django templates. Redirect a newly registered user to a "no household yet" placeholder page. Write integration tests using Django's test client to cover the happy path and duplicate-username error.

---

## 5. Household creation and invite code generation
Goal: Let an authenticated user create a household and receive a shareable join link.
Description: Build a `CreateHouseholdView` that creates a `Household`, promotes the creator to admin via `HouseholdMember`, and displays the unique invite URL (e.g. `/join/<invite_code>/`). The invite code must be a UUID generated on household creation. Write tests covering creation, admin flag, and invite URL rendering.

---

## 6. Household join via invite link
Goal: Let an authenticated user join an existing household by visiting its invite link.
Description: Build a `JoinHouseholdView` that looks up the household by UUID, creates a `HouseholdMember` record for the current user, and redirects to the dashboard. Return a 404 for unknown codes and a message if the user is already a member. Cover both cases in integration tests.

---

## 7. Chore creation and management (admin only)
Goal: Let the household admin create, edit, and archive chores through a form-based UI.
Description: Build create/edit/archive views for `Chore` restricted to `is_admin` members of the household. Each form should capture title, description, frequency, reset weekday, reset time, and rotation sequence (ordered list of household members). Write tests asserting that non-admin members receive a 403 and that all CRUD operations persist correctly.

---

## 8. Rotation logic — cycle generation service
Goal: Implement a pure Python service that generates the next `ChoreCycle` for a chore.
Description: Write a `generate_next_cycle(chore)` function that reads the chore's current rotation position, selects the correct assignee, sets `period_start` and `period_end` from the reset schedule, and creates a `ChoreCycle` record. The function must also mark the previous cycle overdue if it was not completed. Cover the rollover and rotation-advancement logic with unit tests, no HTTP layer involved.

---

## 9. Scheduled cycle rotation with Django-Q (or Celery)
Goal: Execute `generate_next_cycle` automatically on each chore's configured schedule.
Description: Add and configure a background worker (Django-Q or Celery + django-celery-beat). Register a scheduled task that iterates active chores and calls `generate_next_cycle` when the cycle reset time is reached. Verify via a management command (`python manage.py run_cycle_rotation`) that cycles are generated and the worker picks up tasks. Document setup steps in a brief `README` section.

---

## 10. Personal dashboard — "My Chores"
Goal: Show the logged-in member their active and overdue chores for the current cycle.
Description: Build a `DashboardView` that queries `ChoreCycle` for the current user's active and overdue assignments and renders them in a Django template. Each row must show chore name, due date, status, and a "Mark complete" button. Write integration tests confirming that only the current user's cycles appear and that overdue items are visually distinguished.

---

## 11. Mark chore complete
Goal: Let a member mark one of their assigned chores complete with a single click.
Description: Add a `CompleteChoreView` (POST-only) that sets `ChoreCycle.status = 'completed'` and records a `completed_at` timestamp. Reject attempts by users who are not the assignee with a 403. Redirect back to the dashboard on success. Cover the happy path, wrong-user rejection, and already-completed idempotency in tests.

---

## 12. Household overview tab
Goal: Show all members' chore statuses for the current cycle on a shared board.
Description: Add a second tab (or sub-page) to the dashboard that lists every `ChoreCycle` for the household's current period grouped by member, with status badges (Completed / Incomplete / Overdue). Any household member can view this page; it is read-only. Write integration tests confirming all members' data is present and correctly labelled.

---

## 13. Household settings page and member management
Goal: Give the admin a settings page to manage members and the invite link.
Description: Build a `HouseholdSettingsView` showing the current invite link with a copy button, a list of members, and a "Remove" action (admin only) that deletes the `HouseholdMember` record. Regenerating the invite code (invalidating the old link) should also be available. Write tests covering removal, invite regeneration, and that non-admins cannot access admin actions.

---

## 14. Web push notification subscription
Goal: Let members subscribe to browser push notifications from the settings page.
Description: Integrate the Web Push API (using `django-push-notifications` or equivalent). Add a VAPID key pair to settings and a JavaScript snippet that requests notification permission and registers the push subscription. Store the subscription endpoint in the database linked to the user. Write a test or management command that sends a test push to confirm the pipeline works end-to-end.

---

## 15. Send push notifications on cycle rotation
Goal: Dispatch a push notification to each new assignee when a cycle rotates.
Description: Extend `generate_next_cycle` (or the scheduler task from Task 9) to call `send_push_notification(user, message)` after creating each new cycle. The notification payload should include the chore name and due date. Write a unit test using mocks to verify that the notification function is called with the correct arguments and that failures do not abort the rotation.

---

## 16. Basic UI polish and responsive layout
Goal: Apply a consistent, mobile-friendly stylesheet across all pages.
Description: Create a base Django template (`base.html`) with a responsive navigation bar, a simple color palette, and utility classes covering layout, buttons, badges, and form controls. Apply it to all existing views. No JavaScript framework is required; plain CSS is sufficient. Manually verify the dashboard, overview, and settings pages render correctly on a narrow viewport.

---

## 17. End-to-end smoke test of the full MVP flow
Goal: Validate the complete user journey from household creation through automated cycle rotation.
Description: Write a Django integration test (using the test client and `freezegun` or similar for time travel) that: registers two users; the first creates a household and invites the second; the admin creates a chore with both users in rotation; the cycle rotation function runs and creates a cycle for user 1; user 1 marks it complete; a second rotation creates a cycle for user 2. Assert each state transition produces the correct `ChoreCycle` records and statuses.
