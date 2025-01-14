---
title: Windows.Search.Yara
hidden: true
tags: [Client Artifact]
---

Searches for a specific malicious file or set of files by a Yara rule.


```yaml
name: Windows.Search.Yara
description: |
  Searches for a specific malicious file or set of files by a Yara rule.

tools:
  - name: YaraRules

parameters:
    - name: nameRegex
      description: Only file names that match this regular expression will be scanned.
      default: "(exe|txt|dll|php)$"
      type: regex
    - name: AlsoUpload
      type: bool
      description: Also upload matching files.

precondition:
  SELECT * FROM info() WHERE OS =~ "windows"

sources:
  - query: |
        LET yara_rules <= SELECT read_file(filename=FullPath) AS Rule,
           basename(path=FullPath) AS ToolName
        FROM Artifact.Generic.Utils.FetchBinary(
             ToolName="YaraRules", IsExecutable=FALSE)

        LET fileList = SELECT FullPath
        FROM parse_mft(
            accessor="ntfs",
            filename="C:\\$MFT")
        WHERE InUse
          AND FileName =~ nameRegex
          AND NOT FileName = yara_rules[0].ToolName
          AND NOT FullPath =~ "WinSXS"


        -- These files are typically short - only report a single hit.
        LET search = SELECT Rule, String.Offset AS HitOffset,
             str(str=String.Data) AS HitContext,
             FileName,
             File.Size AS Size,
             File.ModTime AS ModTime
        FROM yara(
            rules=yara_rules[0].Rule, key="A",
            files="C:/" + FullPath)
        LIMIT 1

        -- Only do something when yara rules are available.
        SELECT * FROM if(condition=yara_rules,
        then={
          SELECT *, if(condition=AlsoUpload, then=upload(file=FileName)) AS Upload
          FROM foreach(row=fileList, query=search)
        })

```
