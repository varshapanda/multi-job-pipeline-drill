# PIPELINE-AUDIT.md

# Pipeline Audit

## 1. lint

### Purpose
Runs ESLint to check the code for syntax errors and coding standard violations before any other jobs execute.

### Current Issue
- No timeout is configured.
- Other jobs do not wait for lint to finish, so they may continue even if lint fails.

### Correct Fix
- Add `timeout-minutes: 10`.
- Make lint the first job in the pipeline and configure dependent jobs using `needs:`.

---

## 2. unit-tests

### Purpose
Runs Jest unit tests to verify the application logic.

### Current Issue
- Runs in parallel with every other job.
- Does not wait for lint to complete.
- No timeout is configured.

### Correct Fix
- Add `needs: lint`.
- Add `timeout-minutes: 15`.

---

## 3. build

### Purpose
Builds the application and creates the `dist/` directory.

### Current Issue
- Runs in parallel instead of waiting for lint.
- Does not upload the build output, so other jobs cannot access it.
- No timeout is configured.

### Correct Fix
- Add `needs: lint`.
- Upload the `dist/` directory using `actions/upload-artifact@v4`.
- Name the artifact **app-build**.
- Add `timeout-minutes: 20`.

---

## 4. integration-tests

### Purpose
Runs integration tests using the built application.

### Current Issue
- Runs without waiting for the build job.
- Does not download the build artifact, so `dist/` is unavailable.
- No timeout is configured.

### Correct Fix
- Add `needs: build`.
- Download the artifact using `actions/download-artifact@v4`.
- Use the artifact name **app-build**.
- Add `timeout-minutes: 30`.

---

## 5. deploy-staging

### Purpose
Deploys the application to the staging environment after successful validation.

### Current Issue
- Runs on every branch.
- Does not wait for unit tests and integration tests to finish.
- No timeout is configured.

### Correct Fix
- Add:
  ```yaml
  needs:
    - unit-tests
    - integration-tests
  ```
- Add:
  ```yaml
  if: github.ref == 'refs/heads/main'
  ```
- Add `timeout-minutes: 15`.

---

## 6. deploy-production

### Purpose
Deploys the application to the production environment.

### Current Issue
- Runs immediately without waiting for staging deployment.
- Runs on every branch.
- No timeout is configured.

### Correct Fix
- Add:
  ```yaml
  needs: deploy-staging
  ```
- Add:
  ```yaml
  if: github.ref == 'refs/heads/main'
  ```
- Add `timeout-minutes: 15`.

---

## 7. notify

### Purpose
Sends a notification after the pipeline completes.

### Current Issue
- If any previous job fails, the notification job is skipped.

### Correct Fix
- Add:
  ```yaml
  if: always()
  ```
  so the notification job runs whether the pipeline succeeds or fails.