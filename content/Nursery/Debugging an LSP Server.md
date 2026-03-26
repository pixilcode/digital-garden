---
title: Debugging an LSP Server
tags:
  - tips
  - prog-lang
  - lsp
date: 2026-03-11
---
- **Viewing LSP logging** - to view the logs of an LSP server, open the "Output" tab on the bottom panel, then select the source as the name of the LSP server
- **Logging without access to the LSP logger** - if you don't have access to the LSP logger (for sending logging messages to the client), you can simply print to stderr, since that is directly displayed in the "Output" tab. You may need to flush stderr if it isn't displaying, I haven't tested that theory yet.