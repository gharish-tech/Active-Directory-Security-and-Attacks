# SharpHound — Active Directory Security Lab

## 1. Objective

Understand SharpHound's role in Active Directory assessment, document its source acquisition and build status, and explain how collected directory relationships support security analysis.

## 2. Overview

SharpHound is a C# data collector for BloodHound. It gathers information about Active Directory objects and relationships so BloodHound can analyze permissions, group membership, trusts, and potential privilege paths.

Official resources:

- Repository: https://github.com/SpecterOps/SharpHound
- BloodHound documentation: https://bloodhound.specterops.io/

## 3. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| Domain Controller | `DC01` |
| DC01 IP | `192.168.182.10` |
| Windows client | `CLIENT01` |
| Environment | Isolated VMware lab |
| Intended source directory | `C:\Lab\SharpHound` |

## 4. How SharpHound Fits into BloodHound

SharpHound collects directory data. BloodHound imports and analyzes that data using graph relationships.

Examples of relationships that can be examined include:

- User-to-group membership.
- Group-to-group membership.
- Computer and domain relationships.
- Delegated permissions on directory objects.
- Potential paths to privileged accounts.

A graph relationship indicates a relationship or possible access path to investigate. It does not, by itself, prove that an account compromise or privilege escalation succeeded.

## 5. Source Acquisition and Build

The source is obtained from the official SpecterOps repository. The repository documents compilation using the .NET SDK with `dotnet restore` and `dotnet build`.

Progress checklist:

- [ ] Official repository accessed.
- [ ] Source downloaded or cloned.
- [ ] Source extracted to `C:\Lab\SharpHound`.
- [ ] .NET SDK requirements checked.
- [ ] Build result recorded, if compiled.
- [ ] Screenshot of the source or build output added.

The collector version should be compatible with the BloodHound deployment being used.

## 6. Collection Scope

Collection methods can gather different categories of directory information. The selected scope should match the lab objective and the permissions granted to the test account.

For this lab, begin with understanding the data model and reviewing the existing directory configuration. Do not collect more information than necessary.

## 7. Defensive Relevance

SharpHound is useful to defenders because its collected relationships can reveal:

- Excessive group membership.
- Unnecessary delegated permissions.
- Risky access paths to privileged accounts.
- Misconfigured administrative relationships.
- Opportunities to reduce privilege exposure.

Collection activity can also be investigated through domain-controller and endpoint telemetry. No single event should be treated as conclusive evidence of malicious activity.

## 8. Scope and Limitations

This document covers SharpHound's purpose and source/build workflow. It does not claim that the tool has been built or executed until the relevant evidence is recorded.

No domain collection results or privilege-escalation findings are claimed in this module.

## 9. Conclusion

SharpHound supplies relationship data for BloodHound analysis. Combined with careful permission review, it helps administrators and security assessors understand and reduce excessive privileges in Active Directory.

