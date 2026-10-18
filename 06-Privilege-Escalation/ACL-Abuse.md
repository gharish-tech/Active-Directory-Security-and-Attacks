# Active Directory ACL Abuse Lab — Practical Documentation

## 1. Objective

The objective of this lab was to understand how misconfigured Active Directory Access Control Lists (ACLs) can grant users unintended permissions over other domain objects.

The practical focused on inspecting object permissions, identifying potentially risky access rights, and understanding how an attacker might abuse excessive permissions in a controlled lab environment.

## 2. Lab Environment

| Component | Details |
|---|---|
| Domain | `harish.local` |
| NetBIOS domain | `HARISH` |
| Domain Controller | `DC01` |
| DC01 IP | `192.168.182.10` |
| Windows Client | `CLIENT01` |
| Client IP | `192.168.182.20` |
| Kali Linux IP | `192.168.182.129` |
| Test accounts | `acl.target`, `acl.operator` |
| Tools | Windows PowerShell, Active Directory tools, VMware Workstation |

## 3. What Is ACL Abuse?

Active Directory uses Access Control Lists to determine which users and groups can perform operations on directory objects.

An object's Discretionary Access Control List (DACL) contains Access Control Entries (ACEs) that define permissions for security principals.

Examples of potentially sensitive permissions include:

- **GenericAll:** Broad control over an object.
- **GenericWrite:** Ability to modify certain object attributes.
- **WriteProperty:** Permission to modify specified properties.
- **WriteDACL:** Permission to modify an object's DACL.
- **WriteOwner:** Permission to change an object's owner.
- **Extended rights:** Specific operations, such as resetting a password, when the relevant right is granted.

The actual impact depends on the object, the precise ACE, inheritance, group membership, and the operation being attempted. The presence of an ACL entry alone does not prove that a privilege-escalation path is exploitable.

## 4. Lab Workflow

1. Identify the test accounts and their directory objects.
2. Inspect permissions assigned to the target object.
3. Review potentially sensitive rights and inherited ACEs.
4. Determine whether the operator account has a usable permission path.
5. Attempt an authorized validation in the isolated lab.
6. Record the result and document defensive recommendations.

## 5. Practical Investigation and Observations

### Step 1: Identify the Target Object

The lab used a dedicated test account named `acl.target`. Its directory object was identified using the distinguished name:

`CN=ACL Target,CN=Users,DC=harish,DC=local`

The operator account was `acl.operator`.

These accounts were created for the lab and were separate from ordinary user accounts.

### Step 2: Inspect Active Directory Permissions

The target object's permissions were inspected using PowerShell and the Active Directory provider.

During the investigation, the target object showed inherited `ReadProperty` and `WriteProperty` entries associated with the **Key Admins** and **Enterprise Key Admins** groups for the `msDS-KeyCredentialLink` property.

These entries were recorded as findings for further investigation. Their presence alone does not establish that `acl.operator` belongs to either group or can successfully modify the target object's key credentials.

### Step 3: Review Group Membership

The direct membership counts checked for `Key Admins` and `Enterprise Key Admins` were both zero.

This result only describes the direct membership counts that were queried. It does not, by itself, establish all possible effective permissions, nested-group relationships, or access paths in the domain.

### Step 4: Check Password-Reset Permissions

The investigation checked for the Reset Password extended-right GUID:

`00299570-246d-11d0-a768-00aa006e0529`

The query returned no matching entry for the objects that were checked.

Therefore, the investigation did not establish a direct Reset Password ACE on those objects from that query. This does not rule out every other possible permission path.

### Step 5: Validate the Operator Context

The `acl.operator` account was successfully used to start a separate PowerShell process on `CLIENT01`, and the identity check returned:

`harish\acl.operator`

The account could not be used for interactive logon to `DC01` because the requested logon type was not granted. The domain controller's logon policy was not weakened to bypass this restriction.

### Step 6: Attempt Validation

A password-reset validation was planned, but the required `Set-ADAccountPassword` PowerShell cmdlet was unavailable in the client environment because the Active Directory PowerShell module was not installed there.

Consequently, the intended password-reset operation was not completed, and no successful ACL-abuse result was established.

## 6. Results Summary

| Investigation | Observed result |
|---|---|
| Lab test accounts | Created |
| Target object identified | Yes |
| Selected ACL entries inspected | Yes |
| Inherited property permissions observed | Yes |
| Direct group membership counts checked | Both queried groups returned zero |
| Reset Password right queried | No matching entry returned for the queried objects |
| Operator process on CLIENT01 | Started successfully |
| Password-reset validation | Not completed |
| Successful privilege escalation | Not demonstrated |

## 7. Security Impact

A misconfigured ACL can allow a less-privileged account to modify another account or directory object. Depending on the granted right, this may enable unauthorized changes, account takeover, or further privilege escalation.

Potentially sensitive permissions should be assessed in context rather than treated as automatically exploitable. Effective permissions and the actual operation must be validated before concluding that an attack path exists.

## 8. Detection and Monitoring

Defenders can investigate suspicious Active Directory changes using Windows Security auditing and directory-service telemetry.

Relevant events, when the appropriate auditing is configured, may include:

- **4662:** An operation was performed on an Active Directory object.
- **5136:** A directory service object was modified.
- **4724:** An attempt was made to reset an account's password.
- **4738:** A user account was changed.

Event generation depends on audit policy, object SACL configuration where applicable, and the action performed. These are suggested monitoring sources, not events confirmed as collected during this lab.

Investigations should correlate the actor, target object, changed attributes, source host, timestamp, and expected administrative activity.

## 9. Mitigation

1. Follow least privilege when assigning Active Directory permissions.
2. Review ACLs on privileged users, groups, organizational units, and domain objects.
3. Remove unnecessary write and extended-right permissions.
4. Review inherited permissions and nested group memberships.
5. Monitor changes to sensitive directory attributes and ACLs.
6. Use dedicated administrative accounts and restrict privileged access.
7. Reassess permissions after organizational or domain changes.

## 10. Limitations

The investigation identified permission entries that warranted review, but it did not demonstrate successful exploitation of an ACL misconfiguration.

The password-reset validation was blocked by the missing PowerShell module in the client environment. No claim is made that the operator account reset the target account's password, obtained elevated privileges, or compromised the domain.

## 11. Conclusion

This practical introduced Active Directory ACL analysis, inherited permissions, sensitive directory rights, and the importance of validating effective access before declaring a privilege-escalation path.

Although successful ACL abuse was not demonstrated, the investigation provided experience with identifying potentially risky permissions and documenting technical limitations accurately.

**Project status:** ACL inspection completed; exploitation validation incomplete.

