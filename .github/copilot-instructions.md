### Repo overview

- This is a tiny FastAPI app that serves an in-memory "Mergington High School Activities" API plus a small static single-page UI.
- Key files: `src/app.py` (API + app object), `src/static/index.html` and `src/static/app.js` (client), `requirements.txt` (runtime deps).

### Big picture / architecture

- Single-process service: FastAPI `app` defined in `src/app.py`. Static assets are mounted at `/static` and the root endpoint redirects to `/static/index.html`.
- Data is stored in-memory in the `activities` dict inside `src/app.py`. There is no persistent storage — expect state to reset on server restart.
- API surface (examples):
  - `GET /activities` — returns the `activities` dict (see `src/app.py`).
  - `POST /activities/{activity_name}/signup?email=...` — appends an email to the `participants` list for the named activity. Note: activity names include spaces and must be URL-encoded.

### Developer workflows (how to run & debug)

- Install dependencies (recommended virtualenv):

```bash
python -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

- Run the app with Uvicorn (recommended):

```bash
uvicorn src.app:app --reload --port 8000
```

- Open these URLs while the server runs:
  - API docs: `http://localhost:8000/docs` (Swagger)
  - Static UI: `http://localhost:8000/` (redirects to `/static/index.html`)

- Quick curl example for signup (note URL-encoding of activity name):

```bash
curl -X POST "http://localhost:8000/activities/Programming%20Class/signup?email=student%40mergington.edu"
```

### Project-specific conventions & gotchas

- Activity identifier: the project uses the human-readable activity name (e.g. `Programming Class`) as the primary key. This means path parameters will contain spaces — client code uses `encodeURIComponent` in `src/static/app.js`.
- In-memory model: updates mutate `activities` directly (append to `participants`). There are no checks for duplicates or max capacity and no persistence.
- Static files are mounted relative to `src/` (see the `StaticFiles(directory=...)` call in `src/app.py`) — move or rename the `static` directory carefully.

### Patterns an AI agent should follow when editing code

- Small, focused edits: modify `src/app.py` for API changes and `src/static/*` for UI changes. Keep the `activities` shape consistent (dict keyed by activity name mapping to description, schedule, max_participants, participants).
- When adding third-party packages, update `requirements.txt`.
- Prefer adding minimal tests (pytest is present via `pytest.ini`) for any non-trivial logic, but note the repo currently has no test files to inspect.

### Integration points & external dependencies

- Runtime: `fastapi`, `uvicorn` (listed in `requirements.txt`). No external databases, authentication, or external APIs are used.

### Files to inspect for context when making changes

- `src/app.py` — authoritative source of the API surface and in-memory model.
- `src/static/app.js` — client-side behavior: fetch `/activities`, call POST `/activities/{name}/signup`.
- `src/static/index.html` — the simple UI and form structure; useful when changing field names or validation.
- `src/README.md` — quick developer notes and expected endpoints.

If anything above is unclear or you'd like more details (tests, CI workflow, or changing the in-memory model to a database), tell me which area to expand and I'll iterate.
