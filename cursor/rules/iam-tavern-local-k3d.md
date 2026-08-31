---
title: Rule — iam-tavern-local-k3d
type: cursor-rule
created: 2026-08-31
updated: 2026-08-31
live_path: /Users/dev.sinha/nutanix-core/.cursor/rules/iam-tavern-local-k3d.mdc
alwaysApply: false
globs: "**/api_tests/**/*.tavern.yaml"
---

# iam-tavern-local-k3d.mdc

Repo rule on `nutanix-core`. Applies when working tavern YAML. Placeholders, not a home directory.

Index: [[cursor/00 - Index]] · pattern: [[wiki/patterns/ascii-only-tavern]]

## File body (`live_path`)

```mdc
---
description: How to build and run IAM (authn/authz) tavern tests locally against the k3d cluster, including testing your own code changes. Use when asked to run/iterate on iam-themis tavern tests (zz_global_iam, zzz_lattice_shard_copy, etc.) locally, or to validate a code change end-to-end via the tavern suite.
globs: **/api_tests/**/*.tavern.yaml
alwaysApply: false
---

# Running IAM authn/authz tavern tests locally (k3d)

## AT A GLANCE — what this is and when to use it

Use this rule whenever you want to exercise the iam-themis tavern suite
locally instead of waiting on canaveral/CircleCI. There are two paths:

- **Single-test fast loop (no Docker build):** `run-tavern-on-k3d.sh` runs
  one `.tavern.yaml` directly via `tavern-ci`. Best for iterating on a
  single test. See "COMPLEMENTARY PATH".
- **Full Job flow (Docker image):** build the test image into a tar,
  side-load into k3d, and run the real `iam-themis-tests-global` Job.
  Use when you need the suite to run exactly as CI runs it.

### How to ask the agent (prompt patterns)

- **Select one test:** you can ask to run a *specific* tavern file rather
  than the whole suite (pass it via `--test`, e.g.
  `--test zzz_lattice_shard_copy/test_lattice_shard_copy.tavern.yaml`).
- **Provide a GCP tar:** the host-side path needs an `iam-gcp-<build>.tar.gz`.
  Download that tar yourself and **point the agent at its path in the prompt**
  (it feeds `--gcp-tar`).
- **Test YOUR changes:** in the prompt, explicitly ask the agent to (1)
  **build the repo under test** and (2) **patch the freshly-built image tag
  into the Job yaml** (`imagePullPolicy: Never` + the local tag) so your code
  changes are what actually gets exercised — otherwise the Job pulls the
  stock CI image and your changes are never run.

## CONVENTIONS / PLACEHOLDERS

This rule is shared, so it uses placeholders, not any one dev's home dir.
Set these once per shell:

- `CORE`      = root of the nutanix-core checkout (contains iam-themis,
  iam-deployment, ...). e.g. `export CORE="$HOME/go/src/github.com/nutanix-core"`
- `THEMIS`    = `"$CORE/iam-themis"`
- `DEPLOY`    = `"$CORE/iam-deployment"`
- `ARTIFACTS` = scratch dir for tars (e.g. `"$HOME"` or `"$HOME/artifacts"`)
- `<sha>`     = short git sha of the iam-themis commit under test
- `<jumphost>`= ssh target of the Linux build host
- `<namespace>`= k8s namespace IAM came up in (`setup-iam.sh` defaults to `ntnx-base`)

## CONTEXT

The Job that exercises the iam-themis tavern suite (zz_global_iam,
zzz_lattice_shard_copy, etc.) is `iam-themis-tests-global`. In
production / canaveral CI it runs the image
`artifactory.dyn.ntnxdpro.com/canaveral-legacy-docker/nutanix-core/iam-themis-tests:<build>-global`,
built by `iam-deployment/vendor/make_api_tests.sh` during the iam-themis
CircleCI pipeline.

For local iteration the developer runs the SAME Job manifest in their k3d
cluster (`k3d-iamcluster` on Windows, brought up by
`iam-deployment/ftest/setup-iam.sh`). The k3d API server is reachable ONLY
from the developer's Windows box (local SSH tunnel at `0.0.0.0:<random-port>`);
from the Linux jumphost only the cpaas cluster is reachable. So the build
happens on the jumphost and the tar+manifest are side-loaded into k3d on
Windows.

Cluster vs reachability:
- `k3d-iamcluster` (Windows / local k3d) -> only reachable from the Windows box.
- `iam-cpaas-smsp-context` (cpaas cluster) -> reachable from the jumphost;
  this is where canaveral / p10y-style tests run when "test on the cluster"
  is mentioned.

## WHAT THE PRODUCTION BUILD ACTUALLY DOES

iam-themis CircleCI step "Create tavern test images" runs:

```bash
CGO_ENABLED=0 GOOS=linux go build \
    -o api_tests/setup_lattice_cg \
    ./cmd/setup_lattice_cg/
.workspace/dependencies/iam-deployment/vendor/make_api_tests.sh \
    --api_tests_location=api_tests \
    --docker_test_tag=$DOCKER_TEST_TAG \
    --docker_test_tag_global=$DOCKER_TEST_TAG_GLOBAL
```

`make_api_tests.sh`:
1. Stages `iam-deployment/vendor/api_tests/*` (the BASE Dockerfile +
   `/scripts/entrypoint.sh` + base requirements.txt) into
   `.workspace/tmp/api_tests_image_builder/`.
2. Copies `iam-themis/api_tests/*` into that builder's `src/` (so the tavern
   sources land at `/opt/tests_src` in the image, and the Linux
   `setup_lattice_cg` binary built in step 1 lands alongside as
   `/opt/tests_src/setup_lattice_cg`).
3. `docker build -t <DOCKER_TEST_TAG_GLOBAL> .` (and a non-global variant that
   strips zz_global_iam + zzz_lattice_shard_copy).
4. `docker push <tag>` to artifactory.

Note: `iam-themis/api_tests/Dockerfile` exists too but does NOT install
`/scripts/entrypoint.sh`, so the Job command
`bash /opt/tests_src/lattice_cg_setup.sh && exec /scripts/entrypoint.sh`
fails with "No such file" if you build directly from it. ALWAYS use the
`iam-deployment/vendor/api_tests` Dockerfile as the base, layered with
`iam-themis/api_tests/*` into `src/`.

## END-TO-END WORKFLOW (full Job flow)

### Step 1 — on the jumphost: build the image into a tar

```bash
cd "$THEMIS"
CGO_ENABLED=0 GOOS=linux go build -mod=vendor \
    -o api_tests/setup_lattice_cg ./cmd/setup_lattice_cg/
BUILDER=/tmp/iam-themis-tests-builder
rm -rf "$BUILDER" && mkdir -p "$BUILDER"
cp -rf "$DEPLOY"/vendor/api_tests/* "$BUILDER/"
cp -rf "$THEMIS"/api_tests/*        "$BUILDER/src/"
# First build needs an artifactory login because the base image
# (docker.dyn.ntnxdpro.com/ntnx-general-docker/alpine:3.11.6) is private.
# Credentials live in your own artifactory source file, conventionally
# ~/.ntnx/artifactory.dyn.ntnxdpro.com.source (exports DPRO_ARTF_USERNAME
# and DPRO_ARTF_API_KEY).
source ~/.ntnx/artifactory.dyn.ntnxdpro.com.source
echo "$DPRO_ARTF_API_KEY" | docker login docker.dyn.ntnxdpro.com \
    --username "$DPRO_ARTF_USERNAME" --password-stdin
TAG=iam-themis-tests:dev-$(git -C "$THEMIS" rev-parse --short HEAD)-global
cd "$BUILDER" && docker build --network=host -t "$TAG" .
docker save "$TAG" -o "$ARTIFACTS/$(echo "$TAG" | tr ':' '-').tar"
```

### Step 2 — on the Windows box: side-load into k3d and apply the Job

```bash
# copy tar from jumphost to Windows
scp <jumphost>:<ARTIFACTS>/iam-themis-tests-dev-<sha>-global.tar .
# side-load into k3d (deposits into every node's containerd; lets
# imagePullPolicy: Never work)
k3d image import iam-themis-tests-dev-<sha>-global.tar -c iamcluster
# apply the rendered Job manifest (override -n if not ntnx-base)
kubectl --context k3d-iamcluster -n <namespace> apply -f iam-themis-tests-global-k3d.yaml
```

The k3d manifest differs from the canaveral one:
- `image: iam-themis-tests:dev-<sha>-global` (the local tag)
- `imagePullPolicy: Never` (use the side-loaded image; don't hit artifactory)
- `imagePullSecrets:` REMOVED (no auth needed)
- `backoffLimit: 0` (tavern tests are deterministic; retries only delay signal)
- `namespace: <namespace>` (whatever setup-iam.sh used)

Keep your rendered manifest (`iam-themis-tests-global-k3d.yaml`) alongside the
tar on Windows so re-runs only need a refreshed image import.

### Step 3 — watch the test run

```bash
kubectl --context k3d-iamcluster -n <namespace> \
  get pods -l job-name=iam-themis-tests-global -w
kubectl --context k3d-iamcluster -n <namespace> \
  logs -f job/iam-themis-tests-global
# re-runs: delete the Job before re-applying
kubectl --context k3d-iamcluster -n <namespace> \
  delete job iam-themis-tests-global --ignore-not-found
kubectl --context k3d-iamcluster -n <namespace> \
  apply -f iam-themis-tests-global-k3d.yaml
```

## COMPLEMENTARY PATH (no Docker image build — single test)

`iam-deployment/services/ptest/run-tavern-on-k3d.sh` runs ONE iam-themis
tavern test directly from the host via `tavern-ci`, with no image build/push.
Use it for fast inner-loop on a single `.tavern.yaml`; use the Docker-image
flow above only for the full suite as CI runs it. Download the GCP tar first
and pass its path:

```bash
"$DEPLOY"/services/ptest/run-tavern-on-k3d.sh \
    --gcp-tar     <ARTIFACTS>/iam-gcp-<build>.tar.gz \
    --themis-repo "$THEMIS" \
    --test        zzz_lattice_shard_copy/test_lattice_shard_copy.tavern.yaml
```

## COMMON FAILURE MODES

- **`failed to authorize ... 401` on docker pull alpine:3.11.6** -> `source
  ~/.ntnx/artifactory.dyn.ntnxdpro.com.source` and `docker login
  docker.dyn.ntnxdpro.com` (Step 1).
- **Job pod `ImagePullBackOff` for `iam-themis-tests:dev-<sha>-global`** ->
  `imagePullPolicy` must be `Never` AND `k3d image import` must have run AFTER
  the docker build / tar refresh. Confirm:
  `k3d image list -c iamcluster | grep iam-themis-tests` (empty -> re-import).
- **`/scripts/entrypoint.sh: No such file or directory`** -> image was built
  from `iam-themis/api_tests/Dockerfile` instead of the
  `iam-deployment/vendor/api_tests` base. Rebuild through Step 1's staging.
- **Tavern stage "Create lattice consensus group" hangs** -> `setup_lattice_cg`
  binary missing or wrong arch. It MUST be built `CGO_ENABLED=0 GOOS=linux` on
  the jumphost; macOS / arm binaries silently sit at "Lattice Consensus Group
  Setup".
- **`/authorize` stages return "Status code was 401" through
  `HTTPS_AUTH_DOMAIN`** -> bearer token failed ext_authz on `iam-proxy:443`.
  Most likely the `On-Behalf-Of-TenantUUID` header on the `/oidc/token` call
  references a tenant not yet in the iam-user-authn `tenant` table. Order
  matters: tenant CREATE must precede the OBO-tagged `/oidc/token`.
```
