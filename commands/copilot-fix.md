---
description: "Ask GitHub Copilot to fix an error or bug"
---

Run the following command and show the user the output:

```bash
copilot -p "Fix this error or problem: $ARGUMENTS. Show what to change and why." -s --no-ask-user --no-auto-update --no-color --allow-tool='write, shell(git:*), shell(npm run:*), read'
```
