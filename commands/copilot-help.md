---
description: "Show all available copilot-bridge commands"
---

Display the following reference table to the user:

## 🛩️ copilot-bridge — Command Reference

| Command | Description | Example |
|---|---|---|
| `/copilot-ask` | Ask Copilot any question | `/copilot-ask how do I reverse a string in bash?` |
| `/copilot-suggest` | Get a shell command for a task | `/copilot-suggest delete all node_modules folders recursively` |
| `/copilot-explain` | Explain a command or snippet | `/copilot-explain git rebase -i HEAD~3` |
| `/copilot-fix` | Fix an error or bug | `/copilot-fix permission denied when running npm install` |
| `/copilot-review` | Review staged diff or a file | `/copilot-review` or `/copilot-review src/index.ts` |
| `/copilot-help` | Show this reference | `/copilot-help` |

All commands delegate to `copilot -p` with `-s --no-ask-user --no-auto-update --no-color`.
