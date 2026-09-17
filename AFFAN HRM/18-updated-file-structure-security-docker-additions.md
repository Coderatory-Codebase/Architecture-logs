## 18. Updated File Structure --- Security & Docker Additions

These are *additions* to the structures already given in §9 and §10 ---
nothing there is being replaced.

**Backend (**`base_server`**) --- add:**

    src/
      middlewares/
        validateObjectId.ts          (NEW — rejects malformed :id before it reaches Mongoose)
      utils/
        encryption.ts                (NEW — AES-256 helpers for Payslip fields)

    Dockerfile                        (NEW)
    docker-compose.yml                (NEW, repo root — orchestrates backend + frontend + local mongo)
    .github/
      workflows/
        ci.yml                        (NEW — lint, typecheck, test, build on every PR)

**Frontend (**`calendar-frontend`**) --- add:**

    Dockerfile                        (NEW)
    .dockerignore                     (NEW)
    .github/
      workflows/
        ci.yml                        (NEW — lint, typecheck, test, build on every PR)
