# Todo App Test Cases

This document is a cleaner, table-driven version for the Todo app in this workspace.

## Application Scope

- Backend: Node/Express API in `backend/src/index.js`
- Frontend: React + Vite app in `frontend/src/App.jsx` and `frontend/src/main.jsx`
- Core API endpoints:
  - `GET /tasks`
  - `POST /tasks`
  - `PATCH /tasks/:id/complete`
- Core user flow:
  - create a task
  - view the task in the list
  - toggle completion state

## Test Environment

- Backend URL: `http://localhost:4000`
- Frontend URL: `http://localhost:5173`
- Backend storage: in-memory array
- Reset behavior: restarting the backend clears tasks

## Setup Notes

- Use unique titles such as `QA Task <timestamp>`.
- Restart the backend before order-sensitive checks.
- Use `curl`, Postman, or REST Client for API validation.
- Use Jest, Supertest, Playwright or Cypress, k6 or Artillery, and OWASP ZAP where automation is available.

## Backend API

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Accept valid task payload | Positive | High | Backend unit | Validate that a payload with a non-empty title is accepted. | Validation returns no error and the payload is treated as valid. |
| Reject missing title | Negative | High | Backend unit | Validate that a payload without `title` is rejected. | Validation returns `Task title is required.` |
| Reject whitespace-only title | Negative | High | Backend unit | Validate that a title containing only spaces is rejected. | Validation returns `Task title is required.` |
| Reject non-string title values | Negative | High | Backend unit | Validate that invalid title types are rejected. | Each invalid value is rejected with `Task title is required.` |
| POST task and verify with GET | Positive | High | Backend integration | Create a task through `POST /tasks` and confirm it appears in `GET /tasks`. | `POST` returns `201`; `GET` returns `200`; created task is present with `completed: false`. |
| POST task with minimal payload | Positive | High | Backend integration | Create a task with title only and confirm default field handling. | Task is created; `notes` is `""`; `due` is `null`; `completed` is `false`. |
| Reject create request with missing title | Negative | High | Backend integration | Submit `POST /tasks` without a title. | Response status is `400` and body is `{ "error": "Task title is required." }`. |
| Toggle completion for existing task | Positive | High | Backend integration | Create a task and toggle it through `PATCH /tasks/:id/complete`. | `PATCH` returns `200` and the task changes to `completed: true`. |
| Return 404 for missing task toggle | Negative | High | Backend integration | Call the toggle endpoint with an unknown task ID. | Response status is `404` and body is `{ "error": "Task not found." }`. |
| GET /tasks returns JSON array | Positive | Medium | Backend API | Confirm the list endpoint returns a JSON array. | Status is `200` and the body is an array. |
| POST /tasks returns task schema | Positive | High | Backend API | Confirm a successful create returns the expected fields. | Response includes `id`, `title`, `notes`, `due`, `completed`, and `createdAt`. |
| PATCH /tasks/:id/complete returns 404 | Negative | High | Backend API | Confirm the completion endpoint handles unknown IDs cleanly. | Status is `404` and the error message says `Task not found.` |
| POST /tasks rejects invalid payload | Negative | High | Backend API | Confirm invalid create requests are rejected by the API contract. | Status is `400` and the response explains that title is required. |
| Verify generic error handling | Negative | Medium | Backend API | Force an unexpected backend exception. | The API returns `500` with a generic message and no stack trace leak. |
| Verify CORS header is present | Positive | Medium | Backend API | Confirm cross-origin requests are permitted as configured. | `access-control-allow-origin: *` is present on the response. |

## Frontend UI

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Load empty task list | Positive | High | Frontend unit | Render the UI when the backend returns no tasks. | The empty state is shown and no task cards are rendered. |
| Load existing tasks | Positive | High | Frontend unit | Render the UI with a populated task list. | All returned tasks are displayed correctly. |
| Toggle completion from false to true | Positive | High | Frontend unit | Update local task state when a task is marked complete. | The targeted task changes to `completed: true`. |
| Toggle completion from true to false | Positive | Medium | Frontend unit | Update local task state when a completed task is marked undone. | The targeted task changes to `completed: false`. |
| Toggle affects only targeted task | Positive | High | Frontend unit | Confirm the toggle action does not mutate other tasks. | Only the selected task changes state. |
| Required title field blocks empty submit | Negative | High | Frontend UI | Verify the title field prevents empty submission. | Submission is blocked and no success state is shown. |
| Form clears after successful save | Positive | High | Frontend UI | Confirm the form resets after a successful create action. | Title, due date, and notes fields are cleared and success feedback appears. |
| Error message appears on backend failure | Negative | High | Frontend UI | Simulate a failed save request. | Error feedback is rendered and the page remains usable. |
| Completion button label changes | Positive | Medium | Frontend UI | Toggle a task and inspect the action button text. | The button changes from `Mark complete` to `Mark undone`. |
| Render notes safely | Negative | High | Frontend UI | Display task content containing HTML-like text. | The content renders as plain text and does not execute as HTML. |
| Render long task text | Positive | Medium | Frontend UI | Submit a very long title or notes field. | The UI remains usable and the layout does not break. |
| Preserve loading state | Positive | Medium | Frontend UI | Submit a task while the request is in progress. | The submit button is disabled and shows a saving state until completion. |

## Integration

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Create task through UI and verify list update | Positive | High | Frontend integration | Use the browser form to create a task and confirm it appears in the list. | Success feedback appears and the new task is shown with entered details. |
| Toggle completion through UI | Positive | High | Frontend integration | Use the browser to toggle a task and observe the updated state. | The card changes styling and the button label updates. |
| Handle API failure on load | Negative | High | Frontend integration | Simulate backend unavailability during initial load. | The frontend shows an error message and does not crash. |
| Handle API failure on submit | Negative | High | Frontend integration | Simulate a server error while creating a task. | A readable error is shown and the task is not added locally. |
| Create task and verify order | Positive | High | Frontend integration | Create multiple tasks from the UI. | The newest task appears at the top of the list. |
| Verify response shape end to end | Positive | Medium | Integration | Observe data flow from create and toggle operations. | The UI receives the expected task object fields and updates state correctly. |

## End-to-End

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Complete the full add-task journey | Positive | High | E2E | Create a task from the UI and confirm it is persisted in the backend state. | The user sees success feedback and the task is visible in the list. |
| Complete the full toggle journey | Positive | High | E2E | Mark a task complete and then undo it. | The task transitions correctly between complete and incomplete states. |
| Prevent empty title submission from UI | Negative | High | E2E | Attempt to submit the form with an empty title. | Submission is blocked and no task is created. |
| Show validation feedback to end user | Negative | High | E2E | Submit invalid data and observe the result. | The app shows a validation message and keeps the form usable. |
| Recover after backend error | Negative | Medium | E2E | Trigger a backend failure and then retry with a valid action. | The app surfaces the error and still allows a later successful action. |

## Performance

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| GET /tasks responds within threshold | Positive | Medium | Performance | Send repeated `GET /tasks` requests and measure response time. | Error rate stays at `0%` and latency remains within the agreed threshold. |
| Repeated POST requests remain stable | Positive | Medium | Performance | Send many valid create requests with unique titles. | All valid requests return `201` and the server remains responsive. |
| Large payload does not crash service | Negative | Medium | Performance | Submit a request with very large notes content. | The service responds in a controlled way and follow-up requests still work. |
| Moderate list size remains responsive | Positive | Medium | Performance | Create and fetch a moderate number of tasks repeatedly. | The app stays responsive and task rendering remains stable. |

## Security

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Reject empty and whitespace titles | Negative | High | Security | Submit invalid title input such as empty strings and spaces. | Both requests return `400` and no invalid task is created. |
| Handle script-like title safely | Negative | High | Security | Create a task with script-like content in the title. | The content renders as text and no script executes. |
| Handle script-like notes safely | Negative | High | Security | Store HTML-like content in the notes field. | The UI renders plain text only and remains safe. |
| Verify CORS behavior | Positive | Medium | Security | Confirm the backend cross-origin policy behaves as expected. | The configured frontend origin can call the API successfully. |
| Avoid leaking internal errors | Negative | High | Security | Trigger an unexpected backend exception. | The client receives a generic error and no internal stack trace. |

## Regression

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Create and toggle flow still works | Positive | High | Regression | Re-run the main create and toggle journey after changes. | The flow still works and the final state matches the last action. |
| Validation rules do not regress | Negative | High | Regression | Re-test missing, empty, and whitespace-only titles. | All invalid requests still return `400`. |
| Task ordering remains correct | Positive | Medium | Regression | Create multiple tasks and verify display order. | New tasks continue to appear at the top of the list. |
| Form clearing behavior remains correct | Positive | Medium | Regression | Create a task and confirm input reset behavior. | The form clears after successful save. |
| Status messages remain visible | Positive | Medium | Regression | Trigger success and error states in the UI. | Status messages still appear clearly and remain readable. |
| Backend status codes stay stable | Positive | High | Regression | Re-check `400`, `404`, and `500` responses after code changes. | The API continues to return the expected status codes and messages. |

## Smoke

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| Backend starts and serves GET /tasks | Positive | High | Smoke | Start the backend and call the list endpoint. | Backend starts successfully and `GET /tasks` returns `200`. |
| Frontend loads and displays form | Positive | High | Smoke | Start the frontend and open the app root. | The page renders and the task form is visible. |
| Basic task creation works | Positive | High | Smoke | Enter a valid title and submit from the UI. | The task is saved and appears in the list. |

## Acceptance

| Title | Type | Priority | Layer | Description | Expected Result |
|---|---|---:|---|---|---|
| User can create and complete a task | Positive | High | Acceptance | Verify the main business flow from create to completion. | The user can add a task and mark it complete successfully. |
| User gets feedback for invalid input | Negative | High | Acceptance | Verify the app guides the user when required data is missing. | Submission is blocked and the user receives validation feedback. |

## Execution Notes

- Capture screenshots, API responses, or console output when recording actual results.
- Keep the backend restart behavior in mind because data is not persisted.
- Prefer the backend and frontend file paths noted above when mapping automated tests.
