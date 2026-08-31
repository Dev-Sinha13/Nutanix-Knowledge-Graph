---
type: pattern
created: 2026-08-30
tags: [failure]
sources:
  - "[[inbox/ENG-932537 - IAM products field/10 - Process Lessons]]"
---

# Vendor gofmt cascade

## Failure mode
Adding one field to a vendored file, then `gofmt -w`, rewrites indentation (2-space → tabs) and explodes the diff by tens of thousands of lines.

## Workaround
Match the file’s existing indentation via surgical edit. If `git diff --stat` is >~50 lines for a one-field add: restore, re-apply only the semantic change.

## Evidence
ENG-932537 incident B1; lesson 7.
