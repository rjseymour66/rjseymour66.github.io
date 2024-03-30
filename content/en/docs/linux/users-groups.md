---
title: "Users and groups"
weight: 90
description: >
  Linux and localization.
---

_Authentication_ is when you determine that a user is authentic - they are who they claim to be.

## Managing user accounts

### Adding accounts

A few files are used to create accounts:

```
/etc/default/useradd    >>>                             >>> /home/userid

/etc/login.defs         >>>                             >>> /etc/passwd

                                User account created    

/etc/skel               >>>                             >>> /etc/shadow

admin input             >>>                             >>> /etc/group
```

