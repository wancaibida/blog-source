title: Terraform connection ssh private key用法
author: 大丈夫没问题
tags:
  - linux
categories:
  - linux
date: 2021-06-28 21:54:00
---
base64 encode一下private key：

```
cat ~/.ssh/your_private_key | base64
```

connection中decode一下：

```
  connection {
    type        = "ssh"
    user        = "admin"
    private_key = base64decode(var.private_key)
    host        = self.public_ip_address
    timeout     = "3600s"
  }

```

