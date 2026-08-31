---
type: pattern
created: 2026-08-30
tags: [failure]
sources:
  - "[[inbox/ENG-932537 - IAM products field/10 - Process Lessons]]"
---

# ASCII-only Tavern YAML

## Failure mode
Em-dash or `§` in a Tavern stage name. Collector runs on Python 2.7 ASCII `str` and explodes in `"{:d}: {:s}".format(...)`.

## Workaround
Before commit:

```bash
perl -CSD -ne 'print "LINE $.: ", $_ if /[^\x00-\x7F]/' path/to.tavern.yaml
```

No output expected.

## Evidence
ENG-932537 process lesson 5.
