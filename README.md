# open-idp-apps

Catalog of applications registered in [Open IdP](https://github.com/jasondavindev/open-idp). This repository holds no application source code — it only holds the Helm values that ArgoCD needs to deploy each registered app to the cluster.

## Role in the GitOps flow

Open IdP reconciles the cluster from two ArgoCD `Application` sources: `01-applications/` in the `open-idp` repository for platform components, and `apps/<name>/` in **this** repository for developer applications. The `idp-apps` instance of the `argocd-app-of-apps` chart watches this repo and creates one ArgoCD `Application` per folder under [`apps/`](apps/).

An application lands here through its own CI/CD, not by hand:

1. An app repository (e.g. [`safplatform`](https://github.com/jasondavindev/safplatform)) calls the [`build.yaml`](https://github.com/jasondavindev/open-idp/blob/main/.github/workflows/build.yaml) reusable workflow, which builds and pushes its image.
2. It then calls [`deploy.yaml`](https://github.com/jasondavindev/open-idp/blob/main/.github/workflows/deploy.yaml), which copies that repo's `.idp/` Helm chart into `apps/<app_name>/` here, sets `global.image.tag` to the new image tag, and commits (`[skip ci]`) straight to `main`.
3. ArgoCD (auto-sync, prune, self-heal) picks up the change and rolls out the new version.

Because of this, changes under `apps/` are almost always automated commits from an app's deploy job — see the `chore(cd): bump <app> image tag to <sha>` commits in the git history. Registering a new application also requires adding it to `idp-apps.applications` in [`open-idp/00-core/argo/values.yaml`](https://github.com/jasondavindev/open-idp/blob/main/00-core/argo/values.yaml).

## Layout

Each entry under `apps/` is a self-contained Helm chart, typically depending on the shared [`web`](https://github.com/jasondavindev/open-idp/tree/main/charts/web) chart published from `open-idp`:

```
apps/<app_name>/
├── Chart.yaml     # chart metadata + the `web` chart dependency
├── values.yaml    # app-specific values; global.image.tag is what CI bumps
├── Chart.lock     # git-ignored, rebuilt by `helm dependency build`
└── charts/        # git-ignored, vendored dependency .tgz files
```

## Registered applications

| App | Source repository |
| --- | --- |
| [`safplatform`](apps/safplatform/) | [jasondavindev/safplatform](https://github.com/jasondavindev/safplatform) |
