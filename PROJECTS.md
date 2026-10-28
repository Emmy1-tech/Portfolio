# Project engineering notes

A guide for reviewing the front-end, back-end, and full-stack work behind Emmy1-Tech’s portfolio.

## Portfolio links

- **Original live portfolio:** https://emmy1-tech.github.io/Portfolio/
- **Additional live portfolio:** https://emmy-portfolio-dev.epiclamb21.chatgpt.site/
- **GitHub profile:** https://github.com/Emmy1-tech
- **Portfolio source:** https://github.com/Emmy1-tech/Portfolio

Both portfolio sites are retained. Project implementation evidence below comes from the linked repositories, rather than from illustrative dashboard graphics.

## Role focus and review guide

I’m interested in junior front-end, back-end, full-stack, and internship opportunities. My project work focuses on turning a user workflow into an understandable interface, connecting it to an API, validating data, and documenting the behavior and remaining work.

| Area | Evidence to review |
| --- | --- |
| Front-end interfaces | React and TypeScript components, endpoint forms, loading/empty/error states, search, filters, and incident actions in CloudPulse; browser-native UI in AI Career Copilot |
| API integration | A typed Axios service layer in CloudPulse; JSON requests between the Career Copilot browser UI and Express API |
| Back-end design | FastAPI routers and Pydantic schemas; Express routes, client-error responses, provider integration, and a separate storage adapter |
| Data modeling and persistence | SQLAlchemy models for endpoints, results, and incidents; a queued JSON persistence adapter for local application records |
| Testing | UI state tests, API lifecycle and validation tests, isolated database fixtures, and external-provider failure scenarios |
| Delivery | GitHub Actions configurations, build commands, dependency audits, Docker Compose smoke checks, and deployment instructions |
| Technical judgment | Distinct demo/local modes, labelled sample data, server-side secrets, clear trade-offs, and explicit production-readiness gaps |

These are portfolio projects, not claims of professional production experience. Test coverage percentages, commercial outcomes, user counts, and performance scores are not claimed.

## CloudPulse

**Repository:** https://github.com/Emmy1-tech/CloudPulse  
**Browser demo:** https://emmy1-tech.github.io/CloudPulse/  
**Status:** In development; the public UI demonstrates simulated monitoring workflows.

### Problem and observable result

CloudPulse makes endpoint inventory, check history, and incident handling visible in one dashboard. A reviewer can follow a sample endpoint from registration through a simulated check result and an incident status change. The result is a concrete end-to-end workflow to inspect, rather than a claim about real service uptime.

### Stack and architecture

| Layer | Implementation |
| --- | --- |
| Browser | React, TypeScript, CSS, typed domain objects, and dashboard/form components |
| API client | Axios instance configured with `VITE_API_URL`; a demo adapter selected with `VITE_DEMO_MODE` |
| HTTP API | FastAPI routers and Pydantic request/response schemas |
| Persistence | SQLAlchemy ORM and request-scoped sessions with SQLite |
| Quality/delivery | Python tests, front-end tests, GitHub Actions, dependency audits, Docker Compose, and a Pages demo build |

In local backend-connected mode, UI actions call the shared API layer, FastAPI validates the payload, and SQLAlchemy stores the result. Endpoint records, monitoring results, and incidents are separate models. In the hosted front-end demo, the same API-facing interface selects browser-local sample behavior instead of exposing the local FastAPI service.

### Front-end implementation

- State for endpoint records, incidents, recent results, loading, errors, form visibility, and the selected endpoint.
- Search by endpoint information and filtering by active/inactive status.
- Endpoint creation/editing workflows and a details view with recorded results and statistics.
- Summary cards, response-history visualization, incident status badges, and acknowledge/resolve actions.
- A visible loading state, empty-state copy, and an alert banner for workspace loading failures.
- Typed API methods separate HTTP calls from the component’s presentation.

Review the [dashboard component](https://github.com/Emmy1-tech/CloudPulse/blob/main/frontend/src/components/Dashboard.tsx), [API client](https://github.com/Emmy1-tech/CloudPulse/blob/main/frontend/src/services/api.ts), [demo adapter](https://github.com/Emmy1-tech/CloudPulse/blob/main/frontend/src/services/demoApi.ts), and [UI tests](https://github.com/Emmy1-tech/CloudPulse/blob/main/frontend/src/test/Dashboard.test.tsx).

The UI tests include initial loading, empty endpoint data, incident loading through the configured client, the Add Endpoint control, and opening its form. Accessibility and performance remain areas to assess with keyboard testing, assistive technology, and measurements; a test suite alone is not an accessibility certification.

### Back-end implementation

- Create, list, retrieve, update, and delete endpoint records.
- Trigger a manual simulated check with a forced success or failure for a repeatable demonstration.
- Persist check history and derive statistics from recorded simulated outcomes.
- Create an incident after a failure when there is no existing open or acknowledged incident for the endpoint.
- Acknowledge or resolve incidents and record resolution time.
- Return missing-record errors and expose generated API documentation.
- Perform a database query in the health endpoint to distinguish database readiness from a simple process response.

Review the [endpoint routes](https://github.com/Emmy1-tech/CloudPulse/blob/main/backend/app/api/endpoints.py), [monitoring and incident routes](https://github.com/Emmy1-tech/CloudPulse/blob/main/backend/app/api/monitoring.py), [schemas](https://github.com/Emmy1-tech/CloudPulse/blob/main/backend/app/schemas.py), [models](https://github.com/Emmy1-tech/CloudPulse/blob/main/backend/app/models.py), and [database setup](https://github.com/Emmy1-tech/CloudPulse/blob/main/backend/app/database.py).

### Technical decisions and trade-offs

1. **Demo adapter versus local API:** the public interface can be explored with sample data without publishing an API that lacks authentication.
2. **SQLite for a reproducible demonstration:** simple setup and inspectable data, with production migrations, backups, and multi-instance operation left for later work.
3. **Separate incident records:** the workflow can show an incident’s lifecycle independently from its check history. Repeated failures do not intentionally create duplicate active incidents in the sequential demonstration.
4. **Simulated checks:** repeatable success/failure scenarios help explain the workflow, while requiring clear labels so the results are not mistaken for real availability.

### Tests and delivery evidence

The [backend tests](https://github.com/Emmy1-tech/CloudPulse/tree/main/backend/tests) cover endpoint behavior, monitoring, health, database setup, configuration, and security headers. Isolated database fixtures make the API tests independent of an existing local demo database.

The [CI configuration](https://github.com/Emmy1-tech/CloudPulse/blob/main/.github/workflows/ci.yml) defines backend tests, schema initialization, Python dependency auditing, front-end tests/lint/builds, high/critical npm auditing, a public-demo build, and a Docker Compose smoke check.

Use the [latest workflow runs](https://github.com/Emmy1-tech/CloudPulse/actions) for current pass/fail evidence. The presence of these jobs does not establish production readiness or guarantee that every future run passes.

After following the repository’s environment setup:

```bash
# From backend/
python -m pytest tests/ -v

# From frontend/
npm ci
npm test -- --run
npm run lint
npm run build
npm run audit:security
```

### Suggested review walkthrough

1. Open the browser demo and inspect the sample endpoints.
2. Create or select a sample endpoint.
3. Simulate a failure and inspect its recorded result and incident.
4. Acknowledge or resolve the incident and review the displayed state.
5. Inspect the matching API methods, persistence models, and tests in the repository.

### Current limits and next improvements

Checks are simulated and do not contact configured endpoints. Stored intervals do not drive an automatic scheduler. The local API has no authentication, authorization, or per-user data isolation. SQLite is a demo storage choice, not a complete production data strategy.

Next priorities include authenticated ownership, real endpoint execution with appropriate network controls, background scheduling, migrations, durable deployment design, stronger operational visibility, and broader user-flow tests. These are future improvements, not completed features.

**Further reading:** [Architecture](https://github.com/Emmy1-tech/CloudPulse/blob/main/ARCHITECTURE.md) · [Engineering review](https://github.com/Emmy1-tech/CloudPulse/blob/main/ENGINEERING_REVIEW.md) · [Setup and scope](https://github.com/Emmy1-tech/CloudPulse#readme) · [Deployment](https://github.com/Emmy1-tech/CloudPulse/blob/main/DEPLOYMENT.md)

## AI Career Copilot

**Repository:** https://github.com/Emmy1-tech/ai-career-copilot  
**Hosted demo:** https://ai-career-copilot-emmy.onrender.com  
**Status:** Portfolio application with optional AI configuration and documented storage/authentication limits.

### Problem and observable result

Job-search tasks are often scattered across several tools. This project brings resume feedback, application tracking, job-description comparison, interview practice, and an assistant interface into a shared workspace.

A reviewer can inspect how a browser action becomes an API request, how an application record changes status, and how the interface distinguishes sample or rules-based output from configured AI features.

### Stack and architecture

| Layer | Implementation |
| --- | --- |
| Browser | HTML, CSS, and browser-native JavaScript |
| Server | Node.js and Express |
| Data | Local JSON application store through a separate adapter |
| AI integration | Optional server-side Gemini calls |
| Verification | Node API tests and a GitHub Actions matrix for Node.js 20 and 22 |

The [browser application](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/public/app.js) sends JSON requests to the [Express server](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/server.js). The server validates inputs, calls the storage adapter or configured AI provider, and returns structured responses.

### Front-end workflows

- Dashboard summaries and recent application information.
- Application creation, status updates, listing, and deletion.
- Resume analysis and comparison with a pasted job description.
- Interview question practice with STAR-style prompts.
- Sample job listings and external job-search links.
- An assistant interface whose behavior depends on server-side AI configuration.

The project uses native browser code, making DOM behavior and API calls straightforward to inspect. Review the [HTML structure](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/public/index.html), [styles](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/public/styles.css), and [client code](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/public/app.js).

### Back-end and reliability details

- JSON API routes for dashboard data, applications, resume workflows, assistant responses, and a bounded plan-only career checklist.
- Validation for required fields, application statuses, resume/job-description input, and job URLs.
- Consistent JSON errors for malformed bodies, oversized requests, invalid input, and unknown routes.
- Configurable per-process limits on AI endpoints.
- Provider error and timeout handling that avoids returning raw provider failure details.
- AI credentials read on the server rather than embedded in browser code.
- Clearly labelled rules-based feedback or unavailable-configuration responses when the relevant AI feature is not configured.

The [storage adapter](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/src/store.js) queues local mutations so overlapping requests within one process do not perform competing read-modify-write operations. It writes a unique temporary file and renames it into place. This is a useful local reliability measure, not a substitute for a transactional multi-process database.

### Tests and reproducibility

The [API test file](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/tests/server.test.js) includes:

- Application create/list/update/delete behavior.
- Missing fields, unsafe job URLs, invalid statuses, and absent records.
- Malformed and oversized JSON.
- Missing AI configuration and labelled fallback responses.
- Mocked provider errors, timeouts, and empty responses.
- AI request limiting and response security headers.
- Concurrent local writes retaining application records.

The [CI workflow](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/.github/workflows/ci.yml) runs JavaScript syntax checks and API tests on Node.js 20 and 22. Check [current Actions results](https://github.com/Emmy1-tech/ai-career-copilot/actions) for the latest status.

After installing dependencies using the [setup instructions](https://github.com/Emmy1-tech/ai-career-copilot#run-locally-for-development):

```bash
npm run check
npm test
```

### Suggested review walkthrough

Use fictional information in the hosted demo.

1. Create a sample job application and change its status.
2. Follow the client request to its Express route and storage operation.
3. Inspect a validation error and compare it with the corresponding test.
4. Review the resume workflow’s fallback behavior when AI is not configured.
5. Inspect provider error/timeout tests and the queued-write test.

### Current limits and next improvements

Job listings are sample data, not verified vacancies. AI features require the appropriate server configuration. The current hosted application has no sign-in or per-user data isolation, and its JSON-file persistence is not durable on the current hosting setup.

Next priorities are authentication and ownership, a managed relational database, verified job data, distributed rate limits, stronger operational logging, durable hosting, and broader deployed-host tests. The existing planner returns a checklist; it does not execute external actions.

**Further reading:** [Architecture](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/docs/wiki/architecture.md) · [Security and quality](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/docs/SECURITY-QUALITY.md) · [Deployment](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/DEPLOYMENT.md) · [Local setup](https://github.com/Emmy1-tech/ai-career-copilot/blob/main/LOCAL-SETUP.md)

## Portfolio implementation

The original GitHub Pages portfolio uses HTML, CSS, and a small JavaScript footer enhancement. Its existing navigation, GitHub links, project content, and live address are retained. The additional portfolio is a separate live site with Home, About, and Now pages.

The portfolio’s responsive CSS, visible keyboard focus states, and reduced-motion handling can be inspected directly in [index.html](./index.html). Further work can include a measured accessibility audit, real-device coverage, and performance checks. No audit score or performance metric is claimed without measured evidence.

## Further discussion

Useful implementation topics to discuss alongside the source:

- How a front-end loading, empty, or failure state changes a user’s next action.
- Why a shared typed API layer makes changing transport or demo behavior easier.
- The difference between validation, authentication, and authorization.
- What a process-local operation queue protects, and where a relational database is needed.
- How to test a provider outage or timeout without depending on a live AI service.
- How to distinguish a reproducible portfolio demonstration from a production deployment.

For my public development work and ways to follow up, visit [my GitHub profile](https://github.com/Emmy1-tech).
