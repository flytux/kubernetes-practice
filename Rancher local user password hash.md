### Rancher user password hash - bcrypt

```
$ k get User u-gdh6j -o yaml

apiVersion: management.cattle.io/v3
description: ""
displayName: test
enabled: true
kind: User
metadata:
  annotations:
    field.cattle.io/creatorId: user-lv97z
    lifecycle.cattle.io/create.mgmt-auth-users-controller: "true"
  creationTimestamp: "2025-04-22T22:39:51Z"
  finalizers:
  - controller.cattle.io/mgmt-auth-users-controller
  generateName: u-
  generation: 3
  labels:
    cattle.io/creator: norman
  name: u-gdh6j
  resourceVersion: "2247096"
  uid: 9e5daf5a-578f-48e3-bc6b-e0d70f6679fa
password: $2a$10$Wqz9COWMAL5BQCn8Y0njiOdcVALgvH6B7wx7vgppIevJmIYgoGDhy
principalIds:
- local://u-gdh6j
spec: {}
status:
  conditions:
  - lastUpdateTime: "2025-04-22T22:39:51Z"
    status: "True"
    type: InitialRolesPopulated
username: test
```
