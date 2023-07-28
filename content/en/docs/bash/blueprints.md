---
title: "Blueprints"
weight: 10
description: >
  Templates for Bash scripts.
---

## General blueprint

```bash
#!/bin/bash         # declare the shell type, or it uses
# OR                  the default shell.
#!/usr/bin/bash    
# 
# 
# SCRIPT: 
# AUTHOR: 
# DATE: 
# REV: 1.1.A ([A]lpha, [B]eta, [D]ev, [T]est, and [P]roduction)
# 
# PLATFORM: Linux
# 
# PURPOSE: Give a clear description of the shell script.
# 
# REV LIST:
#       DATE:  
#       BY: 
#       MODIFICATION: 
# 
# set -n    # Uncomment to check script syntax, without execution.
#           # NOTE: Do not forget to add comment back in or the 
#                   script will not execute.
# 
# set -x    # Uncomment to debug this shell script.
# 
###############################################################################
#                   DEFINE FILE VARIABLES HERE
###############################################################################


###############################################################################
#                   DEFINE FUNCTIONS HERE
###############################################################################


###############################################################################
#                   BEGINNING OF MAIN
###############################################################################


# End of script
```