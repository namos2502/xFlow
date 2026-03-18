---
description: "Ask GitHub Copilot to review staged changes or a specific file"
---

If $ARGUMENTS is a file path, run:

```bash
copilot -p "/review $ARGUMENTS — focus on bugs, logic errors, and security issues." -s --no-ask-user --no-auto-update --no-color --allow-tool='shell(git:*)'
```

Otherwise run:

```bash
copilot -p "/review the changes on this branch compared to main. Focus on bugs and security issues." -s --no-ask-user --no-auto-update --no-color --allow-tool='shell(git:*)'
```
