---
type: topic
created: {{date}}
tags:
  - topic
---
# {{title}}

Short description of what belongs under this topic.

## Key notes
Hand-picked notes worth re-reading.
-

## Everything linked here
```base
filters:
  and:
    - file.hasLink(this.file)
    - file.folder != "90 Templates"
views:
  - type: table
    name: Linked notes
    order:
      - file.name
      - type
      - status
      - created
```
Any note with `topics: [[{{title}}]]` in its properties shows up in the table above and in the Backlinks pane, so that one line is all it takes to file a note here.

## Open questions
-
