# Set up an SMB server in FSx ONTAP with Local User

1. Create FSx with Mixed as Security style
   - Volume - v1 (Path: /v1)
2. [Login to the Server with ONTAP CLI](https://docs.aws.amazon.com/fsx/latest/ONTAPGuide/managing-resources-ontap-apps.html#vsadmin-ontap-cli)
```shell
➜  ~ ssh vsadmin@<Management endpoint - IP address of SVM>
```
3. [Create the SMB server in a workgroup](https://docs.netapp.com/us-en/ontap/smb-config/create-server-workgroup-task.html)
```shell
➜ fsx::> vserver
➜ fsx::vserver> cifs create -cifs-server SMB01 -workgroup SMB
```
4. [Create a local user](https://docs.netapp.com/us-en/ontap/smb-config/create-local-user-accounts-task.html)
```shell
➜ fsx::vserver> cifs users-and-groups local-user create  -user-name smblocal

Enter the password:
Confirm the password:

➜ fsx::vserver>
```   

5. [Create  a local group](https://docs.netapp.com/us-en/ontap/smb-config/create-local-groups-task.html)
```shell
➜ fsx::vserver> cifs users-and-groups local-group create  -group-name infra
```
6. [Add group members](https://docs.netapp.com/us-en/ontap/smb-config/manage-local-group-membership-task.html)

```shell
➜ fsx::vserver> cifs users-and-groups local-group add-members -group-name infra -member-names smblocal
```
7. [Create a SMB share](https://docs.netapp.com/us-en/ontap/smb-config/create-share-task.html)

```shell
➜ fsx::vserver> cifs share create -share-name smb-share -path /v1
```
8. [Update SMB ACL / Group Permission](https://docs.netapp.com/us-en/ontap/smb-config/create-share-access-control-lists-task.html)

- Delete default ACL
```shell
➜ fsx::vserver> cifs share access-control delete -share smb-share -user-or-group everyone
```
- Create a new ACL for the SMB share
```shell
➜ fsx::vserver> cifs share access-control create -share smb-share -user-group-type windows -user-or-group infra -permission Change
```
- Verify the Access
```shell
➜ fsx::vserver> cifs share access-control show -share smb-share
               Share       User/Group                  User/Group  Access
Vserver        Name        Name                        Type        Permission
-------------- ----------- --------------------------- ----------- -----------
fsx            smb-share   infra                       windows     Change
```