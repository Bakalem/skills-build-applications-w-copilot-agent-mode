# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Keep API access in the shared frontend client and let each route component own its loading, success, empty, and error states. Render API data with accessible headings and status/alert announcements; do not put business rules or persistence in React.
- Logic tier decision: Keep HTTP routing in Express routes and validation/business rules in the service layer. Return deliberate 4xx responses for invalid input, invalid or missing references, and unknown users; reserve the error middleware's 500 response for unexpected failures. Do not add authentication in this exercise.
- Data tier decision: Persist application data through Mongoose models. Require meaningful fields and appropriate types, give teams an empty members array by default, and keep the seed idempotent without deleting unrelated records.
- Activity tracking approach: Validate the request shape and ObjectId format, confirm both referenced user and team exist, and reject invalid requests before writing. Store the references, activity type, and minutes with computed points and an occurrence timestamp; return the persisted record from create and list operations.
- Team management approach: Require a non-empty name, preserve the submitted name and mascot, and persist the supplied member IDs. Return created and listed teams with their members.
- Leaderboard approach: Sum activity points by team, include each team's name, sort by total points descending, and assign ranks starting at 1.
- Workout suggestions approach: Find the user by email, consider their fitness goal and recent activities when ordering available workouts, and include a concise reason for the leading suggestion. Return a deliberate 4xx response when no user matches.

## Test plan

Run the focused contract tests in dependency order, then run both builds. The starter contracts are intentionally red until their capabilities are implemented; do not change protected contract tests or scripts to make them pass.

- Data tier: `npm run test:data --prefix octofit-tracker/backend` checks required model fields, defaults, and repeatable seed behavior without losing unrelated records. A failure points to schema constraints or seed persistence/linking.
- API logic: `npm run test:api --prefix octofit-tracker/backend` checks activity and team creation/validation, leaderboard aggregation and ranking, and user-specific workout suggestions. A failure points to service behavior, HTTP status handling, or response shape.
- Presentation: `npm test --prefix octofit-tracker/frontend` checks API-backed rendering, the workout request, and accessible loading, empty, and error states for each view. A failure points to frontend data fetching, rendering, or state announcements.
- Backend build: `npm run build --prefix octofit-tracker/backend` runs the TypeScript compiler. A failure indicates backend type errors, including mismatches between route, service, and model contracts.
- Frontend build: `npm run build --prefix octofit-tracker/frontend` runs TypeScript checking and the Vite production build. A failure indicates frontend type or bundling errors.
