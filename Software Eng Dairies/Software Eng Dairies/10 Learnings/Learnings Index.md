---
type: index
tags:
  - index
---
# Learnings Index

Notes on articles, talks, papers, books, courses and concepts. One note per thing I read or watched, made from the **Learning** template.

`status` is `reading`, `done` or `revisit`.

```base
filters:
  and:
    - type == "learning"
    - file.folder != "90 Templates"
views:
  - type: table
    name: All learnings
    order:
      - file.name
      - status
      - topics
      - source
      - created
    sort:
      - property: created
        direction: DESC
  - type: table
    name: To revisit
    filters:
      and:
        - status == "revisit"
    order:
      - file.name
      - topics
```
