---
title: Nutanix technical stack
type: source
tags: [notes, nutanix, iam, kronos]
status: living
updated: 2026-08-30
---

Three components of a data center:
	-compute
	-network
	-storage

The nutanix platform provides compute through CPUs and GPUs for applications as well as connectivity across multiple data centers.

The applications themselves run on top of VMs maybe over a kubernetes cluster.

This all runs on the nutanix hypervisor layer(AHV) 

Typical cloud providers will provide everything below the hypervisor layer

Started with the hypervisor and network storage the NKP was added on top of it

	NKP- does the kubernetes based management for customer applications
	 NDB - Helps you manage the database life cycle
	 NUS - nutanix unstructered storage

Prism acts as the management interface 
Nutanix Clound management(NCM) - manages the operations

Nutanix Central(NC) - Acts as a global management platform over all clusters and deployments

The three forms of compute are:
	containers : Done on prem managed by kubernetes
	VMs : Done on prem
	 Lambda functions: If your need is periodic and not continuous you could use lambda functions to respond to data dynamically and per call

Originally this was a datacenter on prem but now it can also be run along with cloud services

###AI

Use Cases:
	Foundation model training:
		Data Hub:
		1.)Injests mutlimodel data and converts it to text
		2.)Cleans the data
		3.) Creates embedding for data and puts into a vector DB
		4.) Used in MCPs or whatever
	Inference
	
Provides a scalable and safe inference endpoint

---

## IAM engineering notebook

> Cross-issue architecture and troubleshooting knowledge validated against current code,
> Git worktrees, PR review discussions, and CI through 2026-07-22. Repository-specific command
> logs, commit SHAs, and test evidence remain in each repository's canonical `.notes` file.
> Re-verify live GitHub state before acting because ticket notes can become stale.

### Issue relationship map

- [[inbox/ENG-915519 - Global IAM authoringScope/00 - Index|ENG-915519 — authoringScope]]
  introduced server-derived provenance for global Roles and Authorization Policies.
- [[inbox/ENG-949861 - Authoring scope mutation guard/00 - Index|ENG-949861 — mutation guard]]
  uses that provenance to prevent PC-side mutation of NC-authored entities.
- **ENG-910344** introduced the global-ACP identity enrich/resolve model.
- [[inbox/ENG-910347 - Resolver Shard ACP/00 - Index|ENG-910347 — resolver shard copy]]
  extends enrich-on-source and resolve-on-target from incremental Lattice changes to
  bulk shard-copy bootstrap.
- [[inbox/ENG-910350 - Global ACP external identities/00 - Index|ENG-910350 — global ACP identity restriction]]
  prevents global ACPs from referencing cluster-local identities that cannot federate.
- [[inbox/ENG-932537 - IAM products field/00 - Index|ENG-932537 — products]]
  adds product membership to Entities and server-derived product unions to Roles.
- **ENG-948242 — backend-driven entity search** adds six optional per-attribute search metadata
  strings to `objects.json`, persists them in the existing Object attribute JSONB document, and
  exposes them through v4 Entity GET/LIST. Themis Tavern coverage exercises direct v1, `/proxy`,
  and v4 seed-config round trips with direct, delegated, and legacy attributes. Live PC
  verification on 2026-07-16 populated direct metadata on `OpsMgmt:aws_image.uuid` and delegated
  metadata on `category_uuid`, reran bootstrap image 5049, and confirmed all six exact values in
  `iam-themis.object.object_attribute_list`. Versioned v4.1.b4 LIST and GET-by-ID through
  `iam-proxy` both returned HTTP 200 with all direct/delegated fields and omitted metadata on the
  legacy attribute. PRs #936, #381, #787, and #1630 were updated in repository-template format
  with this evidence; #1630 includes pushed integration commit `5f7afd871`. Proxy-write and
  v4-seed deployed paths remain separate gates. July 20 review follow-up expanded the Tavern test
  to 14 stages in `c4e1d16f5`: every v1/proxy/v4 write path now covers direct, delegated, and
  legacy attributes and validates all six fields through both LIST and GET-by-ID. Canonical evidence:
  `iam-themis/.notes/ENTITY_SEARCH_METADATA_CONTEXT.md`.
  All four feature branches were subsequently brought onto their current default branches and
  pushed: API `87213cde`, Utils `e7cde58f`, Bootstrap `88d7c3d7`, and Themis `f16dd846`.
  The Bootstrap add/add conflict retained both products and search-metadata tests; GitHub then
  reported all four PRs mergeable. Deployment `1784576518-b5979b` later ran the metadata Tavern
  scenario successfully, but 69 unrelated tests used stale Themis/test images from build 15010:
  fixtures expected `$reserved.$fv=v4.r1.b3` while the runtime correctly returned b4. Current
  source already contains b4 expectations; publish a matched current Themis service/test image
  pair before rerunning Bootstrap automation. PR test sections now contain concise reproduction
  steps—commands, direct/delegated/legacy fixture shape, config load, JSONB check, and Entity
  LIST/GET assertions—without environment-specific IPs or credentials.
  On 2026-07-28, Themis build 15030 and its automation passed. Bootstrap was legitimately updated
  to current master in merge `007d0608` (rather than using an empty CI commit), focused tests
  passed, and the push started fresh CircleCI build 5153 so automation can consume the matched
  Themis service/test image pair.
  On 2026-07-22, live GitHub state showed API #936, Utils #381, and Bootstrap #787 behind their
  moving bases and still review-required. Themis #1630 was policy-blocked for review rather than
  conflicted, with CircleCI, IAMv2 automation, and secret scanning green. Bootstrap #787 retained
  the previously diagnosed stale-image automation failure; do not reinterpret that historical run
  as a metadata regression.
- **ENG-953364** adds the run-once completion contract around the role-products brownfield
  migration.
- **ENG-956163** hardens that migration's optimistic concurrency behavior. Themis returns its
  typed storage ETag mismatch through the normal tenant migration error path instead of skipping
  the role or adding another retry loop. Bootstrap already withholds global completion and retries
  failed migration work; the next Themis invocation re-queries the latest ETag and operation list
  before recomputing products. This is a Themis-only behavior change—no API enum/schema change
  is needed.
  Deterministic Go tests own the internal race; Tavern owns the black-box contract by creating
  a stale brownfield role, migrating it, checking persisted/OData-filterable products, and
  proving a rerun leaves its ETag unchanged.
  Canonical execution evidence: `iam-themis/.notes/PRODUCTS_FIELD_CONTEXT.md` §60.
- **ENG-924709** adds the configurable global-role deletion guardrail. Its in-memory runtime
  flag must not be confused with durable migration completion.

### Cross-repository ownership

For a public IAM API field, trace the chain in this order:

1. `ntnx-api-iam` owns the public v4 schema and generated request/response models.
2. `iam-utils` owns internal Swagger models and shared helpers used by Themis.
3. `iam-themis` owns AuthZ handlers, storage, migrations, Lattice behavior, and most AuthZ
   Tavern tests. It vendors generated code from upstream repositories.
4. `iam-bootstrap` owns startup orchestration and seed-data conversion into service APIs.
5. `iam-user-authn` owns tenants and identity resolution/enrichment behavior.
6. `iam-deployment` owns deployed configuration and the CI/Tavern image/runtime wiring.

Do not assume every ticket needs all six repositories. ENG-910347 and ENG-910350 are
Themis-only because their wire dependencies already existed. ENG-932537 genuinely needed
schema, internal model, service, bootstrap, and later deployment coordination.

### Global versus tenant-scoped execution

- Global coordinators own fan-out and the final completion decision.
- Tenant-scoped workers should process exactly one tenant and report success/failure.
- A global completion flag is written only after every tenant succeeds.
- An empty successful tenant enumeration must not automatically mean "complete" when the
  operation is expected to fan out; ENG-953364 treats it as retryable and leaves completion
  unset.
- Verify pagination from the actual endpoint. AuthN's current v1 tenant list is unpaginated,
  but that is an implementation fact, not a permanent API assumption.
- Lattice tablet copy is tenant-bounded by `WalTenantID`; do not copy that assumption to
  arbitrary bootstrap or REST batches.

### Replication classifications

- `storage.Role` and `storage.AccessPolicy` fields with JSON tags can ride the existing
  Lattice storage-object contract when create/apply paths persist the unmarshaled object.
- Entity/Object products are independently seeded on each cluster; they are not copied by
  Lattice.
- Role products are derived from referenced entity products and are Lattice-replicated for
  global roles.
- Identity federation uses enrich at the producing cluster and resolve at the consuming
  cluster. Enrichment metadata is transit-only and must not become persisted ACP state.
- Tests must say whether they validate replication or equivalent independent seeding.

### End-to-end change checklist

For every new field or behavior, inspect:

1. Source schema and generated model shape.
2. API request conversion and read-response projection.
3. Storage struct and JSON replication contract.
4. SQL migration, insert/update behavior, scan cases, and every SELECT column list.
5. In-memory SQLite/sqlmock schemas used by tests.
6. Bootstrap seeding and global orchestration.
7. Deployment configuration and defaults.
8. Unit, race, API, Tavern, and brownfield coverage.
9. Downstream module pin, vendor output, consumer build, Swagger copy, and `$fv` assertions.

A successful code-generation command proves only that generation ran. It does not prove the
committed generated tree, downstream vendoring, imports, compilation, or automation fixtures
are correct.

For Kronos b4, the runtime Swagger mapping is
`ntnx-api-iam/.../target/generated-api-artifacts/swagger-iam-v4.r1.b4-all.yaml` to
`iam-themis/services/config/v4/schema_yamls/swagger-iam-v4.1.b4-all.yaml`. Copy the generated
artifact byte-for-byte and leave released b3 unchanged; ENG-948242 verified this with matching
SHA-256 plus Themis config/middleware tests. Follow the products dependency workflow with
`make revendor` (`go mod tidy && go mod vendor && go mod verify`), not only `go mod vendor`.

### Troubleshooting patterns

- **Value exists in the DB but reads return empty:** inspect all query column constants and
  scan switches. ENG-915519 found five omitted SELECT lists after migration/INSERT were
  already correct.
- **Seed JSON has the field but runtime rows do not:** inspect each bootstrap conversion.
  ENG-932537 initially dropped products on both v1 and v4 seeding paths.
- **Tavern later reports 428 or an unset variable:** find the first failed stage. Tavern files
  are sequential; a failed strict-body assertion prevents `save`, so the later error is often
  only a symptom.
- **Retry reports "already exists":** the first pod may have created state before failing.
  Retries are not automatically isolated or idempotent.
- **Global Tavern tests contaminate one another:** create a per-test tenant and consistently
  send `On-Behalf-Of-TenantUUID`. Create the tenant before requesting an OBO token.
- **AuthN client resolves to `https://:0`:** inspect the deployed
  `externalServices.authnService`, not only repository defaults.
- **Concurrent calls on a generated SDK client:** run `go test -race`. ENG-910350 proved the
  shared generated client mutates request/logger state and is not safe for goroutine fan-out.
- **Local Tavern image misses `/scripts/entrypoint.sh`:** build from
  `iam-deployment/vendor/api_tests`, not the similarly named Themis Dockerfile.
- **Shard-copy setup hangs:** verify `setup_lattice_cg` was built for Linux with
  `CGO_ENABLED=0`.
- **Local lint differs from CI:** inspect the deployment-owned linter wrapper, exact version,
  and config. Do not infer CI behavior from a similarly named repository-local file.
- **ntnx-api-iam codegen fails on `java.lang.Enum.name`:** the repository uses JDK 21 but the
  generator requires scoped `MAVEN_OPTS --add-opens` flags. Reuse the exact command recorded in
  the authoring-scope/ENG-948242 execution notes rather than changing the repository JDK.
- **A full ntnx-api-iam reactor fails only in JavaScript `npm install` with
  `UNABLE_TO_GET_ISSUER_CERT_LOCALLY`:** verify all earlier reactor modules separately, then use
  a command-scoped npm certificate workaround if approved; do not persistently weaken npm
  configuration. ENG-948242 passed with scoped `npm_config_strict_ssl=false`.
- **`go get` of current ntnx-api-iam breaks Themis at `ApiVersion.GetName`:** newer generated
  models expose `AssociatedEndpoint.ApiVersion` as `*string`. Reuse/reconcile the nil-safe string
  compatibility change from iam-themis PR #1619 rather than inventing a second representation.
- **Alpine `apk` says a known package does not exist after `TLS: server certificate not
  trusted`:** the missing-package message is secondary because `APKINDEX.tar.gz` was unavailable.
  Fix Docker/BuildKit trust or use the approved corporate mirror; do not change feature code,
  package names, Swagger, dependencies, or the shipped Dockerfile. Commit the exact source before
  treating a manually deployed PC image as reproducible validation evidence. For immediate PC
  validation, products proved a network-free fallback: Zig/musl cross-compile the amd64 binary,
  build an overlay on the PC from its cached working Themis image, load it into containerd
  `k8s.io`, and roll out the deployment. A feature with runtime-schema changes such as
  ENG-948242 must overlay `/v4/schema_yamls` as well as the executable.
- **`mspctl application` prints only parent-command usage:** put the operation immediately after
  `application`, for example `mspctl application get iam-themis -u <uuid> -f
  ./iam-themis.yaml`; placing `-u` before `get` did not dispatch the operation. Validate the full
  YAML before delete/apply. Replacing tabs cannot repair duplicated or structurally misplaced
  manifest sections, and a valid local file may request a different image than the healthy live
  deployment—inspect both before applying.
- **MSP delete succeeds but immediate apply hangs or creates partial resources:** application
  delete is asynchronous (`202 Accepted`). Wait until its Kubernetes resources are actually gone
  before applying. Kubernetes image values must omit URL schemes (`image: artifactory...`, not
  `image: http://artifactory...`). A successful bootstrap only validates the file on that same PC;
  confirm metadata keys, feature image, and bootstrap target all refer to one cluster before
  claiming an end-to-end result.
- **Nested internal and public models use different names:** for Entity attributes,
  `displayName`/`uiDisplayName` internally map to `name`/`displayName` publicly. Explicit
  conversion is required; direct JSON unmarshalling can silently put values in the wrong field.
- **Many Tavern tests fail with expected b3 but actual b4:** check which paired Themis service and
  test images the deployment selected. ENG-948242 deployment `1784576518-b5979b` used stale build
  15010 after the current Themis build failed to publish; the feature's metadata scenario passed,
  while broad fixtures still expected b3 against the b4 runtime schema. Fix/publish the current
  Themis build and rerun with matched images instead of changing passing feature assertions.
- **Verbose deployment logs expose credentials:** inspect rendered command lines before sharing
  artifacts. The ENG-948242 automation log printed an Artifactory credential; rotate it and add
  pipeline redaction rather than copying the value into notes or PRs.
- **Optimistic migration update gets an ETag mismatch:** retrying the same storage call cannot
  succeed because its expected ETag is captured. Before adding a local retry, check whether the
  existing coordinator already retries the complete operation. For role products, Themis returns
  the typed conflict; Bootstrap withholds completion and reruns the migration, whose normal query
  obtains fresh ETag and operation state. Established by ENG-956163.

### CI interpretation

- Read the first failing assertion and response body, not only cleanup noise or a generic
  GitHub status.
- Compare the same check against the latest base branch before assigning blame.
- On 2026-07-14, latest `iam-themis`, `iam-bootstrap`, and `iam-deployment` base branches also
  failed IAMv2 automation. This is evidence that current feature-branch automation failures
  may be environmental/base-related, but it does not replace reading the failing artifact.
- For ENG-948242, the later Bootstrap automation failure was deterministic version skew, not a
  metadata regression: the new scenario passed and the repeated assertion was b3 expected versus
  b4 actual from a stale image pair. Resolve the upstream Themis image publication first.
- Verify the actual service image, test image, runtime Swagger, and test expectations selected by
  deployment. A green source build or correct branch does not prove automation exercised that
  artifact set.
- Mergeability and base ancestry are timestamped evidence. ENG-948242's four branches were current
  and mergeable on July 20; three reported `BEHIND` again on July 22 after their bases moved.
  Re-query and rerun repository gates immediately before merge.
- Do not create empty commits to "heal" deterministic failures. Change the cause; rerun only
  when evidence indicates a transient infrastructure failure.

### Reviewer and PR expectations

- Verify reviewer suggestions against PostgreSQL syntax, generated-model shape, current
  handler behavior, and replication semantics before implementing.
- Deterministic sorting can be required even for set-like arrays when values contribute to
  etags or Lattice payloads.
- PostgreSQL `text[]` uses `'{}'` for an empty-array default; `[]` is JSON syntax.
- Reply "done" only when the requested change was actually implemented.
- Follow each repository's PR template and persist the draft in the canonical `.notes` log.
- Open a separate repository PR only when that repository needs a real source/config change.
- GitHub IAM repositories use normal PR pushes. Gerrit review pushes must use
  `refs/for/<target-branch>`.

### Workspace and Git safety

- Check branch, status, tracking, and worktree ownership before every significant action.
- Preserve unrelated local changes. The IAM workspace commonly contains parallel worktrees
  and contaminated vendor state.
- Local `go.mod` replacements can silently regenerate vendor from sibling checkouts.
- Ordinary stashes omit untracked files; recover only intended paths from contaminated
  stashes.
- Inspect the final commit message after hooks. Prior IAM sessions found an injected Cursor
  co-author trailer and used `commit-tree` to reproduce the same tree with the intended
  subject-only message.
- Use `--force-with-lease` only when rewriting a published feature branch and after verifying
  the expected remote tip.

### Feature-specific decisions that are not universal

- ENG-932537 products apply to Roles and Entities, not AccessPolicies.
- ENG-915519 authoring scope is a scalar, immutable, server-stamped value.
- ENG-910350 currently classifies external identities through enrichment connector types
  `{saml, ldap}` and retains a skip flag; its `EXTERNAL`/`PROPAGATED` semantics still require
  product confirmation.
- ENG-953364 rejects an empty tenant list because that migration must fan out; other APIs may
  legitimately return zero objects.
- ENG-956163 delegates retries to iam-bootstrap because it already retries the entire failed
  migration and withholds completion. Do not delegate blindly when the caller retries only a
  partial operation or can incorrectly record success.
- ENG-924709 uses an in-memory runtime guard flag; do not use that pattern for durable,
  cross-pod migration completion.

### Before planning the next IAM issue

Search this vault by issue key, repository, component, reviewer, migration name, and failure
symptom. Then verify the findings against current code, Git state, PR discussion, and CI.
State which lessons apply, which feature-specific decisions do not, and which assumptions
remain open before editing.

Do not store credentials, tokens, private keys, or raw sensitive logs in this vault.

### Related notes

- [[inbox/ENG-915519 - Global IAM authoringScope/11 - Process Lessons|ENG-915519 process lessons]]
- [[inbox/ENG-932537 - IAM products field/10 - Process Lessons|ENG-932537 process lessons]]
- [[inbox/ENG-932537 - IAM products field/11 - Issues Encountered & Fixes|ENG-932537 incidents]]
- [[inbox/ENG-910347 - Resolver Shard ACP/03 - Decisions Log|ENG-910347 decisions]]
- [[inbox/ENG-910350 - Global ACP external identities/03 - Decisions Log|ENG-910350 decisions]]
- ENG-948242 canonical execution log:
  `iam-themis/.notes/ENTITY_SEARCH_METADATA_CONTEXT.md`
- Cross-ticket validated source map:
  `iam-themis/.notes/IAM_CROSS_ISSUE_LESSONS.md`