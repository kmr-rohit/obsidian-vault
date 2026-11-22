---
type: index
tags:
  - index
---
# Tried It Log

Things I've actually run, built or poked at. One note per experiment, made from the **Experiment** template and kept in this folder, newest first below.

`outcome` is one of `worked`, `partial` or `failed`.

What to try next is queued in [[Next Learning and Experiments]].

```base
filters:
  and:
    - type == "experiment"
    - file.folder != "90 Templates"
views:
  - type: table
    name: Log
    order:
      - tried
      - file.name
      - outcome
      - status
      - topics
    sort:
      - property: tried
        direction: DESC
```
