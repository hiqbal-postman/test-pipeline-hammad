# OpenWeather API – Test Automation Pipeline

Postman collection and GitHub Actions pipeline for testing the [OpenWeather API](https://openweathermap.org/api). Week 3 practical assessment deliverable.

---

## What API and why

**API:** [OpenWeather](https://openweathermap.org/api) (Current Weather, Forecast, Geocoding).

**Why:** This is a real public API that is available quickly which also requires an API key to test out auth, and offers several simple endpoints so we can cover smoke, functional, and error-handling tests in one collection.

---

## What the tests cover

| Folder / request | What it does |
|------------------|--------------|
| **Health Check** | Smoke: GET current weather at (0,0) – validates API is up and key works. |
| **Functional Tests** | **Forecast** – 5-day/3h forecast; **Current Weather** – real-time by lat/lon (status, body shape, schema, response time); **Geocoding** – city name → lat/lon (array shape, lat/lon bounds, schema). |
| **Error Handling** | **Invalid API Key** – expects 401 and error body; **Invalid Coordinates** – lat/lon=999 → 400; **Missing Parameters** – no lat/lon → 400. |

Each request includes status checks, response body checks, and (where useful) JSON schema validation and response-time assertions.

---

## Run locally

**Prereqs:** Node.js 20+, [Newman](https://www.npmjs.com/package/newman) (`npm install -g newman`).

1. **Clone the repo**
   ```bash
   git clone <your-repo-url>
   cd openweather-api-tests
   ```

2. **Set your OpenWeather API key**  
   Edit `openweather-env.postman_environment.json` and set the `openweather_api_key` value, or override via CLI (next step).

3. **Run the collection**
   ```bash
   newman run OpenWeather-API.postman_collection.json \
     --environment openweather-env.postman_environment.json \
     --env-var "openweather_api_key=YOUR_OPENWEATHER_API_KEY"
   ```

   Optional: JUnit report
   ```bash
   mkdir -p results
   newman run OpenWeather-API.postman_collection.json \
     --environment openweather-env.postman_environment.json \
     --env-var "openweather_api_key=YOUR_OPENWEATHER_API_KEY" \
     --reporters cli,junit \
     --reporter-junit-export results/tests.xml
   ```

---

## CI/CD (GitHub Actions)

- **Workflow:** `.github/workflows/postman-tests.yml`
- **Triggers:** Push to `main`, pull requests to `main`, and manual `workflow_dispatch`.
- **Steps:** Checkout → Node 20 → Install Newman → Run collection with `OPENWEATHER_API_KEY` from secrets → Upload JUnit artifact (`results/tests.xml`).

**Repository setup**

1. Create a **public or private** GitHub repo and push this folder (including the collection and workflow).
2. Add **GitHub secret:**  
   **Settings → Secrets and variables → Actions → New repository secret**  
   - Name: `OPENWEATHER_API_KEY`  
   - Value: your OpenWeather API key  

After that, every push/PR runs the collection; the job **passes when all tests pass** and **fails when any test fails**, and the JUnit report is available under the run’s **Artifacts** (e.g. `test-results`).

---

## How to add new tests

1. **In Postman:** Open `OpenWeather-API.postman_collection.json` (import into Postman if you prefer to edit in the UI). Add a new request under the right folder (Health Check, Functional Tests, or Error Handling) and write **Tests** (e.g. `pm.test("...", () => { ... });`).
2. **Export:** Replace the repo’s `OpenWeather-API.postman_collection.json` with the exported collection (same name so the workflow does not change).
3. **Run locally:** Use the same `newman run` command above to confirm the new test passes, then push. The existing workflow will run the updated collection.

---

## Failure handling (Step 4)

The pipeline is set up so that:

- **When tests pass:** The workflow completes successfully and the JUnit report is uploaded as an artifact.
- **When tests fail:** The “Run Postman collection” step fails, the job is marked failed, and the JUnit report is still uploaded so you can see which request/test failed.
