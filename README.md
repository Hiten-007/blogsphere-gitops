# blogsphere-gitops

What runs in the cluster. Argo CD watches `apps/` and keeps the cluster equal to it.
Each app repo's CI builds an image, pushes it to GHCR and commits the new `image:` tag here - that commit is the deploy.

| App repo | Image | Manifest |
|---|---|---|
| BlogSphere-backend (Spring Boot) | ghcr.io/hiten-007/blogsphere-backend | apps/backend |
| blogsphere-public (Next.js) | ghcr.io/hiten-007/blogsphere-public | apps/public |
| blogsphere-studio (Vite + nginx) | ghcr.io/hiten-007/blogsphere-studio | apps/studio |

## One-time setup
1. Edit hosts in `apps/ingress.yaml`.
2. Create the backend secret (NOT in git). Postgres/Redis are external (e.g. Railway):
   ```bash
   kubectl create ns blogsphere
   kubectl -n blogsphere create secret generic blogsphere-backend-env \
     --from-literal=PGHOST=... --from-literal=PGPORT=5432 --from-literal=PGDATABASE=... \
     --from-literal=PGUSER=... --from-literal=PGPASSWORD=... \
     --from-literal=REDISHOST=... --from-literal=REDISPORT=... --from-literal=REDISUSER=... --from-literal=REDISPASSWORD=... \
     --from-literal=FILE_STORAGE_TYPE=r2 --from-literal=R2_ACCOUNT_ID=... --from-literal=R2_ACCESS_KEY=... \
     --from-literal=R2_SECRET_KEY=... --from-literal=R2_BUCKET=... --from-literal=R2_PUBLIC_URL=...
   ```
3. `kubectl apply -f argocd/application.yaml`
4. In **each app repo**: Settings > Secrets > Actions > `GITOPS_TOKEN` (fine-grained PAT, `contents: write` on this repo).
   In the two frontend repos also add Actions **variables** (build-time URLs):
   `API_URL`, plus `STUDIO_URL` (public) / `PUBLIC_SITE_URL` (studio).
5. Rollback: `git revert --no-edit HEAD && git push`.
