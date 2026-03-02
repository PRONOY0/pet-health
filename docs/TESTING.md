# Testing Guide

This document describes the testing strategy for the Pet Health monorepo: **unit tests**, **integration tests**, and **manual/exploratory testing**, plus how to run tests and interpret **Pass/Fail** and **coverage** output.

---

## 1. Unit testing

- **Web (Vitest)**  
  - Location: `apps/web/src/**/*.test.{ts,tsx}`  
  - Examples: `petsApi.test.ts` (API client with mocked HTTP).  
  - Run: `pnpm test:run` or `pnpm test:coverage` in `apps/web`.

- **API (Jest)**  
  - Location: `apps/api/src/**/*.spec.ts`  
  - Examples: `pet-with-species.mapper.spec.ts`, `species.service.spec.ts`.  
  - Run: `pnpm test:run` or `pnpm test:coverage` in `apps/api`.

---

## 2. Integration testing

- **Web**  
  - Component + hooks integration: `PetsListPage.integration.test.tsx` (mocked `useListPets`, real UI tree).  
  - Run with the rest of web tests: `pnpm test:run` / `pnpm test:coverage` in `apps/web`.

- **API**  
  - HTTP + controller + mocked service: `species.controller.integration.spec.ts` (supertest, `SpeciesService` mocked).  
  - Run with the rest of API tests: `pnpm test:run` / `pnpm test:coverage` in `apps/api`.

---

## 3. Manual / exploratory testing

Use these to verify behaviour in a real environment (local API + DB, local web app).

### Prerequisites

- API: `.env` (or env) with `DATABASE_URL` and JWT/API config.
- Web: `VITE_API_URL` pointing at the running API (e.g. `http://localhost:5000`).

### Steps

1. **Start API and Web**
   - From repo root: `pnpm dev:api` and `pnpm dev:web` (or run both via `pnpm dev` if configured).
   - Or from `apps/api`: `pnpm dev`; from `apps/web`: `pnpm dev`.

2. **Auth**
   - Register / log in via the web app.
   - Confirm cookie/session and that protected routes load.

3. **Species (read-only)**
   - Open list of species (e.g. `/species` or wherever the UI loads them).
   - Check that species names and optional images match DB/seed.
   - Try an invalid species id and confirm 404 or appropriate error.

4. **Pets**
   - List pets: only the current user’s pets, with species name and image.
   - Create pet: choose species, set name, birth date, breed, optional lifespan; submit and confirm it appears in the list with correct species details.
   - Open a pet detail page and confirm data and species info.
   - (If implemented) Edit/delete and confirm behaviour and errors.

5. **Error cases**
   - Create pet with invalid species id → expect validation/4xx.
   - Unauthenticated request to a protected endpoint → expect 401.
   - Invalid or missing required fields → expect validation errors.

6. **Responsiveness / UX**
   - Resize window; test sidebar and key flows on a small viewport.
   - Check loading and error states (e.g. empty list, failed load).

---

## 4. Scripts and Pass/Fail + coverage

### From repo root

| Script | What it does | Pass/Fail & coverage |
|--------|----------------|------------------------|
| `pnpm test` | Runs default test task for all apps (Turbo). | Each app’s runner prints **Pass/Fail** and, if configured, **coverage**. |
| `pnpm test:all` | Runs `test:run` for all apps (single run, no watch). | Same as above; exit code is non‑zero if any app fails. |
| `pnpm test:coverage:all` | Runs `test:coverage` for all apps. | **Pass/Fail** per file/suite plus **coverage** (e.g. statements, branches, functions, lines). |
| `pnpm test:web` | Runs tests for the web app only. | Vitest: **Pass/Fail** and optional **coverage**. |
| `pnpm test:api` | Runs tests for the API only. | Jest: **Pass/Fail** and **coverage** table + summary. |
| `pnpm test:coverage:web` | Web tests with coverage. | **Pass/Fail** + coverage report (e.g. in terminal and `coverage/`). |
| `pnpm test:coverage:api` | API tests with coverage. | **Pass/Fail** + coverage table and summary (e.g. `coverage/`). |

### From each app

- **Web** (`apps/web`)
  - `pnpm test` – Vitest watch.
  - `pnpm test:run` – Single run; **Pass/Fail** in terminal.
  - `pnpm test:coverage` – Single run with **coverage** (e.g. v8); **Pass/Fail** and summary.

- **API** (`apps/api`)
  - `pnpm test` / `pnpm test:run` / `pnpm test:coverage` – Jest single run with **coverage**; **Pass/Fail** and table in terminal, reports in `coverage/`.

### Interpreting output

- **Pass/Fail**: Shown as “Test Suites: X passed, Y total” and “Tests: X passed, Y total”. Non‑zero exit code means **Fail**.
- **Coverage**: Shown as percentages (e.g. statements, branches, functions, lines) per file and a summary. HTML reports are usually under `coverage/` in each app.

---

## 5. Summary

- **Unit**: Isolated logic (mappers, services with mocks, API client with mocked HTTP).
- **Integration**: HTTP/controller (API) and page/components with mocked data (web).
- **Manual**: Full stack in the browser and against a real API/DB using the steps above.
- Use **`pnpm test:all`** or **`pnpm test:coverage:all`** from the root for a single run with **Pass/Fail** and **coverage** for both apps.
