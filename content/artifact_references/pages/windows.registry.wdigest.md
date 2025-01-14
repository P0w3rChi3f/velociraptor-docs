---
title: Windows.Registry.WDigest
hidden: true
tags: [Client Artifact]
---

Find WDigest registry values on the filesystem.

In order to prevent the “clear-text” password from being placed in
LSASS, the following registry key needs to be set to “0” (Digest
Disabled):

 - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest “UseLogonCredential”(DWORD)

This registry key is worth monitoring in an environment as an
attacker may wish to set it to 1 to enable Digest password support
which forces “clear-text” passwords to be placed in LSASS on any
version of Windows from Windows 7 / 2008R2 up to Windows 10 /
2012R2. Furthermore, Windows 8.1 / 2012 R2 and newer do not have a
“UseLogonCredential” DWORD value, so the key needs to be
added. The existence of the key is suspicious, if not expected.

* ATT&CK tactic: Defense Evasion, Credential Access
* ATT&CK technique: T1112, T1003.001


```yaml
name: Windows.Registry.WDigest
description: |
    Find WDigest registry values on the filesystem.

    In order to prevent the “clear-text” password from being placed in
    LSASS, the following registry key needs to be set to “0” (Digest
    Disabled):

     - HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest “UseLogonCredential”(DWORD)

    This registry key is worth monitoring in an environment as an
    attacker may wish to set it to 1 to enable Digest password support
    which forces “clear-text” passwords to be placed in LSASS on any
    version of Windows from Windows 7 / 2008R2 up to Windows 10 /
    2012R2. Furthermore, Windows 8.1 / 2012 R2 and newer do not have a
    “UseLogonCredential” DWORD value, so the key needs to be
    added. The existence of the key is suspicious, if not expected.

    * ATT&CK tactic: Defense Evasion, Credential Access
    * ATT&CK technique: T1112, T1003.001

type: CLIENT

author: Eduardo Mattos - @eduardfir

precondition:
  SELECT * FROM info() where OS = 'windows'

parameters:
  - name: SearchRegistryGlob
    default: HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\SecurityProviders\WDigest\**
    description: Use a glob to define the files that will be searched.

sources:
  - query: |
        SELECT  Name,
                FullPath,
                Data,
                Sys,
                ModTime as Modified
        FROM glob(globs=SearchRegistryGlob, accessor='registry')
        WHERE Name =~ "LogonCredential"

column_types:
  - name: Modified
    type: timestamp

```
