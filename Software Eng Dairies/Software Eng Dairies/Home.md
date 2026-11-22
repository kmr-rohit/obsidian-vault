---
type: index
tags:
  - index
---
# Home

My notebook for everything I read, try or find interesting in software.

**What's next:** [[Next Learning and Experiments]] is the prioritized list of what to learn and what to build next.

## Where things go

| Folder | What goes in it | Template |
|---|---|---|
| [[Inbox\|00 Inbox]] | Quick captures: a link or an idea, unfiled | Capture |
| [[Learnings Index\|10 Learnings]] | Articles, talks, papers, books, concepts | Learning |
| [[Open Source Index\|20 Open Source]] | Repos, libraries and tools worth remembering | Open Source Repo |
| [[Tried It Log\|30 Experiments]] | Things I actually ran or built, and how it went | Experiment |
| 40 Topics | One page per subject, collecting everything linked to it | Topic |
| 90 Templates | Note templates (don't put notes here) | |
| 99 Attachments | Images and files pasted into notes | |

## Topics
- [[Databases]]
- [[AI and ML]]
- [[Distributed Systems]]
- [[Programming Languages]]
- [[DevOps and Infra]]
- [[Web and Frontend]]
- [[Security]]
- [[Developer Tools]]

New topic? Create a note in `40 Topics` from the **Topic** template and add it to the list above.

## Conventions

**Capture first, file later.** New notes land in `00 Inbox`. Empty it once a week.

**Making a note from a template.** Create the note, then run *Templates: Insert template* from the command palette (Ctrl/Cmd + P) and pick one.

**File names.** Use the thing's real name: `PostgreSQL MVCC`, `ripgrep`, `Trying out DuckDB`. No dates in file names; the date lives in `created`.

**Properties every note has:**

| Property | Values |
|---|---|
| `type` | `capture`, `learning`, `oss`, `experiment`, `topic` |
| `status` | learning: `reading` · `done` · `revisit`<br>oss: `to-try` · `tried` · `using` · `dropped`<br>experiment: `in-progress` · `done` |
| `created` | `YYYY-MM-DD` |
| `topics` | one or more links to topic pages, e.g. `"[[Databases]]"` |
| `tags` | the type tag (`learning`, `oss`, `experiment`), plus any free-form tags |

Notes of other types add their own: `source` and `author` for learnings, `repo`, `language` and `license` for open source, `tried`, `outcome` (`worked` · `partial` · `failed`) and `repo` for experiments.

**Linking.** `topics` is how a note gets filed: link a note to a topic and it shows up on that topic page. Link related notes to each other freely, for example an Open Source note to the Experiment where I tried it.

**Tags** are for cross-cutting labels that aren't topics, like `#til`, `#interview-prep`, `#paper` or `#talk`. Keep them lowercase and hyphenated.
