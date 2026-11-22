---
type: index
tags:
  - index
---
# Open Source Index

Every repo, library or tool I've found worth remembering. One note per project, made from the **Open Source Repo** template and kept in this folder.

`status` moves through `to-try` → `tried` → `using` (or `dropped`).

```base
filters:
  and:
    - type == "oss"
    - file.folder != "90 Templates"
views:
  - type: table
    name: All projects
    order:
      - file.name
      - status
      - language
      - topics
      - repo
      - created
    sort:
      - property: created
        direction: DESC
  - type: table
    name: To try
    filters:
      and:
        - status == "to-try"
    order:
      - file.name
      - topics
      - repo
```
