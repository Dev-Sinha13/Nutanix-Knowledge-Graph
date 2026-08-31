---
tags: [eng-915519, postmortem, issues, fixes]
updated: 2026-05-28
---

# Issues Encountered & Fixes

Concrete problems hit during this work, with symptom → cause → fix → lesson. Companion to [[11 - Process Lessons]] (abstract patterns); this one is specific incidents.

## A. Multi-agent contention on the iam-themis repo

### A1. Branch drift between shell calls

- **Symptom:** Switched to `ENG-915519-authoring-scope-handler` with `git switch`. Ran a `gofmt` check. Tried `git diff` — the diff was empty. `git log` revealed HEAD was actually on `ENG-932537-products-field`. I had not issued any switch command between those two.
- **Cause:** Another agent was running in parallel on `ENG-932537-products-field` and was issuing `git switch` between operations on the same working directory.
- **Fix:** Re-switched explicitly. Chained all subsequent work into a single shell call (`git switch ... && do-things && verify`) to give the parallel agent zero window to interfere.
- **Lesson:** Whenever there's any chance of parallel activity, **run `git rev-parse --abbrev-ref HEAD` as the first command in every multi-step shell call**. Cheap, prevents surprise commits on the wrong branch.

### A2. Sticky `go.mod` TEMPORARY local-replace block

- **Symptom:** After running `go build`, `git status` showed `go.mod` as modified — even though I hadn't touched it. The diff revealed a new block:
  ```go
  // ENG-932537 [TEMPORARY for dev — REMOVE before opening PR; restore real pseudo-versions]
  // Auth issue: GIT_TOKEN lacks access to nutanix-beam transitive dep,
  // blocking `go get @branch`. Pointing to local sibling checkouts.
  github.com/nutanix-core/iam-utils => ../iam-utils
  github.com/nutanix-core/ntnx-api-iam => ../ntnx-api-iam
  ```
- **Cause:** The parallel agent (working on ENG-932537) was using local-sibling replace directives for dev. It would `go mod tidy` before its work, putting the block back in `go.mod`, then carry on. The block was uncommitted in any branch (the comment literally says "REMOVE before opening PR").
- **Fix attempts that failed:** `git checkout -- go.mod` reverted the file, but the next `go build` re-contaminated. `git stash push -- go.mod` succeeded but `go test` triggered re-contamination immediately.
- **Final fix:** Asked the user to stop the parallel agent. Then `git reset --hard HEAD` to fully clean the tree, then proceeded with verification + push.
- **Lesson:** Before any `go build` / `go test`, always check `git diff HEAD -- go.mod` first. If the file is "modified" but you haven't touched it, something — local-replace block, parallel agent — is mutating dependency state behind your back.

### A3. Vendor directory contamination

- **Symptom:** After the local-replace block was active (even briefly), `vendor/` showed modified files: `iam-utils/themisutil/generated/models/*.go` had ENG-932537 `Products` field additions; `ntnx-api-iam/iam-server-codegen/models/.../authz_model.go` had unreleased `AuthoringScope` enum definitions.
- **Cause:** `go build -mod=vendor` and `go test -mod=vendor` will, with `replace ... => ../sibling-repo` active, sync the vendor directory from the local sibling checkouts. Each build silently re-pulls.
- **Fix:** `git checkout -- vendor/` to discard the local pulls, after the TEMPORARY block was gone from `go.mod`.
- **Lesson:** Local replace + vendor mode = silent vendor mutation. If you must use local replaces for dev, **expect `vendor/` to drift** and revert it before any commit. The contamination is invisible in `go.mod` itself once you revert.

## B. Stale CI bot reports

### B1. IamInfra gofmt complaint at the wrong line

- **Symptom:** IamInfra bot comment: `services/server/storage/sql/role.go:73:1: File is not properly formatted (gofmt)`, quoting `// authoring_scope is intentionally omitted from UPDATE statements to ensure the value`. I read `role.go:73` — the content was completely different. I `grep`d the entire repo for the quoted comment text — zero matches.
- **Cause:** The bot fired on an earlier commit on the branch. By the time Praveen reviewed `577fb531`, the offending comment had been removed in a subsequent push. The bot comment was stale, anchored to a commit that no longer represented the file's state.
- **Fix:** Ran `gofmt -l services/ cmd/` to find the ACTUAL current offender. It pointed at `services/server/storage/sql/authoring_scope_sql_test.go` (a different file entirely — Go-standard `*` → `-` bullet conversion in a doc comment). `gofmt -w` on that file resolved CI.
- **Lesson:** Never trust a CI bot's line number without re-running the underlying check against current HEAD. The line/file may have moved; the rule may have already been satisfied. **`gofmt -l` is the ground truth, not the bot comment.**

### B2. Pre-existing `go vet` warning misread as new

- **Symptom:** Post-rebase onto master, `go vet ./services/server/apihandler/` flagged `proxy_role_test.go:297:32: RoleProxyResponse struct literal uses unkeyed fields`. The file `proxy_role_test.go` was nowhere in my PR's diff.
- **Cause:** The warning is pre-existing on master itself. The rebase didn't cause it; vet just discovered it because I happened to vet the whole package.
- **Fix:** Verified by overlaying master's version of the file (`git checkout origin/master -- proxy_role_test.go`) and re-running vet — same warning. Then restored my branch's version. Confirmed not my issue, ignored.
- **Lesson:** When a vet/lint warning appears on a file you didn't touch, **always reproduce it against `origin/master`** before treating it as caused by your change. The cheap check: `git diff origin/master..HEAD -- path/to/file` — if the diff is empty, the issue is upstream.

## C. Force-push after rebase

### C1. Push failed after rebase

- **Symptom:** Tried `git push origin ENG-915519-authoring-scope-handler` after rebasing onto new master. Got the standard "remote contains work that you do not have locally" rejection. Plain push wouldn't work because rebase rewrites SHAs.
- **Cause:** Rebase replaces commits with new SHAs (`577fb531d` became `2cf65eb45`, `e48cc202e` became `e7ad49866`). The remote still has the old SHAs.
- **Fix:** `git push --force-with-lease origin ENG-915519-authoring-scope-handler`. Force-with-lease confirms the remote is still at the expected old SHA before force-pushing — aborts if anyone else pushed in between.
- **Lesson:** After rebasing a feature branch, **always use `--force-with-lease`, never plain `--force`**. The lease guarantees you don't clobber unexpected work. Plain `--force` is genuinely unsafe; `--force-with-lease` is the standard safe variant.

## D. Shell chain failures

### D1. `grep -c` exit code breaking chained commands

- **Symptom:** I had a verification chain `grep -c "ENG-932537" go.mod && echo "clean" && gofmt -l ...`. The `grep -c` returned `0` (meaning no matches — which was the desired clean state), but `grep` exits with code 1 when no matches are found. The chain stopped there; subsequent commands never ran.
- **Cause:** `grep` exit codes: 0 = found, 1 = not found, 2 = error. `&&` treats 1 as failure and stops the chain.
- **Fix:** Either `grep -c ... || true` (to swallow the no-match exit), or run subsequent commands in separate calls.
- **Lesson:** When using `grep` in a `&&` chain to verify ABSENCE of something, **append `|| true`** or use `if ! grep ...; then ...; fi`. Same applies to `diff` (returns 1 when files differ — which is sometimes what you want).

## E. Documentation drift

### E1. Progress doc §8 snapshot was wrong twice

- **Symptom:** Reading `ntnx-api-iam/.notes/AUTHORING_SCOPE_CONTEXT.md` §8 to get up to speed, the section claimed: "Step 2 still pending user sign-off; iam-utils branch has no commits yet; iam-themis not started." All three were stale.
- **Cause:** §8 had been edited without verification against the GitHub state. The branch on `ntnx-api-iam` actually had pushed commits; iam-utils had `f9885b3` on origin; iam-themis had an active PR #1601 that the doc never mentioned.
- **Fix:** Re-derived §8 from actual ground truth: `gh pr list --author "@me" --state all"` in each repo, `git log --oneline origin/<branch>` for HEAD SHAs, plus the open file list for working-tree state. Added a self-correction note at the top of §8: "Always verify GitHub PR state ... before trusting this section."
- **Lesson:** Live status sections in long-lived docs **decay silently**. Either auto-generate them, or treat them as advisory and re-derive from canonical sources (git, GitHub) every session. The progress doc now has a self-correction note prompting future readers to verify.

## F. Build environment (resolved earlier in the work)

### F1. Maven Python SDK build failure — pyenv vs Homebrew Python

- **Symptom:** `mvn install` reactor failed at the `iam-python-client-sdk` module's `generate_rst` step. Error involved Python 3.14.5 features that the codebase didn't support.
- **Cause:** Homebrew's Python 3.14.5 was first on `PATH`. The Maven plugin's venv had been built against it, but the project needed pyenv's 3.9.16.
- **Fix:** Removed the stale `.venv` at `iam-api-external/iam-api-codegen/iam-python-client-sdk/target/generated-sources/swagger/.venv`. Re-ran the Maven command with `export PATH="$HOME/.pyenv/shims:$PATH"` prepended.
- **Lesson:** Project venvs **survive across Python version changes** and become poisoned. When a multi-Python-version build fails after a `brew upgrade`, the venv is often the culprit. Wipe and rebuild.

### F2. Schema-review-tool plugin parameters renamed

- **Symptom:** Added the `lint-checker-maven-plugins` plugin to `pom.xml` with goal `review` and parameter `swaggerFile`. Maven errored: "Could not find goal 'review'" / "parameters are missing or invalid".
- **Cause:** The plugin's API had been updated since the old docs were written. New goal: `generate-report`. New parameter name: `inputOpenApiFile`.
- **Fix:** Updated the pom.xml block to the new names. Build succeeded.
- **Lesson:** When a Maven plugin behaves like it doesn't exist, **inspect its current help** (`mvn help:describe -Dplugin=<group:artifact>`) before assuming you're using it wrong. Plugin maintainers rename goals and parameters between versions.

## G. Tooling reliability

### G1. `user-obsidian` MCP server connection closed

- **Symptom:** Calling MCP tools (`list-available-vaults`, `create-directory`, `search-vault`) consistently returned "Connection closed" errors. Sometimes worked, often didn't.
- **Cause:** MCP server appeared unstable or restarting. No auth requirement was documented.
- **Fix:** Fell back to direct file operations on the vault path (`/Users/dev.sinha/Documents/Obsidian Vault/Nutanix-Core/...`). Obsidian vaults are just folders watched by Obsidian — writing markdown files directly is functionally equivalent to using the MCP API.
- **Lesson:** When an MCP server is flaky and the underlying data is a file system, **bypass the MCP**. Don't burn time on retries when a direct path exists.

## H. PR review interpretation

### H1. Judging "absolutely certain" for review comments

- **Symptom:** User instructed: "for the comments that you are absolutely certain of just add them and then push the code otherwise hold out." Several comments were genuinely ambiguous (e.g., T4 immutability doc — bot AI suggested wording that might not match user voice; T2 invalid product type test — user said optional).
- **Cause:** "Absolutely certain" is a fuzzy bar. Without a framework, I'd either over-act (apply borderline suggestions) or under-act (hold out on clearly correct fixes).
- **Fix:** Adopted the classification table now in [[11 - Process Lessons]]: mechanical fixes (always certain), explicit-doc-edit-per-reviewer (usually certain), behavior-change-with-reason (case-by-case), bot AI suggestions (almost never certain), design pushback (never certain alone). Applied T7/T9/gofmt (mechanical + explicit), held out T4 (bot-AI-borderline-doc), held out T2 (optional).
- **Lesson:** Before acting on any review comment, **classify it** by who said it (human vs bot), what kind of change it requires (mechanical vs design), and whether you have full context. Hold out preserves trust; over-acting on a wrong bot suggestion burns it.

## See also

- [[11 - Process Lessons]] for the abstract patterns derived from these incidents
- Up: [[00 - Index]]
