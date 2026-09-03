# Shared Household Chore Manager — MVP Scope

## Product goal

Help flatmates transparently distribute, track, and rotate recurring household chores on automatic schedules, minimizing shared-house friction through clear individual accountability.

## Core workflow

### 1. Household setup and onboarding
An initial user registers an account, creates a household profile, and generates a unique invite link or join code. Housemates use the link to register their individual accounts and join the household.

### 2. Chore configuration
The household creator defines custom chores by specifying:
- Chore title and description
- Frequency and cycle reset day/time (e.g., weekly on Mondays at 00:00)
- Assigned rotation sequence (ordered list of members)

### 3. Automated cycle execution
At the scheduled interval, the system:
- Rotates task assignments to the next member in the configured sequence
- Carries forward any uncompleted tasks from the previous cycle to the original assignee as overdue items
- Dispatches web push notifications alerting members of new cycle assignments

### 4. Personal task execution and tracking
Members access their personal-first dashboard to:
- View assigned tasks for the active cycle
- Review overdue items carried over from prior cycles
- Mark chores complete with a single click (trust-based system)
- Toggle to a household overview tab to view overall housemate progress

## Decisions made for the MVP

### Time-based automatic cycles with rollover
**Decision:** Chores rotate automatically at a scheduled time (e.g., weekly). Uncompleted tasks remain with the original assignee as overdue items rather than passing to the next person.

**Why:** Prevents members from avoiding chores by letting cycles lapse, avoiding unfair double-duty for the next person in rotation.

**Alternatives considered:**
- Passing unfinished chores to the next assignee: Creates resentment and uneven workloads.
- Expiring missed chores: Removes accountability and encourages neglect.

### Trust-based completion
**Decision:** Marking a chore complete requires a single click with no verification steps.

**Why:** Keeps operational friction low and treats flatmates as adults, avoiding excessive administrative overhead.

**Alternatives considered:**
- Photo proof: Unnecessary friction and hosting overhead for an MVP.
- Housemate sign-off/review: Introduces bottlenecks and interpersonal policing.

### Out-of-scope chore swapping
**Decision:** Formal chore trading or task-marketplace mechanics are excluded from the MVP.

**Why:** Flatmates can agree to trade chores informally in person or via group chat. The designated assignee remains responsible in-app for checking the box.

**Alternatives considered:**
- Formal 1-to-1 swap workflows: Adds UI complexity, notification loops, and state-machine overhead for edge cases.

### Custom chore creation
**Decision:** The household admin defines chores, details, and rotation sequences manually from scratch.

**Why:** Every household has unique cleaning splits, physical layouts, and chore boundaries that fixed templates rarely capture cleanly.

**Alternatives considered:**
- Rigid static templates: Faster initial onboarding, but frustrating when predefined chores do not match household routines.

### Web push notifications
**Decision:** Deliver cycle reminders and overdue notices via standard Web Push API notifications.

**Why:** Ensures timely visibility on mobile and desktop browsers without requiring third-party transactional email delivery services or native mobile app builds.

**Alternatives considered:**
- In-app only: Low engagement; users forget to check without outbound triggers.
- Transactional email: Requires domain configuration, deliverability management, and external service costs.

### Technical architecture: Python & Django
**Decision:** Build as a monolithic Django application leveraging Django templates, standard authentication, and background workers (e.g., Celery or Django-Q) for scheduled cycle rotations and push delivery.

**Why:** Provides built-in user authentication, an ORM for relational household data, an admin panel for debugging, and a robust scheduler ecosystem.

**Alternatives considered:**
- Decoupled SPA (React/Vue + Django REST Framework): Adds API maintenance overhead and deployment complexity unnecessary for an MVP.

## Main screens

### Personal dashboard ("My Chores")
Shows:
- Overdue tasks carried over from prior cycles
- Active tasks for the current cycle with one-click completion buttons
- Timestamp of the next scheduled cycle rotation

### Household overview tab
Shows:
- All household members and their active tasks
- Status indicators (Completed, Incomplete, Overdue)
- Current rotation sequence per chore

### Chore management page (Admin)
Allows the household creator to:
- Create, edit, and archive chores
- Define rotation order among members
- Set cycle cadence (e.g., weekly day/time)

### Household settings & member list
Shows:
- Active household members
- Shareable invite link / join code
- Push notification subscription toggle

## Roles

### Household member
Can:
- Accept an invite link to join the household
- View personal tasks and the household overview board
- Mark assigned tasks complete
- Enable/disable personal web push notifications

### Household admin (Creator)
Can also:
- Generate and revoke household invite links
- Create, edit, and delete custom chores
- Define rotation order and cycle schedules
- Remove members from the household

## Explicitly excluded from the MVP

- In-app chore trading, swapping, or marketplace bidding
- Photo uploads, inspection workflows, or proof-of-work sign-offs
- Gamification, chore points, penalties, or financial allowances
- Native iOS/Android applications
- Subtasks, checklist breakdowns within a single chore, or micro-scheduling (e.g., specific hours of the day)
- Multi-household membership per single account

## Suggested success metrics

- **Task completion rate:** Percentage of assigned chores marked complete within the active cycle
- **Rollover rate:** Percentage of chores that roll over into an overdue state
- **Weekly active rate:** Percentage of household members who log in or mark a task complete weekly
- **Cycle onboarding time:** Time from household creation to all members joining and first cycle activation

## MVP definition

The MVP is successful when:
1. An admin creates a household and invites flatmates via a link.
2. The admin configures custom chores with defined rotation sequences.
3. The system executes automated time-based rotations and dispatches push notifications.
4. Members track and complete chores via a personal dashboard, with uncompleted chores rolling over automatically as overdue.

## Key scope principle

Keep this a **predictable accountability loop**, not a chore marketplace, project-management suite, or household surveillance tool.