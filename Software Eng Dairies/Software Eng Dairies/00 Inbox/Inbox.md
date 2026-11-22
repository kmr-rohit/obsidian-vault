---
type: index
tags:
  - index
---
# Inbox

Drop anything here first: a link, a half-formed idea, a repo someone mentioned. New notes land in this folder by default. Don't organise while capturing.

Once a week, go through the list below and for each capture either:
1. turn it into a **Learning**, **Open Source Repo** or **Experiment** note and move it to its folder,
2. or delete it if it no longer seems interesting.

```base
filters:
  and:
    - file.inFolder("00 Inbox")
    - file.name != "Inbox"
views:
  - type: table
    name: Waiting to be filed
    order:
      - file.name
      - source
      - created
    sort:
      - property: created
        direction: ASC
```
