---
title: Server.Utils.DeleteNotebook
hidden: true
tags: [Server Artifact]
---

Completely removes a notebook from the server including all its cells, attachments etc.


```yaml
name: Server.Utils.DeleteNotebook
description: |
  Completely removes a notebook from the server including all its cells, attachments etc.

type: SERVER

parameters:
  - name: NotebookId
    description: The ID of the notebook to remove.
  - name: ReallyDoIt
    type: bool
    description: Set to really remove the notebook - otherwise it is a dry run.

sources:
  - query: |
      SELECT * FROM notebook_delete(
          notebook_id=NotebookId, really_do_it=ReallyDoIt)

```
