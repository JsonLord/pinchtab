# Agent.md

## 1. Deployment Configuration

### Target Space
- **Profile:** `AUXteam`
- **Space:** `WitNote`
- **Full Identifier:** `AUXteam/WitNote`
- **Frontend Port:** `7860` (mandatory for all Hugging Face Spaces)

### Deployment Method
Choose the correct SDK based on the app type based on the codebase language:

- **Gradio SDK** — for Gradio applications
- **Streamlit SDK** — for Streamlit applications
- **Docker SDK** — for all other applications (recommended default for flexibility)

### HF Token
- The environment variable **`HF_TOKEN` will always be provided at execution time**.
- Never hardcode the token. Always read it from the environment.
- All monitoring and log‑streaming commands rely on `$HF_TOKEN`.

### Required Files
- `Dockerfile` (or `app.py` for Gradio/Streamlit SDKs)
- `README.md` with Hugging Face YAML frontmatter:
  ```yaml
  ---
  title: <APP NAME>
  sdk: docker | gradio | streamlit
  app_port: 7860
  ---
  ```
- `.hfignore` to exclude unnecessary files
- This `Agent.md` file (must be committed before deployment)

---

## 2. API Exposure and Documentation

### Mandatory Endpoints
Every deployment **must** expose:

- **`/health`**
  - Returns HTTP 200 when the app is ready.
  - Required for Hugging Face to transition the Space from *starting* → *running*.

- **`/api-docs`**
  - Documents **all** available API endpoints.
  - Must be reachable at:
    `https://HF_PROFILE-WitNote.hf.space/api-docs`

### Functional Endpoints

### /navigate
- Method: POST
- Purpose: Navigate the current tab to a specified URL.
- Request Example:
  ```json
  {
    "url": "https://example.com"
  }
  ```
- Response Example:
  ```json
  {
    "status": "ok",
    "url": "https://example.com"
  }
  ```

### /text
- Method: GET
- Purpose: Extract structured text from the current page.
- Request Example: `?maxChars=1000&format=markdown`
- Response Example:
  ```json
  {
    "text": "Extracted text content..."
  }
  ```

### /action
- Method: POST
- Purpose: Perform a single action on the page (e.g., click, type).
- Request Example:
  ```json
  {
    "action": "click",
    "selector": "#submit-btn"
  }
  ```
- Response Example:
  ```json
  {
    "status": "ok"
  }
  ```

### /snapshot
- Method: GET
- Purpose: Get an accessibility snapshot of the current page.
- Request Example: (no body)
- Response Example:
  ```json
  {
    "snapshot": [ ... ]
  }
  ```

### /evaluate
- Method: POST
- Purpose: Run JavaScript in the current tab.
- Request Example:
  ```json
  {
    "expression": "document.title"
  }
  ```
- Response Example:
  ```json
  {
    "result": "Example Domain"
  }
  ```

### /macro
- Method: POST
- Purpose: Execute a macro action pipeline.
- Request Example:
  ```json
  {
    "actions": [
      {"action": "click", "selector": "#btn"}
    ]
  }
  ```
- Response Example:
  ```json
  {
    "status": "ok"
  }
  ```

All endpoints listed here **must** appear in `/api-docs`.

---

## 3. Deployment Workflow

### Standard Deployment Command
After any code change, run:

```bash
hf upload AUXteam/WitNote --repo-type=space
```

This command must be executed **after updating and committing Agent.md**.

### Deployment Steps
1. Ensure all code changes are committed.
2. Ensure `Agent.md` is updated and committed.
3. Run the upload command.
4. Wait for the Space to build.
5. Monitor logs (see next section).
6. When the Space is running, execute all test cases.

### Continuous Deployment Rule
After **every** relevant edit (logic, dependencies, API changes):

- Update `Agent.md`
- Redeploy using the upload command
- Re-run all test cases
- Confirm `/health` and `/api-docs` are functional

This applies even for long-running projects.

---

## 4. Monitoring and Logs

### Build Logs (SSE)
```bash
curl -N \
  -H "Authorization: Bearer $HF_TOKEN" \
  "https://huggingface.co/api/spaces/AUXteam/WitNote/logs/build"
```

### Run Logs (SSE)
```bash
curl -N \
  -H "Authorization: Bearer $HF_TOKEN" \
  "https://huggingface.co/api/spaces/AUXteam/WitNote/logs/run"
```

### Notes
- If the Space stays in *starting* for too long, `/health` is usually failing.
- If the Space times out after ~30 minutes, check logs immediately.
- Fix issues, commit changes, redeploy.

---

## 5. Test Run Cases (Mandatory After Every Deployment)

These tests ensure the agentic system can verify the deployment automatically.

### 1. Health Check
```
GET https://HF_PROFILE-WitNote.hf.space/health
Expected: HTTP 200, body: {"status": "ok"} or similar
```

### 2. API Docs Check
```
GET https://HF_PROFILE-WitNote.hf.space/api-docs
Expected: HTTP 200, valid documentation UI or JSON spec
```

### 3. Functional Endpoint Tests
For each endpoint documented above, define:

- Example request
- Expected response structure
- Validation criteria (e.g., non-empty output, valid JSON)

Example:

```
POST https://HF_PROFILE-WitNote.hf.space/predict
Payload:
{
  "text": "test"
}
Expected:
- HTTP 200
- JSON with key "prediction"
- No error fields
```

### 4. End-to-End Behaviour
- Confirm the UI loads (if applicable)
- Confirm API endpoints respond within reasonable time
- Confirm no errors appear in run logs

---

## 6. Maintenance Rules

- `Agent.md` must always reflect the **current** deployment configuration, API surface, and test cases.
- Any change to:
  - API routes
  - Dockerfile
  - Dependencies
  - App logic
  - Deployment method
  requires updating this file.
- This file must be committed **before** every deployment.
- This file is the operational contract for autonomous agents interacting with the project.
