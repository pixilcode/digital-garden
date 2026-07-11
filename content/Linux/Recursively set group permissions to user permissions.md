---
title: Recursively set group permissions to user permissions
tags:
  - tips
  - topic-linux
  - topic-shell
date: 2026-08-26
---
If you need to set group permissions to match user permissions (such as if you are creating a shared folder and want to copy some user files in), you can use the following command:

```bash
chmod -R g=u <files/folders to update>
```

Note that this will recursively update any directory contents as well.

Taken from [this Server Fault answer](https://serverfault.com/a/241118).